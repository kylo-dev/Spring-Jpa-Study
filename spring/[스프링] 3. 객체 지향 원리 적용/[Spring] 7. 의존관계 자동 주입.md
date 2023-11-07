## 다양한 의존관계 주입 방법

의존관계 주입은 크게 4가지 방법이 있습니다.

- 생성자 주입 ⭐
- 수정자 주입(setter 주입) ❎
- 필드 주입
- 일반 메서드 주입 ❎

### 생성자 주입

- 생성자를 통해서 의존 관계를 주입 받음
- 특징
    - 생성자 호출 시점에 딱 1번만 호출되는 것이 보장
    - 불변, 필수 의존관계에 사용

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

> 생성자가 딱 1개만 있으면 @Autowired를 생략해도 자동 주입 됩니다. (스프링 빈에만 해당)
> 

### 필드 주입

- 필드에 바로 주입하는 방법
- 특징
    - 코드가 간결해서 좋아보이지만 외부에서 변경이 불가능해서 테스트 하기 힘들다
    - DI 프레임워크가 없으면 아무것도 할 수 없다.

```java
@Component
public class OrderServiceImpl implements OrderService {
	 @Autowired
	 private MemberRepository memberRepository;

	 @Autowired
	 private DiscountPolicy discountPolicy;
}
```

### 생성자 주입을 선택해서 사용하기!!

**불변**

- 대부분의 **의존관계 주입은 한번 일어나면 애플리케이션 종료시점까지 의존관계를 변경할 일이 없습니다.**
    - 오히려 대부분의 의존관계는 애플리케이션 종료 전까지 변하면 안됩니다. (불변해야 한다.)
- 수정자 주입을 사용하면, setXxx 메서드를 public으로 열어두어야 한다.
- 누군가 실수로 변경할 수 도 있고, 변경하면 안되는 메서드를 열어두는 것은 좋은 설계 방법이 아니다.
- **생성자 주입은 객체를 생성할 때 딱 1번만 호출되므로 이후에 호출되는 일이 없다**. 따라서 불변하게 설계할 수 있습니다.

**final 키워드**

생성자 주입을 사용하면 필드에 final 키워드를 사용할 수 있습니다.

생성자에서 혹시라도 값이 설정되지 않는 오류를 컴파일 시점에서 발견할 수 있습니다.

## 롬복과 최신 트렌드

막상 개발을 하다보면, 대부분이 다 불변이고 , 그래서 필드에 final 키워드를 사용합니다.

롬복을 이용한 최적화

```java
@Component
@RequiredArgsConstructor
public class OrderServiceImpl implements OrderService {
	 private final MemberRepository memberRepository;
	 private final DiscountPolicy discountPolicy;
}
```

- 롬복 라이브러리가 제공하는 **`@RequiredArgsConstructor`** 기능을 사용하면 final이 붙은 필드를 모아서 생성자를 자동으로 만들어줍니다. (다음 코드에는 보이지 않지만 실제 호출 가능합니다.)

## 조회되는 빈이 2개 이상일 때 해결하는 방법

- @Autowired 필드 명 매칭
- @Qualifier @Qualifier끼리 매칭 빈 이름 매칭
- @Primary 사용

### @Autowired 필드 명 매칭

- @Autowired는 타입 매칭을 시도하고, 이때 여러 빈이 있으면 필드 이름, 파라미터 이름으로 빈 이름을 추가 매칭합니다.

```java
//기존 코드
@Autowired
private DiscountPolicy discountPolicy;

//필드 명을 빈 이름으로 변경
@Autowired
private DiscountPolicy rateDiscountPolicy;
```

### @Qualifier 사용

- 추가 구분자를 붙여주는 방법입니다.

```java
//빈 등록시 @Qualifier를 붙여 준다.
@Component
@Qualifier("mainDiscountPolicy")
public class RateDiscountPolicy implements DiscountPolicy {}

@Component
@Qualifier("fixDiscountPolicy")
public class FixDiscountPolicy implements DiscountPolicy {}
```

**주입 시에 @Qualifier를 붙여주고 등록한 이름을 적어줍니다.**

```java
//생성자 자동 주입 예시
@Autowired
public OrderServiceImpl(MemberRepository memberRepository, @Qualifier("mainDiscountPolicy") DiscountPolicy discountPolicy) {
	 this.memberRepository = memberRepository;
	 this.discountPolicy = discountPolicy;
}

//수정자 자동 주입 예시
@Autowired
public DiscountPolicy setDiscountPolicy(@Qualifier("mainDiscountPolicy") DiscountPolicy discountPolicy) {
	 this.discountPolicy = discountPolicy;
}
```

**@Qualifier 로 주입할 때 @Qualifier("mainDiscountPolicy") 를 못찾으면 어떻게 될까?**

그러면 mainDiscountPolicy라는 이름의 스프링 빈을 추가로 찾는다. 하지만 경험상 @Qualifier 는
@Qualifier 를 찾는 용도로만 사용하는게 명확하고 좋습니다.

### @Primary 사용

- 우선순위를 정하는 방법

```java
@Component
@Primary
public class RateDiscountPolicy implements DiscountPolicy {}

@Component
public class FixDiscountPolicy implements DiscountPolicy {}
```

## 조회한 빈이 모두 필요할 때 List, Map

예를 들어서, 할인 서비스를 제공하는데, 클라이언트가 할인의 종류(rate, fix)를 선택할 수 있는 경우

```java
public class AllBeanTest {
	 @Test
	 void findAllBean() {
	 ApplicationContext ac = new AnnotationConfigApplicationContext(AutoAppConfig.class, DiscountService.class);
	 DiscountService discountService = ac.getBean(DiscountService.class);

	 Member member = new Member(1L, "userA", Grade.VIP);
	 int discountPrice = discountService.discount(member, 10000, "fixDiscountPolicy");

	 assertThat(discountService).isInstanceOf(DiscountService.class);
	 assertThat(discountPrice).isEqualTo(1000);
	 }

	 static class DiscountService {
		 private final Map<String, DiscountPolicy> policyMap;
		 private final List<DiscountPolicy> policies;

		 public DiscountService(Map<String, DiscountPolicy> policyMap, List<DiscountPolicy> policies) {
			 this.policyMap = policyMap;
			 this.policies = policies;

			 System.out.println("policyMap = " + policyMap);
			 System.out.println("policies = " + policies);
		 }

		 public int discount(Member member, int price, String discountCode) {
			 DiscountPolicy discountPolicy = policyMap.get(discountCode);

			 System.out.println("discountCode = " + discountCode);
			 System.out.println("discountPolicy = " + discountPolicy);

			 return discountPolicy.discount(member, price);
			 }
		 }
}
```

**로직 분석**

- DiscountService는 Map으로 모든 DiscountPolicy 를 주입받는다. 이때 fixDiscountPolicy ,
rateDiscountPolicy 가 주입됩니다.
- discount () 메서드는 discountCode로 "fixDiscountPolicy"가 넘어오면 map에서
fixDiscountPolicy 스프링 빈을 찾아서 실행합니다.

**주입 분석**

- Map<String, DiscountPolicy> : map의 키에 스프링 빈의 이름을 넣어주고, 그 값으로DiscountPolicy 타입으로 조회한 모든 스프링 빈을 담아줍니다.
- List<DiscountPolicy> : DiscountPolicy 타입으로 조회한 모든 스프링 빈을 담아준다.
만약 해당하는 타입의 스프링 빈이 없으면, 빈 컬렉션이나 Map을 주입합다
