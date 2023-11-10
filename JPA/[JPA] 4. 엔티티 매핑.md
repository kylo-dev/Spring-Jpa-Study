## 객체와 테이블 매핑

### @Entity

- @Entity가 붙은 클래스는 JPA가 관리, 엔티티라고 부름

주의

- 기본 생성자가 필수로 있어야 한다. (파라미터가 없는 public or protected)
- final, enum, interface, inner 클래스 사용 불가능
- 저장할 필드에 final 사용 불가능

### @Entity 속성 정리

속성 : name

- JPA에서 사용할 엔티티 이름을 지정
- 기본값 : 클래스 이름을 그대로 사용

## 데이터베이스 스키마 자동 생성

- DDL을 애플리케이션 실행 시점에 자동 생성
- 테이블 중심에서 객체 중심으로 생성
- 데이터베이스 방언을 활용해서 데이터베이스에 맞는 적절한 DDL 생

![image](https://github.com/kylo-dev/Spring-Jpa-Study/assets/103489352/e4c16283-d30b-4675-97bf-10367962cf1a)


주의

- 개발 초기 단계는 create 또는 update
- 테스트 서버는 update 또는 validate
- 운영 서버는 validate 또는 none

### DDL 생성 기능 - 제약조건

- @Column(nullable = false, length = 10)

## 필드와 컬럼 매핑

```java
@Entity 
public class Member { 
	 @Id 
	 private Long id; 

	 @Column(name = "name") 
	 private String username; 

	 private Integer age; 

	 @Enumerated(EnumType.STRING) 
	 private RoleType roleType; 

	 @Temporal(TemporalType.TIMESTAMP) 
	 private Date createdDate; 

	 @Temporal(TemporalType.TIMESTAMP) 
	 private Date lastModifiedDate; 

	 @Lob 
	 private String description; 
	 //Getter, Setter… 
}
```

### 매핑 어노테이션 정리

- @Column - 컬럼 매핑
- @Temporal - 날짜 타입 매핑
- @Enumerated - enum 타입 매핑
- @Lob - BLOB, CLOB 매핑
- @Transient - 특정 필드를 컬럼에 매핑하지 않음

### @Column

- name : 필드와 매핑할 테이블의 컬럼 이름
- nullable(DDL) : null 값의 허용 여부를 설정. false로 설정하면 DDL 생성 시에 not null 제약조건이 붙는다.
- unique(DDL) : 한 컬럼에 간단히 유니크 제약조건을 걸 때 사용한다.
- legnth(DDL) : 문자 길이 제약조건, String 타입에만 사용

### @Enumerated

- 자바 enum 타입을 매핑할 때 사용

> 주의! 무조건 **`EnumType.STRING`** 사용하기
> 

속성 value

- EnumType.ORDINAL : enum 순서를 데이터베이스에 저장
- EnumType.STRING : enum 이름을 데이터베이스에 저장

### @Temporal

- 날짜 타입(java.util.Date, java.util.Calendar)을 매핑할 때 사용

> LocalDate, LocalDateTime을 사용할 때는 생략 가능
> 

## 기본 키 매핑

```java
@Id @GeneratedValue(strategy = GenerationType.AUTO)
private Long id;
```

자동 생성(@GeneratedValue)
• IDENTITY: 데이터베이스에 위임, MYSQL
• SEQUENCE: 데이터베이스 시퀀스 오브젝트 사용, ORACLE
• @SequenceGenerator 필요
• TABLE: 키 생성용 테이블 사용, 모든 DB에서 사용
• @TableGenerator 필요
• AUTO: 방언에 따라 자동 지정, 기본값

## 권장하는 식별자 전략

- 기본 키 제약조건 : null 아님, 유일성, 불변성
- 권장 : Long형 + 대체키 + 키 생성전략 사용
