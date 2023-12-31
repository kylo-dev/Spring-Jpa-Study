# [스프링] 3. 객체 지향 원리 적용

## 새로운 할인 정책 개발

할인 정책을 고정 금액 할인에서 정률 할인 정책으로 변경하고자 한다.

> 객체지향 설계 원칙을 준수하여 OCP, DIP 규칙을 지켜보자

### RateDiscountPolicy 추가

![Untitled](https://github.com/kylo-dev/Spring-Jpa-Study/assets/103489352/3d91299e-87ad-4d41-8456-85af23a0e188)


OrderServiceImpl은 DiscountPolicy 역할(인터페이스)에 의존하고 있으며,

RateDiscountPolicy는 DiscountPolicy를 구현하고 있어 의존관계를 설정해주면 된다.

## 새로운 할인 정책 적용과 문제점

```java
public class OrderServiceImpl implements OrderService {
	// private final DiscountPolicy discountPolicy = new FixDiscountPolicy();
	 private final DiscountPolicy discountPolicy = new RateDiscountPolicy();
}
```

문제점 파악하기

⭕ : 역할과 구현을 분리했는가,

⭕ - 다형성을 활용하고, 인터페이스와 구현 객체를 분리했는가,

❌ - OCP, DIP 객체지향 설계 원칙을 준수했는가.

DIP : 주문 서비스 클라이언트(OrderServiceImpl)은 DiscountPolicy 인터페이스에 의존하면서 구현 클래스에도 의존하고 있다. (= 할인 정책이 바뀌니 의존하고 있던 구현 클래스도 변경한다,)

OCP : 변경하지 않고 확장할 수 있어야 하는데 사용되는 클라이언트 코드에 영향을 주고 있다.

![Untitled 1](https://github.com/kylo-dev/Spring-Jpa-Study/assets/103489352/10df623f-55e1-4ee9-8caa-4ff5d66f1c07)


바람직한 의존관계가 아님

### DIP 위반 해결 방법

클라이언트 클래스가 추상 클래스에만 의존하도록 변경한다.

![Untitled 2](https://github.com/kylo-dev/Spring-Jpa-Study/assets/103489352/d55771d5-db03-4dd7-9e3a-dd57ffd5d433)


```java
public class OrderServiceImpl implements OrderService {
	 //private final DiscountPolicy discountPolicy = new RateDiscountPolicy();
	 private DiscountPolicy discountPolicy;
}
```

(AppConfig, IoC 컨테이너, DI 컨테이너, 스프링 컨테이너) 누군가가 클라이언트인 OrderServiceImpl에 DiscountPolicy의 구현 객체를 대신 생성하고 주입해 주어 해결합니다.

## 관심사의 분리

- 애플리케이션을 하나의 공연으로 생각하자.
- 배우는 본인의 역할인 배역을 수행하는 것에만 집중한다.
    - 남자 주인공은 어떤 여자 주인공이 선택되더라도 똑같이 공연할 수 있어야 한다.
- 공연을 구성하고, 담당 배우를 섭외하고, 역할에 맞는 배우를 지정하는 책임을 담당하는 별도의 공연 기획자를 구한다.
- 배우와 공연 기획자의 책임을 확실히 분리하여 진행한다.

### AppConfig

- 애플리케이션의 전체 동작 방식을 구성하기 위해, 구현 객체를 생성하고, 연결하는 책임을 가지는 별도의 설정 클래스이다.

```java
public class AppConfig {
	 public MemberService memberService() {
		 return new MemberServiceImpl(memberRepository());
	 }

	 public OrderService orderService() {
		 return new OrderServiceImpl(
		 memberRepository(),
		 discountPolicy());
	 }

	 public MemberRepository memberRepository() {
		 return new MemoryMemberRepository();
	 }

	 public DiscountPolicy discountPolicy() {
		 return new FixDiscountPolicy();
	 }
}
```

### MemberServiceImpl - 생성자 주입

```java
public class MemberServiceImpl implements MemberService {
	 private final MemberRepository memberRepository;

	 public MemberServiceImpl(MemberRepository memberRepository) {
		 this.memberRepository = memberRepository;
	 }

	 public void join(Member member) {
		 memberRepository.save(member);
	 }

	 public Member findMember(Long memberId) {
		 return memberRepository.findById(memberId);
	 }
}
```

- AppConfig는 생성한 객체 인스턴스의 참조를 생성자를 통해서 주입해줍니다.
- 설계 변경으로 MemberServiceImpl은 MemoryMemberRepository를 의존하지 않고 MemberRepository 인터페이스에만 의존합니다.
- MemberServcieImpl 입장에서는 어떤 구현 객체가 들어올지 알 수 없으며 실행에만 집중할 수 있습니다.

![Untitled 3](https://github.com/kylo-dev/Spring-Jpa-Study/assets/103489352/418f0932-d13f-4cab-b9ef-19abf6e934d3)


결론

- 객체의 생성과 연결은 AppConfig가 담당한다,
- DIP 해결 : MemberServiceImpl은 MemberRepository 인터페이스에만 의존한다.
- 관심사 분리 : 객체를 생성하고 연결하는 역할과 실행하는 역할이 완전히 분리되었다.

### DI란 무엇인가

DI란 의존 관계 주입 또는 의존성 주입으로, 위의 그림에서 AppConfig 객체가 MemoryMemberRepository 객체를 생성하고 그 참조값을 MemberServiceImpl을 생성하면서 생성자로 전달하는 것을 의미합니다.

## 새로운 구조와 할인 정책 적용

- 정액 할인 정책을 정률 할인 정책으로 변경하자
- AppConfig을 통해 애플리케이션을 사용 영역과 객체를 생성하고 구성하는 영역으로 분리 됨

![Untitled 4](https://github.com/kylo-dev/Spring-Jpa-Study/assets/103489352/b8b716aa-4128-4f30-9f06-546106f39316)


```java
public class AppConfig {
	 public MemberService memberService() {
		 return new MemberServiceImpl(memberRepository());
	 }

	 public OrderService orderService() {
		 return new OrderServiceImpl(
		 memberRepository(),
		 discountPolicy());
	 }

	 public MemberRepository memberRepository() {
		 return new MemoryMemberRepository();
	 }

	 public DiscountPolicy discountPolicy() {
		// return new FixDiscountPolicy();
		 return new RateDiscountPolicy();
	 }
}
```

- 이제 할인 정책을 변경해도, 애플리케이션의 구성 역할을 담당하는 AppConfig만 변경하면 된다.
- 클라이언트 코드인 OrderServiceImpl을 포함해서 사용 영역의 어떤 코드도 변경할 필요가 없다. (= OCP 해결)

---

## 좋은 객체 지향 설계의 5가지 원칙 적용

여기서 3가지 SRP, DIP, OCP 적용

### SRP(Single Responsibility Principle) 단일 원칙 책임

한 클래스는 하나의 책임만 가져야 한다.

- (이전) 클라이언트 객체는 직접 구현 객체를 생성하고, 연결하고, 실행하는 다양한 책임을 가지고 있음
- SRP 단일 책임 원칙을 따르고 관심사 분리
- 구현 객체를 생성하고 연결하는 책임은 AppConfig가 담당하게 됨
- 클라이언트 객체는 실행하는 책임만 담당

### DIP (Dependency Inversion Principle) 의존 관계 역전 원칙

추상화에 의존해야 하며, 구체화에 의존하면 안된다.

- AppConfig가 FixDiscountPolicy 객체 인스턴스를 클라이언트 코드 대신 생성해서 클라이언트 코드에 의존관계를 주입했다.
- 클라이언트 코드는 DiscountPolicy 추상화 인터페이스에만 의존하고 있다.

### OCP (Open/Closed Principle) 개방 폐쇄 원칙

소프트웨어 요소는 확장에는 열려 있으나, 변경에는 닫혀 있어야 한다.

- 애플리케이션을 사용 영역과 구성 영역으로 나눔
- AppConfig가 의존 관계를 FixDiscountPolicy 에서 RateDiscountPolicy로 변경해서 클라이언트 코드에 주입하므로 클라이언트 코드는 변경하지 않아도 됨

## IoC, DI, 컨테이너

### IoC (Inversion of Control) 제어의 역전

- AppConfig를 통해 구현 객체는 자신의 로직을 실행하는 역할만 담당하며, 프로그램의 제어 흐름은 이제 AppConfig가 가져갑니다.
- 즉, 프로그램에 대한 제어 흐름에 대한 권한은 AppConfig가 가지고 있습니다.
- 프로그램의 제어 흐름을 직접 제어하는 것(클라이언트 코드에서)이 아니라 외부(AppConfig)에서 관리하는 것을 의미합니다.

### DI (Dependency Injection) 의존관계 주입

- 애플리케이션 실행 시점에 외부에서 실제 구현 객체를 생성하고 클라이언트에 전달해서 클라이언트와 서버의 실제 의존관계가 연결되는 것을 의미합니다.
- 객체 인스턴스를 생성하고, 그 참조값을 전달해서 연결합니다.
- 정적인 클래스 의존관계를 변경하지 않고, 동적인 객체 인스턴스 의존관계를 쉽게 변경할 수 있습니다.

### 컨테이너

- AppConfig 처럼 객체를 생성하고 관리하면서 의존관계를 연결해주는 것을 IoC 컨테이너 & DI 컨테이너 라고 부름

나중에  스프링 컨테이너를 통해 관리됨.
