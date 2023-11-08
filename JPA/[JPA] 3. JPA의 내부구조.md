## ※ JPA에서 가장 중요한 2가지

1.  객체와 관계형 데이터베이스 매핑하기 ( Object Relational Mapping ) 객체 - 데이터베이스
2.  영속성 컨텍스트 이해하기

### 1. EntityManagerFactory와 EntityManager 

![image](https://github.com/kylo-dev/Spring-Jpa-Study/assets/103489352/54ece946-563d-4c9b-bc43-9f575de76099)


✔ EntityManagerFactory는 **하나만 생성**해서 애플리케이션 전체에서 공유하여 사용합니다.

✔ EntityManager는 **Thread 간에 공유하지 않습니다.**

✔ **EntityManager를 통해서 영속성 컨텍스트에 접근**할 수 있습니다.

✔ **JPA의 모든 데이터 변경은 트랙잭션 안에서 실행**되어야 합니다.

### 2. 영속성 컨텍스트란

: "엔티티를 영구 저장하는 환경" 입니다.

```
// 엔티티를 영속성 컨텍스트에 저장하기
EntityManager.persist(entity);
```

![image](https://github.com/kylo-dev/Spring-Jpa-Study/assets/103489352/8780b6cb-843a-4dbd-8f78-11ae953dbb6e)


* Spring과 Spring-boot에서 주로 JPA를 이용해 데이터베이스 관리를 합니다.
* 스프링 프레임워크 환경에서는 엔티티 매니저와 영속성 컨텍스트 관계가 N:1 관계입니다.

(즉, 여러 엔티티 매니저에서 저장한 엔티티들을 **한 영속성 컨텍스트에 저장**하는 것을 의미합니다.)

### 3. 엔티티의 생명주기 4가지

✔ 비영속 (new/transient)

: 영속성 컨텍스트와 **전혀 관계가 없는** 새로운 상태

![image](https://github.com/kylo-dev/Spring-Jpa-Study/assets/103489352/b7922324-6876-4e2e-a18f-177f7b1f543b)


```
// 객체를 생성한 상태 (비영속)
Member member = new Member();
member.setId("memberA");
member.setUsername("AAA);
```

✔ 영속 (managed)

: 영속성 컨텍스트에 **관리**되는 상태

![image](https://github.com/kylo-dev/Spring-Jpa-Study/assets/103489352/62ec1607-9cd7-4cbf-95a9-699293ffc13d)


```
// 객체를 생성한 상태
Member member = new Member();
member.setId("memberA");
member.setUsername("AAA);

EntityManager em = emf.createEntityManager();
em.getTransaction().begin();

// 객체를 저장한 상태 (영속)
em.persist(member);
```

✔ 준영속 (detached)

: 영속성 컨텍스트에 저장되었다가 **분리**된 상태

```
// 회원 엔티티를 영속성 컨텍스트에서 분리 (준영속)
em.detach(member);
```

✔ 삭제 (removed)

: **삭제**된 상태

```
// 객체를 삭제한 상태 (삭제)
em.remove(member);
```

### 엔티티의 생명주기

![image](https://github.com/kylo-dev/Spring-Jpa-Study/assets/103489352/7e88bb17-443f-416f-96a6-eba04dd9553c)


### 4. 영속성 컨텍스트의 이점

✔ 1차 캐시 (이미 저장된 엔티티를 빠르게 조회할 수 있음)

![image](https://github.com/kylo-dev/Spring-Jpa-Study/assets/103489352/90149cf4-ac5d-47f1-95f0-6317fc4097d2)


```
// 객체를 생성한 상태
Member member = new Member();
member.setId("member1");
member.setUsername("AAA);

// 1차 캐시에 저장 (영속성 켄텍스트에 저장)
em.persist(member);

// 1차 캐시에서 조회
Member findMember = em.find(Member.class, "member1");
```

\* 데이터베이스에서 조회

![image](https://github.com/kylo-dev/Spring-Jpa-Study/assets/103489352/9f31f2c0-3e3e-4a9c-8ae6-867f3abcf3fc)


```
// "member2"는 영속성 컨텍스트에 없고, DB에 저장되어 있다는 가정
Member findMember2 = em.find(Member.class, "member2");
```

✔ 동일성(Identity) 보장

```
Member a = em.find(Member.class, "member1");
Member b = em.find(Member.class, "member1");

System.out.println( a == b ); // true
```

: 1차 캐시로 반복 가능한 읽기 등급의 트랜잭션 격리 수준을 데이터베이스가 아닌 애플리케이션 차원에서 제공합니다.

✔ 트랜잭션을 지원하는 쓰기 지연 (Transactional Write-behind)

```
EntityManager em = emf.createEntityManager();
EntityTransaction transaction = em.getTransaction();

//엔티티 매니저는 데이터 변경시 트랜잭션을 시작해야 한다.
transaction.begin(); // [트랜잭션] 시작
em.persist(memberA);
em.persist(memberB); //여기까지 INSERT SQL을 데이터베이스에 보내지 않는다.

//커밋하는 순간 데이터베이스에 INSERT SQL을 보낸다.
transaction.commit(); // [트랜잭션] 커밋
```
![image](https://github.com/kylo-dev/Spring-Jpa-Study/assets/103489352/7386ab6d-bcff-4dfb-ba3a-65541ca05390)

![image](https://github.com/kylo-dev/Spring-Jpa-Study/assets/103489352/d6236f49-8be5-491d-a8f1-337700a4bbd9)


✔ 변경 감지 (Dirty Checking)

![image](https://github.com/kylo-dev/Spring-Jpa-Study/assets/103489352/7d5e7a59-d419-4a87-b72e-cdfc99db049f)


스냅샷 공간에 초기에 설정된 엔티티 값이 저장되어 있습니다.

set 함수를 통해 엔티티 값을 변경하면 1차 캐시에 정보가 변경되고,

**트랜잭션 commit** 시 **Entity와 스냅샷의 값을 비교**하여 변경된 사항을 반영합니다.

```
// 영속 엔티티 조회
Member memberA = em.find(Member.class, "memberA");

// 영속 엔티티 데이터 수정
memberA.setUsername("new name");
memberA.setAge(100);

transaction.commit(): // [트랜잭션] 커밋 -> 변경 사항 파악후, DB에 반영
```

### 플러시란

✔ 영속성 컨텍스트의 변경 내용을 데이터베이스에 반영

* 변경 감지
* 수정된 엔티티 쓰기 지연 SQL 저장소에 등록
* 쓰기 지연 SQL 저장소의 쿼리를 데이터베이스에 전송

영속성 컨텍스트를 플러시하는 방법
* em.flush() - 직접 호출
* 트랜잭션 커밋 - 플러시 자동 호출

#### flush 더 알아보기

* 변경 내용을 데이터베이스 반영하고, 영속성 컨텍스트를 비우지는 않음
* 영속성 컨텍스트의 변경 내용을 데이터베이스에 동기화 해줌
* 트랜잭션 작업 단위로 실행 됨 
