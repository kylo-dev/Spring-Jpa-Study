## 스프링 컨테이너 생성

```java
//스프링 컨테이너 생성
ApplicationContext applicationContext = new AnnotationConfigApplicationContext(AppConfig.class);
```

- ApplicationContext 를 스프링 컨테이너라 합니다. (인터페이스)

### 1. 스프링 컨테이너의 생성 과정

![image](https://github.com/kylo-dev/Spring-Jpa-Study/assets/103489352/503cba16-8a51-4aa5-a2fa-11d1e0cfe06a)


- new AnnotationConfigApplicationContext(AppConfig.class)
- 스프링 컨테이너를 생성할 때는 구성 정보를 지정해 주어야 합니다. (AppConfig.class)

### 2. 스프링 빈 등록

![image](https://github.com/kylo-dev/Spring-Jpa-Study/assets/103489352/2d8ab2df-f04b-4613-9e17-db81c843135e)

- 스프링 컨테이너는 **파라미터로 넘어온 설정 클래스 정보를 사용**해서 **스프링 빈을 등록**합니다.

### 빈 이름

- 메서드 이름을 사용합니다.
- 빈 이름을 직접 부여할 수 있습니다.
    
    **`@Bean(name=”memberService2”)`**
    

> **`주의` 빈 이름은 항상 다른 이름을 부여해야 합니다.** 같은 이름을 부여하면, 다른 빈이 무시되거나, 기존 빈을 덮어버리는 설정 오류가 발생할 수 있습니다.
> 

### 3. 스프링 빈 의존 관계 설정 - 준비

![image](https://github.com/kylo-dev/Spring-Jpa-Study/assets/103489352/7555fd09-9c6f-458b-ac6e-e45266e97f48)

- 각 빈들이 컨테이너에 생성됩니다.

### 4. 스프링 빈 의존 관계 설정 - 완료

![image](https://github.com/kylo-dev/Spring-Jpa-Study/assets/103489352/ae98488a-f9f3-4438-b0c8-4898e4bf2ec9)

- 스프링 컨테이너는 설정 정보를 참고해서 의존 관계를 주입(DI)합니다.

**정리**

스프링 컨테이너를 생성하고, 설정 정보를 참고해서 스프링 빈도 등록하고, 의존 관계를 설정합니다.

## 컨테이너에 등록된 모든 빈 조회

```java
class ApplicationContextInfoTest {
	AnnotationConfigApplicationContext ac = 
					new AnnotationConfigApplicationContext(AppConfig.class);
	 @Test
	 @DisplayName("모든 빈 출력하기")
	 void findAllBean() {
		 String[] beanDefinitionNames = ac.getBeanDefinitionNames();
		 for (String beanDefinitionName : beanDefinitionNames) {
			 Object bean = ac.getBean(beanDefinitionName);
			 System.out.println("name=" + beanDefinitionName + " object=" + bean);
		 }
	 }
}
```

- 모든 빈 출력하기
    - **`ac.getBeanDefinitionNames()`** 스프링에 등록된 모든 빈 이름을 조회
    - **`ac.getBean()`** 빈 이름으로 빈 객체를 조회

## 스프링 빈 조회

- **`ac.getBean(빈 이름, 타입)`**
- **`ac.getBean(타입)`**

- 조회 대상 스프링 빈이 없으면 예외가 발생합니다.
    - `NoSuchBeanDefinitionException : No bean named ‘XXXX’ available`

```java
class ApplicationContextBasicFindTest {
	 AnnotationConfigApplicationContext ac = 
			new AnnotationConfigApplicationContext(AppConfig.class);
	 @Test
	 @DisplayName("빈 이름으로 조회")
	 void findBeanByName() {
		 MemberService memberService = ac.getBean("memberService",
																									MemberService.class);
		 assertThat(memberService).isInstanceOf(MemberServiceImpl.class);
	 }

	 @Test
	 @DisplayName("이름 없이 타입만으로 조회")
	 void findBeanByType() {
		 MemberService memberService = ac.getBean(MemberService.class);
		 assertThat(memberService).isInstanceOf(MemberServiceImpl.class);
	 }

	 @Test
	 @DisplayName("구체 타입으로 조회")
	 void findBeanByName2() {
		 MemberServiceImpl memberService = ac.getBean("memberService",
																												MemberServiceImpl.class);
		 assertThat(memberService).isInstanceOf(MemberServiceImpl.class);
	 }

	 @Test
	 @DisplayName("존재하지 않는 빈 조회")
	 void findBeanByNameX() {
	 //ac.getBean("xxxxx", MemberService.class);
	 Assertions.assertThrows(NoSuchBeanDefinitionException.class, () ->
																			ac.getBean("xxxxx", MemberService.class));
	 }
}
```

> **`참고`** 구체 타입으로 조회하면 변경시 우연성이 떨어지게 됩니다.
> 

## 스프링 빈 조회 - 동일한 타입이 둘 이상

- 타입으로 조회 시 같은 타입의 스프링 빈이 둘 이상이면 오류가 발생합니다. (빈 이름을 지정하자)
- **`ac.getBeansOfType()`** 을 사용하면 해당 타입의 모든 빈을 조회할 수 있습니다.

```java
class ApplicationContextSameBeanFindTest {
 AnnotationConfigApplicationContext ac = 
				new AnnotationConfigApplicationContext(SameBeanConfig.class);

 @Test
 @DisplayName("타입으로 조회시 같은 타입이 둘 이상 있으면, 중복 오류가 발생한다")
 void findBeanByTypeDuplicate() {
	 //MemberRepository bean = ac.getBean(MemberRepository.class);
	 assertThrows(NoUniqueBeanDefinitionException.class, () ->
																ac.getBean(MemberRepository.class));
 }

 @Test
 @DisplayName("타입으로 조회시 같은 타입이 둘 이상 있으면, 빈 이름을 지정하면 된다")
 void findBeanByName() {
	 MemberRepository memberRepository = ac.getBean("memberRepository1",
																											MemberRepository.class);
	 assertThat(memberRepository).isInstanceOf(MemberRepository.class);
 }
 
 @Configuration
 static class SameBeanConfig {
	 @Bean
	 public MemberRepository memberRepository1() {
		 return new MemoryMemberRepository();
	 }

	 @Bean
	 public MemberRepository memberRepository2() {
		 return new MemoryMemberRepository();
	 }
	}
}
```

## 스프링 빈 조회 - 상속 관계

- 부모 타입으로 조회하면, 자식 타입도 함께 조회됩니다.

![image](https://github.com/kylo-dev/Spring-Jpa-Study/assets/103489352/2767f0b5-2e5c-47a0-8129-d36527188964)

## BeanFactory와 ApplicationContext

![image](https://github.com/kylo-dev/Spring-Jpa-Study/assets/103489352/cb0a23a5-56d9-4787-a5e2-9c3c92f2f9a9)

**BeanFactory**

- 스프링 컨테이너의 최상위 인터페이스
- 스프링 빈을 관리하고 조회하는 역할
- getBean() 을 제공

**ApplicationContext**

- BeanFactory 기능을 모두 상속받아서 제공
- 빈 관리 기능 + 편리한 부가 기능을 제공

## 스프링 빈 설정 메타 정보 - BeanDefinition

- 스프링 컨테이너는 **`@Bean`** 이 메타정보를 기반으로 스프링 빈을 생성합니다.
