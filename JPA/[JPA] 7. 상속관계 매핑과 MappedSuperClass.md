## 상속관계 매핑

- 관계형 데이터베이스에는 상속 관계가 없습니다.
- 객체의 상속 구조와 DB의 슈퍼타입 - 서브타입 관계를 매핑

![image](https://github.com/kylo-dev/Spring-Jpa-Study/assets/103489352/4f267ceb-e58a-495f-999a-752f2c6e37a2)


### 슈퍼타입-서브타입 논리 모델을 물리 모델로 구현하는 3가지 방법

1. **조인 전략** : 각각 테이블로 변환
2. **단일 테이블 전략** : 통합 테이블로 변환
3. 구현 클래스마다 테이블 전략 : 서브 타입 테이블로 변환 (비추천)

### 주요 어노테이션

@Inheritance(strategy=InheritanceType.XXX)

- JOINED : 조인 전략
- SINGLE_TABLE : 단일 테이블 전략
- TABLE_PER_CLASS : 구현 클래스마다 테이블 전략

슈퍼 타입 엔티티

- @DiscriminatorColumn(name=’DTYPE’)
- 부모 클래스에 선언합니다. 하위 클래스를 구분하는 용도의 컬럼입니다. 관례는 default = DTYPE

서브 타입 엔티티

- @DiscriminatorValue(’XXX’) → DTYPE명 작성

## 조인 전략

![image](https://github.com/kylo-dev/Spring-Jpa-Study/assets/103489352/bb39077e-b107-4b44-9699-d7d579646e26)


```java
@Entity@Inheritance(strategy = InheritanceType.JOINED) 
// 상속 구현 전략 선택
public class Item {    
		@Id    
		@GeneratedValue(strategy = GenerationType.IDENTITY)    
		private Long id;    
		private String name;    
		private int price;
}

```

```java
@Entity
public class Movie extends Item {    
		private String director;    
		private String actor;
}

```

```java
Hibernate:     
create table Album (       
artist varchar(255),        
id bigint not null,        
primary key (id)    
)

Hibernate:     
create table Book (       
author varchar(255),        
isbn varchar(255),        
id bigint not null,        
primary key (id)    
)

Hibernate:     
create table Item (       
DTYPE varchar(31) not null,        
id bigint generated by default as identity,        
name varchar(255),        price integer not null,        
primary key (id)    
)

Hibernate:     
create table Movie (       
actor varchar(255),        
director varchar(255),        
id bigint not null,        
primary key (id)    
)

Hibernate:     
alter table Album        
add constraint FKcve1ph6vw9ihye8rbk26h5jm9        
foreign key (id)        references Item

Hibernate:     
alter table Book        
add constraint FKbwwc3a7ch631uyv1b5o9tvysi        
foreign key (id)        
references Item

Hibernate:     
alter table Movie        
add constraint FK5sq6d5agrc34ithpdfs0umo9g        
foreign key (id)        
references Item

```

### 장점

- 테이블 정규화
- 외래 키 참조 무결성 제약조건 활용가능
- 저장공간 효율화

### 단점

- 조회시 조인을 많이 사용, 성능 저하
- 조회 쿼리가 복잡함
- 데이터 저장시 INSERT SQL 2번 호출

## 단일 테이블 전략

![image](https://github.com/kylo-dev/Spring-Jpa-Study/assets/103489352/2b41914f-00ce-4e45-98cd-019a0a8102e7)


```java
@Entity
@DiscriminatorColumn@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
public class Item {    
		@Id    
		@GeneratedValue(strategy = GenerationType.IDENTITY)    
		private Long id;    
		private String name;    
		private int price;
}
```

```java
Hibernate:     
create table Item (       
		DTYPE varchar(31) not null,        
		id bigint generated by default as identity,        
		name varchar(255),        
		price integer not null,        
		artist varchar(255),        
		author varchar(255),        
		isbn varchar(255),        
		actor varchar(255),        
		director varchar(255),        
		primary key (id)    
)
```

### 장점

- 서비스 규모가 크지 않고, 굳이 조인 전략을 선택해서 복잡하게 갈 필요가 없다고 판단 될 때에는한 테이블에 다 저장하고, DTYPE으로 구분하는 단일 테이블 전략을 선택할 수 있다.
- INSERT 쿼리도 한 번, SELECT 쿼리도 한 번이다. 조인할 필요가 없고, 성능이 좋다.

### 단점

- 자식 엔티티가 매핑한 컬럼은 모두 null을 허용한다.
- 단일 테이블에 모든 것을 저장하므로 테이블이 커질 수 있습니다.

---

## @MappedSuperClass

- 공통 매핑 정보가 필요할 때 사용합니다.
- 주로 등록일, 수정일, 등록자, 수정자 같은 전체 엔티티에서 공통으로 적용하는 정보를 저장합니다.

### 특징

- 엔티티가 아니며, 테이블과 매핑되지 않습니다.
- 부모 클래스를 상속받는 자식 클래스에 매핑 정보만 제공합니다.
- 직접 생성해서 사용할 일이 없으므로 **추상 클래스**를 권장합니다.

```java
@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
@Getter
public abstract class BaseEntity {
    @CreatedDate
    private LocalDateTime createdAt;

    @LastModifiedDate
    private LocalDateTime updatedAt;
}
```

- @EntityListeners(AuditingEntityListener.class) 를 사용하는 경우 main 함수에 @EnableJpaAuditing 어노테이션을 추가해주어야 합니다.

```java
@SpringBootApplication
@EnableJpaAuditing
public class Application {

	public static void main(String[] args) {
		SpringApplication.run(Application.class, args);
	}

}
```
