# [스프링] 5. 싱글톤(스프링) 컨테이너

### 스프링 없는 순수한 DI Container

✔ 스프링을 사용하지 않는 DI 컨테이너인 AppConfig는 사용자가 요청을 할 때마다 객체를 계속해서 새로 생성합니다.

- > ***요청할 때마다 객체를 계속해서 생성하므로 메모리 낭비***가 심합니다.

### 해결방안

✔ 해당 객체가 딱 1개만 생성되고 (Singleton), 공유하도록 설계하여 반복 생성되는 구조를 피하는 것입니다.

![https://blog.kakaocdn.net/dn/b0UPbh/btr6eUAS6LJ/Dh98GYZdVKgqsWgZpsLV3K/img.png](https://blog.kakaocdn.net/dn/b0UPbh/btr6eUAS6LJ/Dh98GYZdVKgqsWgZpsLV3K/img.png)

```java
public class SingletonTest {

    @Test
    @DisplayName("스프링 없는 순수한 DI 컨테이너")
    void pureContainer(){
        AppConfig appConfig = new AppConfig();

				// 1. 조회: 호출할 때 마다 객체를 생성
        MemberService memberService1 = appConfig.memberService();
				// 2. 조회: 호출할 때 마다 객체를 생성
        MemberService memberService2 = appConfig.memberService();

				// 참조값이 다른 것을 확인
        System.out.println("memberService1 = " + memberService1);
        System.out.println("memberService2 = " + memberService2);

				// memberService1 != memberService2
        assertThat(memberService1).isNotSameAs(memberService2);
    }
```

![https://blog.kakaocdn.net/dn/d6frx8/btr6oJSFnBy/pnsZTWunKFFaKL9IC5Skq1/img.png](https://blog.kakaocdn.net/dn/d6frx8/btr6oJSFnBy/pnsZTWunKFFaKL9IC5Skq1/img.png)

### 싱글톤 패턴이란

✔ **클래스의 인스턴스가 딱 1개만 생성**되는 패턴을 의미합니다.

✔ **private 생성자**를 사용하여 외부에서 임의로 new 키워드로 객체 생성을 하지 못하게 합니다.

✔ 싱글톤 클래스를 Static 영역에 미리 만들어 하나를 메모리에 올려둡니다.

✔ 해당 객체가 필요한 경우 추가로 생성할 수 없고, **오직 getInstance() 메서드를 사용하여 객체**를 불러옵니다.

- > 이 메서드를 호출하여 사용함으로써 **항상 같은 인스턴스를 사용**하게 됩니다.

```java
public class SingletonService {

    private static final SingletonService instance = new SingletonService();

    public static SingletonService getInstance(){
        return instance;
    }

    private SingletonService(){

    }

    public void logic(){
        System.out.println("싱글톤 객체 로직 호출");
    }
}
```

### 하지만! 싱글톤 패턴의 문제점

- 싱글톤 패턴을 구현하는 코드 자체가 많이 들어갑니다.
- 의존 관계상 클라이언트가 구체화된 클래스에 의존합니다. ( DIP 위반)
- 클라이언트가 구체화된 클래스에 의존해서 OCP 원칙을 위반할 가능성이 높습니다.
- 내부 속성을 변경하거나 초기화하기 어렵습니다.
- private 생성자로 인해 자식 클래스를 만들기 어렵습니다.

### 싱글톤 컨테이너

스프링 컨테이너는 위의 싱글톤 패턴의 문제점을 해결하면서, 객체 인스턴스를 싱글톤으로 관리해 줍니다!!

✔ 스프링 컨테이너는 싱글턴 패턴을 적용하지 않아도, 객체 인스턴스를 싱글톤으로 관리합니다.

```java
@Test
@DisplayName("스프링 컨테이너와 싱글톤")
void springContainer(){

    ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

		// 1. 조회: 호출할 때 마다 같은 객체를 생성
    MemberService memberService1 = ac.getBean("memberService", MemberService.class);
		// 2. 조회: 호출할 때 마다 같은 객체를 생성
    MemberService memberService2 = ac.getBean("memberService", MemberService.class);

		// 참조값이 같은 것을 확인
    System.out.println("memberService1 = " + memberService1);
    System.out.println("memberService2 = " + memberService2);

		// memberService1 == memberService2
    assertThat(memberService1).isSameAs(memberService2);
}
```

![https://blog.kakaocdn.net/dn/bGn4gb/btr6ngi7EHG/stC8NxZAYA8TOh1a0koRA1/img.png](https://blog.kakaocdn.net/dn/bGn4gb/btr6ngi7EHG/stC8NxZAYA8TOh1a0koRA1/img.png)

### 싱글톤 방식의 주의할 점

✔ 객체 인스턴스를 하나만 생성해서 공유하는 방식으로 여러 클라이언트가 같은 객체 인스턴스를 공유하기 때문에 싱글톤 객체는 상태를 유지(stateful) 하게 설계해서는 안 됩니다.

✔ **무상태(stateless)로 설계**해야 합니다.

- 특정 클라이언트에 의존적인 필드가 있으면 안 됩니다.

- 특정 클라이언트가 값을 변경할 수 있는 필드가 있으면 안됩니다.

- 읽기만 가능해야 합니다.

- 필드(field) 대신에 자바에서는 공유되지 않는 지역변수, 파라미터, ThreadLocal 등을 사용해야 합니다.

```java
public class StatefulService {
    private int price;
    public void order(String name, int price){
        System.out.println("name = " + name + " price = " + price);
        this.price = price;// 여기가 문제 - 1. 값이 변하는 필드 변수 사용 2. 값을 변경함
    }
    public int getPrice(){
        return price;
    }
}
```

Test 코드

```
class StatefulServiceTest {
    @Test
    void statefulServiceSingleton(){
        ApplicationContext ac = new AnnotationConfigApplicationContext(TestConfig.class);
        StatefulService statefulService1 = ac.getBean(StatefulService.class);
        StatefulService statefulService2 = ac.getBean(StatefulService.class);

				// ThreadA: A사용자 10000원 주문
        statefulService1.order("userA", 10000);
				// ThreadB: B사용자 20000원 주문
        statefulService2.order("userB", 20000);

				// ThreadA: 사용자A 주문 금액 조회int price = statefulService1.getPrice();
        System.out.println("price = " + price);

        Assertions.assertThat(statefulService1.getPrice()).isEqualTo(20000);
    }

    static class TestConfig{
        @Bean
        public StatefulService statefulService() {
            return new StatefulService();
        }
    }
}
```

- StatefulService의 price 필드는 공유되는 필드인데, 특정 클라이언트가 값을 변경할 수 있으므로 문제가 발생할 수 있습니다.
- userA의 주문금액은 10000원이어야 하는데, userB 사용자가 주문을 한 이후에 조회를 하면 20000원이라는 userA의 주문 금액이 나오지 않는 오류가 발생합니다.

**✔ 스프링 빈은 항상 무상태(stateless)로 설계해야합니다.**

### @Configuration과 바이트코드 조작

```
@Test
void configurationDeep() {
 ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

//AppConfig도 스프링 빈으로 등록된다.
AppConfig bean = ac.getBean(AppConfig.class);
System.out.println("bean = " + bean.getClass());
//출력: bean = class hello.core.AppConfig$$EnhancerBySpringCGLIB$$bd479d70//출력 기대값 : bean = class hello.core.AppConfig
}
```

AnnotationConfigApplicationContext에 파라미터로 넘긴 값(AppConfig.class)은 스프링 빈으로 등록됩니다.

그래서 AppConfig도 스프링 빈이 됩니다.

순수한 클래스라면 "class hello.core.AppConfig"가 출력되어야 하는데 스프링을 사용하면 다른 클래스 정보를 출력합니다.

✔ **스프링이 CGLIB라는 바이트코드 조작 라이브러리를 사용**해서, AppConfig 클래스를 상속받은 임의의 다른 클래스를 만들고, 그 다른 클래스를 통해 스프링 빈을 등록했기에

"class hello.core.AppConfig$$EnhancerBySpringCGLIB$$bd479d70" 값이 출력됩니다.

![https://blog.kakaocdn.net/dn/ygO6b/btr53IHNSV9/zTyiOItTNqP81kNWTnjKf0/img.png](https://blog.kakaocdn.net/dn/ygO6b/btr53IHNSV9/zTyiOItTNqP81kNWTnjKf0/img.png)

✔ **instance: AppConfig@CGLIB 클래스가 싱글톤이 보장**되도록 해줍니다.

AppConfig@CGLIB 예상 코드

```
@Bean
public MemberRepository memberRepository() {

 	if (memoryMemberRepository가 이미 스프링 컨테이너에 등록되어 있으면?) {
 		return 스프링 컨테이너에서 찾아서 반환;
 	} else {//스프링 컨테이너에 없으면
 		기존 로직을 호출해서 MemoryMemberRepository를 생성하고 스프링 컨테이너에 등록
 		return 반환
 	}
 }
```

✔ @Bean이 붙은 메소드마다 이미 스프링 빈이 존재하면 스프링 컨테이너에 존재하는 빈을 반환하고, 스프링 빈이 없으면 생성해서 스프링 빈을 등록하고 반환하는 코드가 동적으로 만들어집니다.

✔ 스프링 설정 정보는 항상 @Configuration 을 사용해야 합니다. @Bean만 사용하게 되면 스프링 빈은 등록할 수 있지만, 싱글톤을 보장하지 못해, 같은 객체를 요청할 때마다 생성하게 됩니다.
