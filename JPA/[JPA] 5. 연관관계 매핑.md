목표

- 객체와 테이블 연관관계의 차이 이해
- 객체의 참조와 테이블의 외래 키 매핑 이해
- 단방향, 양방향 이해
- 다중성 : 다대일, 일대다, 다대다 이해
- **`연관관계의 주인`**

## 연관관계가 필요한 이유

### 객체를 테이블에 맞추어 모델링

(연관관계가 없는 객체 / 사용 X)

![image](https://github.com/kylo-dev/Spring-Jpa-Study/assets/103489352/58486588-0a87-4dcc-a6e2-e39a7c5961c9)


```java
@Entity
 public class Member { 
	 @Id @GeneratedValue
	 private Long id;

	 @Column(name = "USERNAME")
	 private String

	@Column(name = "TEAM_ID")
	 private Long teamId;
 } 

 @Entity
 public class Team {
	 @Id @GeneratedValue
	 private Long id;

	 private String name; 
 }
```

참조 대신에 외래 키를 그대로 N측에서 관리합니다.

팀과 회원 저장 과정

```java
//팀 저장
 Team team = new Team();
 team.setName("TeamA");
 em.persist(team);

 //회원 저장
 Member member = new Member();
 member.setName("member1");
 member.setTeamId(team.getId());
 em.persist(member);
```

외래 키 식별자를 직접 다루어야 해서 비효율적임

```java
//조회
Member findMember = em.find(Member.class, member.getId()); 
Long findTeamId = findMember.getTeamId

//연관관계가 없음
Team findTeam = em.find(Team.class, team.getId())
```

- 식별자로 다시 조회, 객체 지향적인 방법이 아님

> 객체를 테이블에 맞추어 데이터 중심으로 모델링하면, 협력 관계를 만들 수없습니다.
> 

⇒ 참조를 사용해서 연관된 객체를 찾도록 바꿔줍니다.

### 단방향 연관관계

![image](https://github.com/kylo-dev/Spring-Jpa-Study/assets/103489352/ac182750-1daa-4e0b-87f7-487105139a79)

```java
@Entity
 public class Member { 
	 @Id @GeneratedValue
	 private Long id;

	 @Column(name = "USERNAME")
	 private String name;

	 private int age;

	 @ManyToOne
	 @JoinColumn(name = "TEAM_ID")
	 private Team team;
}
```

- 객체의 참조와 테이블의 외래 키를 매핑합니다.

단방향 연관관계이므로 Member 측에서만 Team을 조회할 수 있습니다.

![image](https://github.com/kylo-dev/Spring-Jpa-Study/assets/103489352/2d583af8-51b1-4ba8-bb9c-3859a568e9ff)

```java
//팀 저장
 Team team = new Team();
 team.setName("TeamA");
 em.persist(team);

 //회원 저장
 Member member = new Member();
 member.setName("member1");
 member.setTeam(team); //단방향 연관관계 설정, 참조 저장
 em.persist(member);

//조회
 Member findMember = em.find(Member.class, member.getId()); 

//참조를 사용해서 연관관계 조회
 Team findTeam = findMember.getTeam();

// 새로운 팀B
 Team teamB = new Team();
 teamB.setName("TeamB");
 em.persist(teamB);

 // 회원1에 새로운 팀B 설정
 member.setTeam(teamB);
```

### 양방향 연관관계와 연관관계의 주인

![image](https://github.com/kylo-dev/Spring-Jpa-Study/assets/103489352/9707c96f-d1d1-49fe-a1ae-c0c25bbf6d33)

- Member와 Team 엔티티에서 서로 양방향으로 참조하고 있어 양측에서 조회가 가능합니다.

```java
@Entity
 public class Member { 
	 @Id @GeneratedValue
	 private Long id;

	 @Column(name = "USERNAME")
	 private String name;

	 private int age;

	 @ManyToOne
	 @JoinColumn(name = "TEAM_ID")
	 private Team team;
}

@Entity
 public class Team {
	 @Id @GeneratedValue
	 private Long id;
	
	 private String name;
	
	 @OneToMany(mappedBy = "team")
	 List<Member> members = new ArrayList<Member>();
 }

//조회
Team findTeam = em.find(Team.class, team.getId()); 
int memberSize = findTeam.getMembers().size(); //역방향 조회
```

`@OneToMany(mappedBy = "team")`

`List<Member> members = new ArrayList<Member>();`

이 부분에서 mappedBy=”team”이 의미하는 것은 Member 엔티티의 team 속성과 연결함을 의미합니다.

### 객체와 테이블이 관계를 맺는 차이

- 객체는 연관관계가 2개
    - Member → Team
    - Team → Member
- 테이블 연관관계는 1개
    - Member ↔ Team (FK)

![image](https://github.com/kylo-dev/Spring-Jpa-Study/assets/103489352/09b02793-b06c-4ab7-aa2b-3eece575ff5e)


테이블에서는 외래키를 통해 (조인) 관계를 맺지만, 객체는 참조를 통해서 다른 객체와 매핑하여 조회합니다.

### 연관관계의 주인 (⭐)

- 객체의 두 관계 중 하나를 연관관계의 주인으로 지정해야 합니다.
- 연관관계의 주인만이 외래 키를 관리합니다.
- 주인이 아닌 쪽은 읽기만 가능합니다.
    - 주인은 mappedBy 속성 사용 X
    - 주인이 아니면 mappedBy 속성으로 주인 지정

### 누구를 주인으로 지정?

- 외래 키가 있는 곳을 주인으로 정합니다.
- Member.team이 두 관계에 있어서 연관관계의 주인
- 주로, 외래키와 비슷하게 N측이 주인이 됨

### 양방향 연관관계 주의

1. 순수 객체 상태를 고려해서 항상 양쪽에 값을 설정
2. 연관관계 편의 메소드 생성하기

## 객체 매핑 연습하기

![image](https://github.com/kylo-dev/Spring-Jpa-Study/assets/103489352/998ab901-99dc-4447-91b5-6cf90d8722a8)


테이블 구조 (외래키를 이용해 매핑)

![image](https://github.com/kylo-dev/Spring-Jpa-Study/assets/103489352/b4960e27-c508-4d19-af3d-effb2996646a)


객체 구조 (참조를 이용해 매핑 / N측에서 외래키 관리 - 주인이 아닌 측에서 컬렉션 설정
