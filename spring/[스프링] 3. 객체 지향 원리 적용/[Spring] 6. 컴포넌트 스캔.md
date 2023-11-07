## 컴포넌트 스캔과 의존관계 자동 주입

스프링은 설정 정보가 없어도 자동으로 스프링 빈을 등록하는 컴포넌트 스캔이라는 기능을 제공

`@Autowired` 를 통해 의존관계도 자동 주입해줍니다.

```java
@Configuration
@ComponentScan(excludeFilters = @Filter(type = FilterType.ANNOTATION, classes =Configuration.class))
public class AutoAppConfig {
 
}
```

- 컴포넌트 스캔을 사용하려면 먼저 `@ComponentScan`을 설정 정보에 붙여줘야 합니다.

@Bean으로 등록한 클래스가 없다. 

컴포넌트 스캔은 이름 그대로 `@Component` 애노테이션을 붙은 클래스를 스캔해서, 스프링 빈으로 등록해줍니다.

```java
@Component
public class OrderServiceImpl implements OrderService {
	 private final MemberRepository memberRepository;
	 private final DiscountPolicy discountPolicy;

	 @Autowired
	 public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
		 this.memberRepository = memberRepository;
		 this.discountPolicy = discountPolicy;
		 }
}
```

- 이전에 AppConfig에서는 @Bean으로 직접 설정 정보를 작성했고, 의존관계도 직접 명시했다.
    
    이제는 설정 정보 자체가 없기 때문에, 의존관계 주입도 @Component를 작성한 클래스 안에서 해결해야 합니다.
    
- @Autowired는 의존관계를 자동으로 주입해줍니다.
    - 생성자에서 여러 의존관계도 주입받을 수 있습니다.

![image](https://github.com/kylo-dev/Spring-Jpa-Study/assets/103489352/46f2718f-ad99-467b-9e7e-662e45d0c8d4)


- `@ComponentScan`은 `@Component`가 붙은 모든 클래스를 스프링 빈으로 등록합니다.
- 이때 스프링 빈의 기본 이름은 클래스명을 사용하되 맨 앞글자만 소문자를 사용합니다.

![image](https://github.com/kylo-dev/Spring-Jpa-Study/assets/103489352/3c1b5e48-24f5-4bfa-9a89-f15511704479)


- 생성자에 `@Autowired`를 지정하면, 스프링 컨테이너가 자동으로 해당 스프링 빈을 찾아서 주입합니다.
- 기본 조회 전략은 타입이 같은 빈을 찾아서 주입합니다.
    - `getBean(MemberRepository.class)`와 동일하다고 생각할 수 있습니다.
    - 즉, MemberRepository.class의 자식 객체들도 조회됨 (memoryMemberRepository가 주입)

## 탐색 위치와 기본 스캔 대상

권장하는 방법

패키지 위치를 지정하지 않고, 설정 정보 클래스의 위치를 프로젝트 최상단에 위치 시키기

> 스프링 부트를 사용하면 스프링 부트의 대표 시작 정보인 @SpringBootApplication을 프로젝트 시작 루트 위치에 둔다. (이 설정안에 @ComponentScan이 들어있음
> 

### 컴포넌트 스캔 기본 대상

- @Component : 컴포넌트 스캔에서 사용
- @Controlller : 스프링 MVC 컨트롤러에서 사용
- @Service : 스프링 비즈니스 로직에서 사용
- @Repository : 스프링 데이터 접근 계층에서 사용
- @Configuration : 스프링 설정 정보에서 사용

## 중복 등록과 충돌

1. 자동 빈 등록 vs 자동 빈 등록
2. 수동 빈 등록 vs 자동 빈 등록

### 자동 빈 등록 vs 자동 빈 등록

- 컴포넌트 스캔에 의해 자동으로 스프링 빈이 등록되는데, 그 이름이 같은 경우 스프링은 오류를 발생합니다.
    - ConflictingBeanDefinitionException 예외 발생
    

### 수동 빈 등록 vs 자동 빈 등록

- 수동 빈 등록이 우선권을 가집니다. (수동 빈이 자동 빈을 오버라이딩 합니다.)
