# 자바 ORM 표준 JPA프로그래밍

#### **이 페이지는 인프런의 자바 ORM 표준 JPA 프로그래밍 - 기본편 (김영한님)의 강의를 정리한 내용입니다.**

# 목차

-   [JPA 소개](#jpa)
-   [영속성 관리](#영속성-관리)
-   [엔티티 매핑](#엔티티-매핑)
-   [연관관계 매핑 기초](#연관관계-매핑-기초)
-   [다양한 연관관계 매핑](#다양한-연관관계-매핑)
-   [고급 매핑](#고급-매핑)
-   [프록시와 연관관계 정리](#프록시와-연관관계-정리)
-   [값 타입](#값-타입)
-   [JPQL (객체지향 쿼리 언어)](#jpql-객체지향-쿼리-언어)
-   [Fetch Join (패치 조인)](#fetch-join-패치-조인)

# JPA

**Java Persistent API**

JPA는 Java Persistent API의 줄임말로써, 자바 진영에서 사용하는 ORM 기술의 표준을 의미한다.

**ORM?**

ORM은 Object-Relational Mapping의 약자로써, 객체 관계 매핑을 의미한다.
객체지향과 관계형 데이터베이스는 서로 추구하는 방향이 다르기 때문에 패러다임의 불일치가 발생한다. ORM을 이용하면 객체는 객체대로, DB는 DB대로 설계하고 ORM 프레임워크가 중간에서 이를 해결해 줄 수 있다.

**JPA는 애플리케이션과 JDBC사이에서 동작한다.**

자바에서는 데이터베이스를 추상화한 JDBC API가 존재한다. JPA는 JDBC API가 아닌 다른 무언가를 사용하는게 아니다. 단지 Java와 JDBC 사이에서 ORM의 역할을 맡은 하나의 기술이라고 생각해야한다.

**JPA는 표준 명세이다.**

JPA는 인터페이스의 모음이며, JPA자체로 실제 사용할 수 있는 기술이 아니다. JPA는 단지 "JPA는 이렇게 동작해야한다!" 라는 스펙이며 JPA를 이용하고싶다면 여러가지의 JPA 구현체중 한가지를 선택해서 사용해야한다.

JPA의 구현체는 Hibernate, EclipseLink, DataNucleus 등이 존재한다. 하지만 거의 Hibernate를 사용하는 편이다.

**JPA를 사용하면 좋은점?**

1. SQL중심적인 개발이 아닌, **객체지향 중심의 개발을 시도할 수 있다**. 기존 ORM이 없다면 패러다임의 불일치로 인해, 객체가 SQL에 의존적인 코드를 작성해야했지만, JPA를 이용하게되면서 그럴일이 많이 줄어들었다.
2. 시간이 곧 생산성으로 이어지는 개발자에게 테이블이 생길때마다 CRUD 쿼리와 코드를 작성하는 일은 지루하고, 생산성이 떨어진다. **JPA를 이용하면 JPA가 SQL의 쿼리를 직접 생성하기 때문에 개발자의 입장에서 생산성이 향상된다.**
3. 테이블의 명세가 바뀌면, 그에 따라 쿼리도 변경된다. JPA를 이용하면 이러한 면에서 유지보수가 굉장히 쉬워진다.
4. Layered Architecture에서는 다른 계층을 신뢰해서 사용해야하고, 다른 계층의 내부 구현을 신경쓸 필요가 없다. 하지만 **SQL중심적인 개발에서는 DB계층이 넘긴 객체의 그래프를 어디까지 탐색할 수 있을까에 대한 의문이 생긴다**. 즉 개발자는 DB계층에 대해 신뢰를 하기 힘들어진다. 하지만 JPA를 이용하면 객체 그래프 탐색이 어디까지 가능한가에 대한 의문을 하지 않아도 된다.

# 영속성 관리

### Persistence Context

![JPA%20b10287beceee42bcb0a2503671f0b00e/Untitled.png](../assets/jpa_orm_programming_basic_persistence_context.png)

**EntityManagerFactory**

EntityManagerFactory는 persistence 설정 정보를 통해서 생성된다. Persistence-Unit은 애플리케이션에서 접근하려고 하는 DB를 의미하며, DB와 통신하기 위해서는 EntityManagerFactory로 부터 EntityManager를 얻어야한다.

-   EntityManagerFactory의 생성비용은 굉장히 크다. 그렇기 때문에 애플리케이션 실행간 한번만 생성하며, 애플리케이션의 종료시 닫는다.
-   EntityManagerFactory는 Multi-Thread-Safe하다. 그렇기때문에 여러쓰레드에서 해당 객체를 참조하여, EntityManager를 생성한다.

**EntityManager**

EntityManager는 영속성 컨텍스트와 상호작용하기 위해 만들어졌다. 영속성 컨텍스트는 Entity 객체들의 집합이며, Entity 객체들의 LifeCycle을 관리한다.

이러한 영속성 컨텍스트에 Entity Instance들을 추가하고, 삭제하기 위해서 EntityManager라는것이 존재하며, EntityManager는 PK로 Entity를 찾거나, 쿼리를 보낼 수 있다.

**Persistence Context**

JPA에서 굉장히 중요한 용어로써, **Entity를 영구 저장하는 환경**이라는 뜻이다. 논리적인 개념이며, EntityManager를 통해서 영속성 컨텍스트에 접근할 수 있다. 영속성 컨텍스트는 Entity들의 생명주기를 관리하는데 상태는 아래와 같다.

-   비영속 ( new / traisent ) : 영속성 컨텍스트와 전혀 관련 없는 새로운 상태
-   영속 ( managed ) : 영속성 컨텍스트에 관리되는 상태
-   준영속 ( detached ) : 영속상태에서 분리된 상태 (관리되지 않음)
-   삭제 ( removed ) : 삭제된 상태

![JPA%20b10287beceee42bcb0a2503671f0b00e/Untitled%201.png](../assets/jpa_orm_programming_basic_lifecycle.png)

영속성 컨텍스트는 내부적으로 **1차캐시** 라는것을 가지고 있다. 1차 캐시는 쉽게 생각하면 자바의 Map<Id, Object>라고 생각하면 편하다.
Id는 각 Entity의 Primary Key를 의미하고, Object는 각 Entity이다.

EntityManager를 통해서 영속성 컨텍스트에 접근할 수 있다. 만약 EntityManager를 통해서 Entity를 찾고자 한다면, 일단 1차 캐시에서 Entity가 존재하는지 확인 뒤, 존재하지 않을때만 DB에 접근하게 된다. 그렇기 때문에 **EntityManager에서 얻은 동일한 PrimaryKey를 가진 Entity는 동일한 객체라는것을 보장할 수 있다.**

![JPA%20b10287beceee42bcb0a2503671f0b00e/Untitled%202.png](../assets/jpa_orm_programming_basic_entitymanager.png)

영속성 컨텍스트는 쓰기 SQL을 DB에 바로 보내는것이 아니라, 내부적으로 저장한뒤 flush() 메서드가 호출되면 DB에 일괄적으로 전송한다.

![JPA%20b10287beceee42bcb0a2503671f0b00e/Untitled%203.png](../assets/jpa_orm_programming_basic_lazy_sql_write.png)

**Insert 쿼리를 쓰기 지연 저장소에 저장한다.**

![JPA%20b10287beceee42bcb0a2503671f0b00e/Untitled%204.png](../assets/jpa_orm_programming_basic_lazy_sql_write_2.png)

**Commit() 메서드는 내부적으로 flush()메서드를 호출한다. 이후 버퍼링 되있던 INSERT쿼리가 전부 DB에 보내진다.**

영속성 컨텍스트는 영속상태의 Entity가 수정되면 이를 자동으로 감지하여, UPDATE 쿼리를 발생시킨다. 이를 **Dirty Checking**이라고 한다.

이러한 방법이 가능한 이유는 영속성 컨텍스트는 1차캐시에 Entity가 처음 저장될때 SNAPSHOT을 저장하기 때문이다.

이후 flush() 메서드가 호출되면
SNAPSHOT과, 현재의 Entity의 값들을 비교해서 차이가 존재하면, UPDATE 쿼리가 발생하게 되는것이다.

![JPA%20b10287beceee42bcb0a2503671f0b00e/Untitled%205.png](../assets/jpa_orm_programming_basic_dirty_checking.png)

**Flush란?**

영속성 컨텍스트의 변경내용을 데이터베이스에 반영하는것이다. 이때 변경사항이란 Persist를 통해 새로 관리되는 Entity, Dirty Checking을 통해 변경된 Entity, 삭제된 Entity등을 의미한다.

flush()메서드는 DEFAULT로, Commit()메서드나 JPQL 쿼리를 실행하면 자동 호출되지만 EntityManager에서 flush()메서드를 직접호출할 수 도 있다.
EntityManager는 언제 Flush를 시도할것인지(FlushModeType.AUTO, FlushModeType.COMMIT) FlushModeType을 지원한다.

보통 자바에서 flush() 메서드는 Buffering된 무언가를 처리하고 비우는 메서드이다. 영속성 컨텍스트에서 flush()는 지금까지 저장된 변경사항들을 처리하고 비우는 역할이다. 이때 **1차캐시를 지우는것은 아니라는점을** 확실히 해야한다.

**준영속상태 Detached**

준영속상태는 영속상태에서 전이될 수 있는 상태이다. 영속 상태의 엔티티가 영속성 컨텍스트에서 분리된 상태 (Detached)를 의미하며 이말은 즉슨, 영속성 컨텍스트가 해당 Entity를 관리하지 않는다는 의미이다.

이 말은 1차캐시에 해당 Entity가 없다는말과 동일하며, 동일성 비교, Dirty Checking등의 영속성 컨텍스트가 지원하는 기능을 사용하지 못한다는 뜻이다.

준영속상태는 detach(Object o) 메서드를 호출함으로써 사용할 수 있다. 단 모든 1차 캐시를 비우고 싶다면 clear()메서드를 이용하라.

# 엔티티 매핑

**객체와 테이블 Mapping**

JPA를 사용하기 위해서는 객체를 테이블과 매핑할 필요가 있다. **@Entity** 어노테이션을 이용하면 해당 클래스는 JPA가 관리하는 클래스란 뜻이며, 이를 Entity라고 한다.
만약 JPA를 사용하고 싶다면 @Entity 어노테이션은 필수다. 반드시 사용하도록 하자.

JPA Spec상 Entity클래스는 기본생성자를 지원해야한다. 내부적으로 Reflection기술을 사용하는데 이때 기본생성자가 필요하기때문이다.

**@Table** 어노테이션은 클래스를 어떤 테이블에 Mapping할 것인지에 대한 값을 할당할 수 있다. 기본적으로 선언되어있지 않다면 클래스 이름과 똑같은 테이블에 할당하게 되지만,
테이블 이름과 클래스 이름이 불일치할 경우 해당 어노테이션을 사용해서 매핑할 테이블 이름을 설정할 수 있다.

또한 데이터베이스의 **유니크 제약조건 또는 인덱스**를 설정할 수 있다. 유니크 제약조건 같은 경우 **@Column** 에서도 지원하긴 하지만, 여기서는 단일 컬럼에 대한 제약조건만 설정할 수 있으며, 생성되는 제약조건의 이름도 알아보기 힘들기 때문에 가능하다면 **@Table** 어노테이션에서 Unique 제약조건을 설정해주는것이 좋다.

**데이터베이스 Schema 자동생성**

JPA는 테이블에 대한 명세를 전부 가질 수 있기 때문에 자신이 DDL을 생성해서 데이터베이스에 테이블을 자동 생성할 수 있다. 하지만 이런 자동 생성방식은 운영환경, 스테이징 환경, 테스트 환경에서 굉장히 위험하게 작용할 수 있다. 그렇기 때문에 이런 자동생성 기능은 최대한 개발 초기환경에서만 사용하도록 하자.

자동생성기능은 **hibernate.hbm2ddl.auto**를 통해 설정한다.

-   create : 기존 테이블 삭제 후 다시 생성 (DROP + CREATE)
-   create-drop : create와 같으나 App이 끝나는 시점에 테이블 DROP
-   update : Table의 변경분만 반영
-   validate : 엔티티와 테이블이 정상 매핑되었는지 확인. 정상적으로 매핑되어있지 않으면 에러발생!
-   none : 사용하지 않음

**DDL 생성 기능**

예를들면 **@Column(nullable = false, length = 10)** 이라는 어노테이션이 존재하면 NULL 제약조건은 FALSE이고, 길이는 10이하인 DDL을 생성한다. 하지만 생각해볼 수 있는점은 이 기능이 JPA의 실행로직에는 영향을 주지 않는다는것이다. 만약 테이블의 실제 제약조건이 명시한 제약조건과 다른 조건이라면 별 문제가 발생하지 않는다. 즉 DDL을 AUTO-CREATE을 하는 경우가 아니라면 DDL의 제약조건을 명시할 필요는 없지만, 그래도 개발자의 가독성을 위해서 명시적으로 선언해주는것이 좋은 방법일 것이다.

-   @Column : 컬럼 매핑

    name, insertable, updateable, nullable, unique, columnDefinition, length, precision, scale 존재

-   @Temporal : 날짜 타입 매핑 (LocalDate, LocalDateTime을 이용한다면 사용할 필요가 없다.)
-   @Enumerated : Enum타입 매핑 (EnumType.STRING을 항상 이용하자)
-   @Lob : BLOB, CLOB을 매핑한다. Binary 데이터의 경우 BLOB, 문자열 데이터의경우 CLOB으로 자동으로 매핑된다.
-   @Transient : 데이터베이스에 매핑하지 않을 컬럼을 명시할 수 있다.

**기본 키 매핑**

기본키는 데이터베이스에서 굉장히 중요하다. 기본 키를 이용해서 데이터를 고유적으로 식별할 수 있으며, 사실 PK가 존재하지 않는 데이터는 생각하기 힘들다.

기본키를 매핑하는 방법은 개발자가 직접 기본키를 매핑하거나, 데이터베이스에 위임하는 방식이 있다.

-   직접할당 : @Id 어노테이션만 이용한다. 이 어노테이션은 해당 필드가 PK라는 것을 명시한다. 모든 Entity는 @Id라는 어노테이션이 존재해야한다.
-   IDENTITY : @GeneratedValue(strategy=GenerationType.IDENTITY)

    기본 키 생성을 데이터베이스에 위임한다. 보통 Database에서 AUTO_INCREMENT를 생각하면 편리하다. IDENTITY방식을 이용하면 몇가지 주의할점이 존재한다.
    JPA는 내부적으로 1차캐시를 관리하고, 1차 캐시는 PK와 Entity로 이뤄져있다. 또한 JPA는 내부적으로 쓰기 SQL을 지연한다. 하지만 이때 IDENTITY방식으로 Entity를 저장하면,
    PK가 존재하지 않으므로 1차캐시에 데이터를 관리할 수 없다. 그렇기 때문에 IDENTITY방식의 PK 매핑 방식은 SQL쓰기지연을 하지 않는다. Insert Query가 발생하면 바로 데이터베이스에
    데이터를 삽입하게 된다.

-   SEQUENCE : @GeneratedValue(strategy=GenerationType.SEQUENCE)

    데이터베이스 시퀀스는 유일한 값을 순서대로 생성하는 특별한 데이터베이스의 오브젝트를 의미한다. 보통 Oracle DB에서 많이 사용한다. 데이터베이스 시퀀스는 데이터베이스에게 PK값을 요구하면 PK값을 응답받는 식으로 동작한다.
    Insert Query가 발생할때마다 데이터베이스 시퀀스에게 PK를 요청하는것은 사실 네트워크 자원을 낭비하는것과 다름이없다. 사실 이럴바에는 Insert 쿼리를 바로 날리는것이 낫다. 그렇기 때문에 데이터베이스 시퀀스는 PK값을 요청받으면 한번에 큰 폭의 PK값을 응답하게 된다. 그후 해당 메모리에서 이 PK값을 전부 사용할때까지는 데이터베이스 시퀀스에게 PK값을 요구하지 않게된다.

-   Table : @GeneratedValue(strategy = GenerationType.TABLE)

    테이블 전략은 키 생성 전용 테이블을 하나 만들어서 데이터베이스 시퀀스를 흉내내는 전략이다. 이 전략의 장점은 모든 데이터베이스가 이 전략을 사용할 수 있다는 점이지만, 단점으로는 SEQUENCE에 비해 성능이 떨어진다. 나머지는 SEQUENCE 전략과 거의 동일하다.

기본키는 사실 서비스가 운영되면서 유일해야하며 변하지 않아야한다. 이러한 조건을 만족하기 위해서는 **비지니스 로직과 상관없는 키**를 할당하는게 가장 쉽고 편하다. 권장되는 기본키는 Long형으로 선언하고 Key 생성전략을 사용하는것이다.

# 연관관계 매핑 기초

객체지향 연관관계와 데이터베이스 연관관계는 서로 패러다임이 다르다. 객체지향은 Reference를 통해 다른 객체와 연관되지만, 데이터베이스의 테이블은 다른 테이블의 PK를 FK로 설정함으로써, 연관된다. 이런 상황에서 객체를 테이블에 맞춰서 데이터 중심으로 모델링한다면 협력관계를 만들 수 없다.

**단방향 연관관계**

객체끼리 참조 관계가 있다면 회원 → 팀 또는 팀 → 회원 둘 중 한쪽만 참조하는것을 단방향 관계라고 하며, 회원 → 팀, 팀 → 회원 양쪽 모두 서로 참조하는것을 양방향 관계라고 한다.
방향은 객체 세계에서만 존재한다. 데이터베이스 세계에서는 FK를 이용하기 때문에 항상 양방향 관계이다.

여러명의 회원과 회원들이 속해있는 한가지의 팀이 있다면, 이는 다대일 관계이다. 객체는 일반적으로 아래와 같은 코드로 관계에 속해있는 객체를 가져올 수 있다.

```java
member.getTeam();
```

Member에서 Team을 조회할수 있지만 Team에서는 Member를 조회하지 못하는 것. 그것이 바로 단방향 관계이다.

하지만 데이터베이스 세계에서는 단방향 관계란 존재하지 않는다. 단지 FK를 알고만 있다면 어느쪽에서든 JOIN 연산을 통해 관계에 대한 데이터를 얻을 수 있다.

```sql
SELECT * FROM MEMBER M JOIN TEAM T ON M.TEAM_ID = T.TEAM_ID;
SELECT * FROM TEAM T JOIN MEMBER M ON T.TEAM_ID = M.TEAM_ID;
```

사실 객체에서 참조를 통한 연관관계는 항상 단방향이다. 객체간에 연관관계를 양방향으로 만든다는것은 단지 반대쪽에서도 단방향 관계를 만든다는 뜻으로, 서로 다른 단방향 관계가 2개가 있는것이다.

JPA는 위와같은 패러다임의 불일치를 해결하기 위해서 연관관계를 매핑할 수 있는 어노테이션들을 제공한다.

-   **@JoinColumn :** 조인 컬럼은 외래키를 매핑할때 사용된다. name 속성에 매핑할 외래키 이름을 지정한다.
-   **@OneToOne :** 1 : 1 관계를 매핑할때 이용된다.
-   **@ManyToOne :** N : 1 관계를 매핑할때 이용된다.
-   **@OneToMany :** 1 : N 관계를 매핑할때 이용된다.
-   **@ManyToMany :** N : N 관계를 매핑할때 이용된다.

Entity간의 연관관계를 잘 매핑했다면 Entity끼리 객체 그래프를 탐색하는것을 통해 데이터를 얻어올 수 있다.

```java
Member findMember = entityManager.find(Member.class, 1L); //PK가 1인 Member 데이터를 가져온다.
Team findTeam = findMember.getTeam(); //참조를 이용해서 연관관계의 데이터를 조회한다.
```

**양방향 연관관계**

객체에서의 양방향 연관관계는 서로 반대 방향의 단방향 관계가 2개 있는것이다. 데이터베이스 테이블의 경우 FK값을 통해서 조인연산을 수행할 수 있지만 객체는 참조를 통해서 가져온다.
양방향 연관관계는 단방향 연관관계와는 다르게 **연관관계의 주인 (Owner)**을 설정해줘야 한다.
일반적으로 데이터베이스는 N : 1 관계에서 N에 해당하는 테이블이 1에 해당하는 테이블의 PK를 FK로 가지게 된다. 객체는 서로간에 누가 이 FK 값을 관리할것인지 결정해야한다. 그래서 연관관계의 주인만이 외래 키를 관리 (등록, 수정) 하며 주인이 아닌 객체는 읽기만 가능하다.

연관관계의 주인을 설정하는 방법은 연관관계의 매핑을 도와주는 어노테이션에 **mappedBy** 라는 속성을 이용하는것이다. 연관관계 주인은 이 속성을 사용하지 않고, 주인이 아닌 객체는 mappedBy 속성으로 주인을 지정한다.

**누구를 주인으로?**

외래키가 있는곳을 주인으로 정하는것이 중요하다.

**양방향 매핑시 연관관계의 주인에 값을 입력해야 한다.**

만약 연관관계의 주인이 아닌곳에 값을 입력한다면 데이터베이스에 반영되지 않을것이다. 사실 객체지향을 따진다면 두 곳 모두 값을 입력하는게 옳다. 이는 연관관계 편의 메소드를 이용해서 구현하자. 예를들면 아래와 같다.

```java
public void changeTeam(Team team) {
	this.team = team;
	team.getMembers().add(this);
}
```

# 다양한 연관관계 매핑

Entity들간 연관관계를 매핑할 때는 다음과 같은 3가지를 고려해야한다.

1. **다중성**
    - 다대일 : @ManyToOne
    - 일대다 : @OneToMany
    - 일대일 : @OneToOne
    - 다대다 : @ManyToMany
2. **단방향, 양방향**
    - 테이블은 외래키만 있다면 양방향 관계가 성립된다. 테이블은 사실 방향이라는 개념이 없다.
    - 객체는 단뱡향 관계만 존재한다. 참조를 통해 접근하기 때문에 참조 필드쪽으로 관계가 성립된다. 객체에서 양방향 관계란 단방향 관계가 2개 존재하는것이다.
3. **연관관계의 주인**
    - 객체에서 양방향 관계를 성립하고 싶다면, Owner를 지정해야한다. 어떤 객체가 외래키를 수정할 책임이 있는지에 대해 설정해야한다. 하지만 왠만하면 외래키를 가지고 있는 객체가
      외래키를 수정하는것이 옳다. 주인이 아닌 객체는 외래키에 영향을 주지 못한다. 단순 조회만 가능하다.

### 다대일 단방향

가장 많이 사용하는 연관관계이다. @ManyToOne 어노테이션을 지정하고 @JoinColumn을 이용하여 어떤 값을 이용하여 조인할것인지 설정한다.

### 다대일 양방향

양방향 관계에서는 연관관계의 주인, Owner를 설정해야한다. 일반적으로 외래키가 있는 쪽이 연관관계의 주인으로 두는것이 옳다. 객체에서는 서로 객체간 접근할 수 있게 참조필드를 선언하고 사용한다.

### 일대다 단방향

일대다 단방향은 사실 좀 이상한 구조이다. 테이블의 외래키는 1: N 관계에서 N쪽에 생길 수 밖에 없다. 하지만 일대다 단방향을 설정하면 외래키가 N쪽에 존재함에도 불구하고 1쪽에서 외래키를 수정하게된다.

![JPA%20b10287beceee42bcb0a2503671f0b00e/Untitled%206.png](../assets/jpa-orm_programming_basic_onetomany.png)

즉 위와 같은 그림에서 Team에 Member를 추가하게 되면 Member Table의 TEAM_ID가 변경되는것이다. 딱 봐도 무언가 이상하다. 분명히 Team을 수정했는데 바뀌는것이 Member이기 때문에 개발자에게 혼동을 줄 수 있다. 이런 연관관계는 가능한 사용하지 말자.

### 일대다 양방향

이런 매핑을 사실 공식 Spec으로는 존재하지 않지만 편법을 이용해서 구현할 수 있다. 연관관계의 주인이 아닌 쪽에서 @JoinColumn(insertable=false, updatable=false) 를 이용해서 읽기 전용 필드를 만들어버리는것이다. 위와같이 일대다 단방향보다 더 이상한 연관관계이다. 가능한 사용하지 말자.

### 일대일 관계

일대일 관계는 외래키가 존재할 곳을 선택할 수 있다. 주 테이블 또는 대상 테이블 어디에나 외래키가 존재할 수 있으며, 다대일 관계와의 차이점은 외래키가 UNIQUE라는 점이다. 이 뜻은 이 외래키가 하나만 존재한다는뜻으로 1:1 관계임을 뜻한다.

![JPA%20b10287beceee42bcb0a2503671f0b00e/Untitled%207.png](../assets/jpa-orm_programming_basic_onetoone.png)

위의 구조는 MEMBER 라는 주 테이블이 외래키를 관리하는 연관관계이다. 강의에서는 이런 구조를 사용한 이유가 몇가지 존재한다.

1. MEMBER에서 LOCKER에 접근하기 쉬움

2. Lazy전략을 사용가능함. (만약 LOCKER가 MEMBER의 외래키를 가지고있다면 주로 작업하는 MEMBER에서 Lazy전략을 사용할 수 없다. (값이 있는지 없는지 확인 불가능))

    처음에 개인적인 생각으로는 LOCKER가 MEMBER_ID를 외래키로 관리하는것이 옳다고 생각했었다. 그 이유는 외래키가 NULL이 될 수 있다는 점과 이후에 요구사항이 늘어나면 MEMBER테이블이 자연스럽게 커지게 될것이며, MEMBER는 단지 순수한 MEMBER의 정보만 가지고 있어야된다고 생각했다. 하지만 강의를 듣고나서 뭔가 1 : 1 관계에서는 LazyLoading과 관련된 이야기를 듣고 정말 Trade-Off가 될 수 있는 지점이라는것을 알게 되었다.

1:1 관계는 어디에서 외래키를 관리할것인지 신중하게 생각해서 사용하자.

### 다대다 관계

RDB는 다대다 관계를 표현할 수 없다. 일반적으로 중간 테이블을 두고, 일대다, 다대일 관계로 풀어내야한다. 하지만 객체는 다대다 관계가 가능하다.
분명히 JPA가 제공하는 다대다 관계는 편리해보이지만, Entity사이에 존재하는 중간 테이블에는 여러가지 데이터가 추가될 수 있다. 이러한 이유로 인해 다대다 관계는 사용되지 않는다.

![JPA%20b10287beceee42bcb0a2503671f0b00e/Untitled%208.png](../assets/jpa-orm_programming_basic_manytomany.png)

# 고급 매핑

## 상속관계 매핑

RDB는 객체지향처럼 상속관계는 존재하지 않지만, 슈퍼타입과 서브타입이 존재해서 서브타입이 슈퍼타입의 PK값을 FK로 가지는 모델링 기법이 객체 상속과 유사하다.

JPA는 RDB와 객체지향 패러다임의 차이를 극복하기 위해 상속관계 매핑 기능을 여러가지 전략(@Inheritance)으로 제공한다.

-   **@Inheritance(strategy = InheritanceType.JOINED)**

    조인전략으로써, 상위 타입과 하위 타입이 각각 테이블로 분리되어 상위타입의 PK를 하위 타입이 FK로 가지는 형태이다. 정규화된 테이블을 작성할 수 있고, 외래키 제약조건 설정, 저장공간을 효율적으로 저장할 수 있다.

    단 조회시 항상 Join연산을 필요로 하므로, 성능에 문제가 생길수 있고, INSERT 쿼리시에 상위 테이블, 하위 테이블에 각각의 INSERT 쿼리가 발생한다.

-   **@Inheritance(strategy = InheritanceType.SINGLE_TABLE)**

    상위 Entity와 이를 상속받는 하위 Entity들의 모든 Mapping 필드들을 한개의 테이블에서 관리하는 형태이다. 그렇기 때문에 조인연산이 필요하지 않다는것이 장점이다.
    하지만 필요없는 필드가 많아서 저장공간의 효율성이 떨어지고, 하위 Entity의 모든 Column들은 Nullable이여야한다.

    또한 이 레코드가 어느 Entity에 속하는지 알기위해 DTYPE을 항상 사용해야한다.

-   **@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)**

    하위 Entity마다 상위 Entity의 모든 Column을 복사하여, 독립적인 한개의 TABLE을 구성하는 방식이다. 그렇기 때문에 해당 전략에서는 DTYPE이 아예 사용되지 않는다. NOT NULL제약조건도 이용할 수 있으며, 타입을 확실하게 구분할 수 있지만, 상위 Entity를 이용하여 조회하고 싶다면 성능이 매우 느린 UNION SQL을 이용해야한다. 이 방식은 잘 추천되지 않는다.

상속관계간 상위타입의 테이블에서는 이 레코드가 누구의 레코드인지 판단하기 위해서 DTYPE이라는것을 이용한다. DTYPE은 현재 레코드가 무슨 타입인지 알려주는 역할을 한다.
JPA에서는 DTYPE을 명시적으로 선언할 수 있는 방법을 제공한다. **@DiscriminatorColumn, @DiscriminatorValue 와** 같은 어노테이션을 이용한다면 명시적으로 DTYPE을 선언할 수 있다.

## 공통 정보 매핑

### @MappedSuperclass

만약 상속관계가 아니라, 공통으로 매핑을 하고싶은 정보가 있다면 JPA에서는 해당 기능을 제공하기 위해 **@MappedSuperclass** 라는 어노테이션을 지원한다. 이 어노테이션을 이용하면 부모 클래스를 상속받는 자식 클래스에 매핑 정보만 제공한다. 그렇기 때문에 상위 클래스를 이용해 Persistence Context에서 조회 및 검색이 불가능하다. 코드를 아래와 같이 작성해서 사용할 수 있다.

```java
@MappedSuperclass
public abstract class BaseEntity {
	private String createdBy;
	private LocalDateTime createdAt;
	private String lastModifiedBy;
	private LocalDateTime lastModifiedAt;
}
```

이후 이 매핑 정보를 원하는 Entity는 BaseEntity를 상속받아서 이용하면 된다. 보통 이러한 클래스는 직접 생성해서 사용할 일이 없기 때문에 추상 클래스를 이용한다.

## **@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS) vs @MappedSuperclass**

이 질문은 굉장히 개인적인 의문점으로, 사실 위의 두개의 코드를 통해 생성되는 테이블은 동일하다.

그러면 어떤상황에서 상속관계매핑을 이용하고 어떤 상황에서 공통정보매핑을 이용해야 할까? 그것은 바로 상위 타입이 Entity인가? 에 대한 근본적인 의문으로부터 시작된다.
상위타입이 Entity라는것은 PK를가지며, 특정 객체로부터 이 객체가 사용될 수 있다는 의미이다. 즉 비지니스 로직에 포함되는 객체이다. 이러한 Entity는 상속관계 매핑을 이용하는것이 옳다.

공통관계매핑에서도 PK의 생성전략이 동일하다면 공통 매핑이 가능하다. 하지만 이 객체는 비지니스 로직에 포함되지 않는다. 단지 매핑할 데이터 컬럼만 가지고 있을뿐이다.

위의 두가지를 잘 파악해서 상속관계 매핑을 이용할것인지 공통정보 매핑을 이용할것인지 결정해야한다. 결정 기준은 상위 타입의 객체가 Entity가 될 수 있는가?에 대한 의문점에 대한 답이다.

# 프록시와 연관관계 정리

### 프록시

IT쪽에서 보통 프록시라는 용어는 **대리자**라는 의미로 많이 사용되는것 같다. JPA에서도 프록시가 이용되는데, 보통 지연로딩을 하기 위해서 많이 이용된다.

JPA 프록시는 Entity클래스를 상속받아서 만들어지기 때문에 겉으로 보기에는 Entity와 똑같이 보이나 내용물은 텅 비어있는 형태이다. 하지만 이후에 프록시 객체를 사용하면, 실제 객체에게 메서드를 위임함으로써 동작하게 된다. 이렇게 프록시 객체를 사용했을때의 장점은 조회 쿼리를 최대한 늦출 수 있다는점이다. 동작과정은 아래와 같다.

![JPA%20b10287beceee42bcb0a2503671f0b00e/Untitled%209.png](../assets/jpa_orm_programming_basic_proxy.png)

프록시는 처음 사용될때 영속성 컨텍스트에게 초기화를 요청한다. 이후 영속성 컨텍스트는 실제 Entity를 DB로부터 데이터를 가져와 생성하고 프록시 객체에게 넘기게된다.

### 즉시로딩과 지연로딩

즉시로딩과 지연로딩은 연관관계 매핑에서 FetchType의 값을 통해 설정할 수 있다.

-   즉시로딩 (FetchType.EAGER)

    처음 Entity를 가져올때 관련된 Entity들을 JOIN 연산을 통해서 가져온다.

    즉시로딩은 JPQL에서 문제가 발생할 수 있다. 실제 개발자가 보낸 쿼리에 즉시로딩 할 연관된 Entity들을 쌓기 위해 또 다른 쿼리가 발생한다. 이때 각 쿼리가 N개 발생할 수 있는데 이를 N+1 문제라고 한다.
    @ManyToOne, @OneToOne은 Default가 즉시로딩이다. **가능하면, 지연로딩을 사용하자!
    만약 데이터를 JOIN으로 한번에 가져오고 싶다면 이후에 JPQL 또는 Entity Graph 기능을 이용하자.**

-   지연로딩 (FetchType.LAZY) :

    처음 Entity를 가져올때 관련된 Entity들은 Proxy객체로 대체하고, 이후에 사용될때 조회 쿼리를 날리게 된다.
    @OneToMany, @ManyToMany는 Default가 지연로딩이다.

### 영속성 전이 : CASCADE

특정 Entity가 영속성 컨텍스트에 의해 영향을 받을때 연관된 다른 Entity도 같이 영향을 받는다.

-   **CasCadeType.PERSIST** : Entity가 영속화될때 연관된 Entity들도 전부 영속화된다. 다만 바로 발생하는것이 아니라, flush() 메서드가 호출할때 발생한다.
-   **CasCadeType.REMOVE :** Entity가 삭제될때 연관된 Entity들도 전부 삭제된다. 다만 바로 발생하는것이 아니라, flush() 메서드가 호출할때 발생한다.
-   **CasCadeType.DETACH :** Entity가 준영속화될때 연관된 Entity들도 전부 준영속화된다.
-   **CasCadeType.ALL :** 위의 모든것을 전부 적용한다.

### 고아 객체

JPA는 부모 엔티티와 연관관계가 끊어진 자식 엔티티를 자동으로 삭제하는 기능을 가진다. 이를 Orphan (고아) 객체 제거라고 한다. 고아 객체 제거 기능을 이용하려면
orphanRemoval = true 설정해야한다. 이 기능을 사용했을때 부모 Entity에서 자식 Entity를 컬렉션에서 제거하면, DELETE 쿼리가 발생한다.

**고아객체는 반드시 참조하는곳이 하나일때 사용해야한다. 즉 약개체 또는 특정 엔티티가 대상 엔티티를 개인 소유할때 이용해야한다.**

고아 객체 기능을 이용하면 CasCadeType.REMOVE 처럼 동작할 수 있다. 부모 객체가 사라지면 자식 객체의 참조지점이 사라지기 때문이다. 하지만 CasCadeType.REMOVE 와 고아 객체는 확연히 다르다.

### 영속성 전이 + 고아 객체, 생명주기

**CascadeType.ALL + orphanRemoval = true**

이 뜻은 부모 Entity가 자식 Entity의 생명주기를 관리한다는 뜻이다. 보통 Entity의 생명주기는 Persistence Context 즉, 영속성 컨텍스트가 EntityManager의 API를 통해서 관리한다. 하지만 위와 같이 선언하면 부모 Entity에서 컬렉션에 자식 Entity를 추가하고 삭제하는것만으로도 기능이 작동한다. **이는 부모 Entity가 자식 Entity의 생명주기를 관리할 수 있다는 뜻이다.**

# 값 타입

자바에서는 크게 값 타입, 레퍼런스 타입 2가지가 존재한다. 값 타입은 일반적으로 변수가 가르키는 곳이 그대로 값을 가지고 있는 형태이며, 레퍼런스 타입은 변수가 가르키는 곳이 실제 값을 가지고 있는 레퍼런스를 가르키는 형태이다.

**JPA에서도 타입이 크게 2종류가 존재한다.**

- **엔티티 타입**

  @Entity로 정의하는 객체, 데이터가 변해도 식별자(PK)로 지속해서 추적

- **값 타입**

  int, Integer, String처럼 단순히 값으로 사용하는 자바 기본 타입 또는 객체. 식별자가 없고 값만 있으므로 변경시 추적 불가. 즉 Id값이 존재하지 않기때문에 데이터 베이스에서 추적하기 힘들다.

  자바 기본 타입 (int, double), Wrapper 클래스 (Integer, Long), String, Embedded Type, ElementCollection이 존재한다.

JPA에서 객체는 값 타입이 될 수 있다. 이를 Embedded Type이라고 하며, 주로 기본 키 타입을 모아서 만들기 때문에 복합 값 타입이라 불리기도 한다. 보통 아래와 같이 사용된다.

```java
@Embeddable
public class Address {
	private String city;
	private String address;
	private String zipcode;
	public Adderss(){} //기본생성자 필수
}
```

JPA에서 이러한 값타입을 사용할때 발생하는 장점은 많이 있다.

1. 응집성이 좋아진다.  (비슷한 속성값들을 묶음으로써, 책임을 행할 수 있는 하나의 객체로 만들 수 있다.)
2. 재사용성이 좋아진다. (여러곳에서 중복되서 사용되는 비슷한 모양들의 값은 중복될 수 있으므로 이러한 중복을 줄일 수 있다.)
3. Embedded Type은 Entity가 생명주기를 관리한다. 그렇기 때문에 편하게 값을 관리할 수 있다.

### AttributesOverrides (속성 재정의)

만약 Entity에서 같은 값 타입을 사용하여, Column 명이 중복된다면, **@AttributeOverrides** 라는 어노테이션을 이용하여 컬럼 명 속성을 재정의 할 수 있다. 



## 값 타입과 불변 객체

자바와 JPA가 명시하는 값 타입은 약간의 차이가 존재한다. 자바에서 객체는 값타입이 될 수 없지만, JPA에서는 값타입이 될 수 있다는것이다. 그렇기 때문에 JPA에서는 값 타입을 Entity 객체끼리 공유할 수 있다. 

![JPA%20b10287beceee42bcb0a2503671f0b00e/Untitled%2010.png](../assets/jpa_orm_programming_basic_value_type_1.png)

위의 그림에서 보다시피 주소라는 값 타입을 회원 Entity끼리 공유하다가 한 회원이 값타입을 변경하면 side effect(부작용) 이 생기게 된다. 이런문제는 값타입 객체를 불변객체로 만듦으로써 해결할 수 있다. 

사실 불변객체는 생성자를 통해서 값을 초기화하고 내부 속성들을 final로 설정해야, 완벽한 불변객체임을 보장할 수 있지만, JPA의 특성상 기본 생성자로 객체를 생성하고 이후 필드들에 값을 주입하기 때문에 내부 필드를 final로 설정하는것은 불가능하다.

그렇기 때문에 JPA 값타입을 이용하기 위해서는 Setter 메서드만 접근제어자를 Private 수준으로 설정하여, 이후에 값을 변동할 수 없도록하고, 값을 바꾸고 싶다면 Static Factory Method 또는 생성자를 통해 새로운 객체를 만들어 값을 바꾸도록 해야한다.

![JPA%20b10287beceee42bcb0a2503671f0b00e/Untitled%2011.png](../assets/jpa_orm_programming_basic_value_type_2.png)

**값 타입은 동등성 (Equals)비교시 모든 필드가 같다면, 똑같은 인스턴스라고 판단하고, True를 반환해야한다. 그러므로 값 타입으로 사용하고 싶은 객체는 equals(), hashcode() 메서드를 적절하게 Overriding 해야한다.**

## 값 타입 컬렉션

Entity가 하나이상의 값 타입을 저장할때 사용할 수 있다. @ElementCollection, @CollectionTable을 사용하여, 정의할 수 있다.  사실 RDB에서는 Entity에서 Collection을 저장할 수 있는 방식은 존재하지 않는다. 다만 별도의 테이블을 생성하여 그곳에 값들을 저장한다. 

값 타입 컬렉션 또한 모든 값들을 불러오기 위해서는 JOIN연산을 수행해야한다. 또한 기본적으로 LazyLoading을 수행하며, 생명주기를 Entity가 관리한다. 위에서 봤던 영속성 전이 **CascadeType.ALL 과 orphanRemoval = true**를 설정한것과 동일하다.

**값 타입 컬렉션 주의점**

값 타입 컬렉션을 사용하다가 변경 사항이 발생하면, 주인 Entity와 관련된 모든 레코드를 삭제하고, 현재 컬렉션에 있는 데이터를 기반으로 레코드를 다시 저장한다. **(굉장히 비효율적)** 
또한 값타입 컬렉션을 매핑하는 테이블은 Id가 존재하지 않기 때문에, 모든 컬럼을 묶어서 하나의 PK를 구성해야한다. **(PK가 존재하지 않는 테이블은 있어서는 안된다..)**

**그래서 값 타입 컬렉션은 되도록 사용하지 않는것이 좋다.**

값타입 컬렉션 대신 @OneToMany 일대다 연관관계 매핑을 이용하는것을 고려하자. 일대다 관계 Entity가 하나의 Wrapper클래스처럼 동작하는형식으로 클래스를 만드는것이다. 그리고 영속성 전이 + 고아 객체 제거를 사용하면 값 타입 컬렉션 처럼 사용할 수 있다.

```java
@Entity
public class AddressEntity{
	@Id @GeneratedValue
	private Long id;
	@Embedded
	private Address address;
}
```

**Entity와 값 타입을 혼동해서 실제로 Entity여야 할 데이터를 값 타입으로 만들어서는 안된다. 식별자가 필요하고, 지속해서 값을 추적, 변경해야한다면 값 타입이 아닌 Entity로 생성해야한다.**



# JPQL (객체지향 쿼리 언어)

JPA를 사용하게 되면, RDB에 쿼리를 날리는게 아니라, Entity를 중심으로 개발하기 때문에 Entity에 쿼리를 날리게 된다. Entity에 쿼리를 날린다는 의미는 모든 DB 데이터를 객체로 만들어서 검색하는것이 아니다.(이런 기능은 불가능하다) 오히려 **Entity 쿼리를 적절한 SQL 쿼리로 변경을 해준다는 의미이다.**

JPA는 SQL을 추상화한 JPQL이라는 객체 지향 쿼리 언어를 표준으로 제공한다. 이는 특정 데이터베이스에 언어가 의존하지 않는다는 의미이며, JPQL은 SQL 문법과 굉장히 유사하고, SELECT, FROM, WHERE, HAVING, JOIN, GROUP BY 등의 명령어를 지원한다.

JPA는 JPQL뿐만 아니라 다양한 검색 방법도 표준으로 지원한다.

- JPQL : 객체 지향 쿼리 언어
- Criteria Query : JPQL 쿼리의 Builder 역할. 동적인 쿼리를 생성 가능 ( 잘 사용하지 않음 )
- Native Query : JPQL대신에 데이터베이스에 직접 SQL을 사용하는 방식 ( SQL이 데이터베이스마다 약간씩 다르기 때문에 데이터베이스에 의존적임 )

표준은 아니지만 많이 사용되는 검색방법도 존재한다.

- QueryDSL : Criteria 쿼리처럼 JPQL을 편하게 작성해주도록 도와주는 Builder 클래스 모음이다. 비표준이지만 많이 사용된다.
- JDBC, MyBatis 같은 SQL 매퍼 프레임워크 사용 : 필요하면 JDBC를 직접 사용하여 쿼리를 날린다.



**JPQL을 사용하는 간단한 예제**

```java
List<Member> members = entityManager.createQuery("select m from Member as m where m.username = 'kim'", Member.class).getResultList();
```

JPQL에서는 쿼리를 Entity를 대상으로 날리며, Where절에서 조건검사도 객체의 필드 명을 통해 쿼리를 수행한다.



### Criteria Query

Criteria는 JPQL을 생성하는 빌더 클래스이다. Criteria의 장점은 Java 프로그래밍 코드로 JPQL을 작성할 수 있다는 점으로, SQL 오류를 컴파일 시점에서 잡아낼 수 있다. 또한 Builder의 역할이기 때문에 동적 쿼리를 생성해낼 수 있다.

Criteria Query를 사용하는 간단한 예제

```java
CriteriaBuilder cb = em.getCriteriaBuilder();
CriteriaQuery<Member> query = cb.createQuery(Member.class);

Root<Member> m = query.from(Member.class);

CreateiaQuery<Member> cq = query.select(m).where(cb.equal(m.get("username"), "kim"));
List<Member> members = em.createQuery(cq).getResultList();
```

Criteria는 JPA 표준이고 많은 장점을 가지지만, 너무 복잡하고 장황하다. 그래서 많이 사용되지 않는다.



### Query DSL

QueryDSL도 Criteria처럼 JPQL 빌더 역할을 한다. 하지만 Criteria Query보다 훨씬 사용하기 쉽고 편하다. 또한 동적 쿼리를 매우 쉽게 작성할 수 있고, 컴파일 시점에 문법 오류를 찾을 수 있다.

Query DSL을 사용하는 간단한 예제

```java
JPAQueryFactory query = new JPAQueryFactory(em);
QMember m = QMember.member;
List<Member> members = query.selectFrom(m).where(m.age.gt(18)).orderBy(m.name.desc()).fetch();
```



### Native SQL

JPA가 제공하는 SQL을 직접 사용하는 기능이다. JPQL로써 해결할 수 없는 특정 데이터베이스에 의존하는 기능을 사용할때 주로 사용한다.

```java
List<Member> resultList = em.createNativeQuery("SELECT * FROM MEMBER WHERE NAME = `kim`", Member.class).getResultList();
```



### JDBC 직접 사용, SpringJdbcTemplate 등

JPA를 사용하면서 JDBC Connection을 직접 사용하거나 MyBatis를 사용할 수 는 있다. 하지만 이는 JPA에서 Persistence Context 바깥의 일이기 때문에 이런것을 사용하기 이전에 JPA의 쓰기 지연 SQL을 flush()메서드를 호출하여 데이터베이스를 갱신해줄 필요가 있다. 

만약 갱신하지 않는다면, JPA에서 쓰기가 지연된 데이터들은 JDBC 커넥션을 통해서 얻어올 수 없을것이다.



## JPQL 문법

JPQL은 SQL을 추상화하여 만들어졌기 때문에 SQL과 굉장히 비슷하게 작성할 수 있다. JPQL 문법의 특징은 아래와 같다.

- SQL 테이블이름이 아니라 Entity이름을 이용한다.
- Entity 변수를 선언하듯이, 별칭을 필수로 이용해야한다 **ex) Member as m**



### TypedQuery 와 Query

JPQL이 실행되고 나서 데이터를 담을 쿼리 객체를 정해야한다. 

- TypedQuery : 반환타입이 명확할때 사용한다.
- Query : 반환타입이 명확하지 않을때 사용한다.

```java
TypedQuery<Member> query = em.createQuery("select m from Member as m", Member.class); // Member 객체로 타입이 명확하다.
Query query = em.createQuery("select m.username, m.age from Member as m") // Member클래스 중에서 username과 age만 얻어오는데, 이를 담을 객체가 명확하지 않다.
```



### 결과 조회 API

- **getResultList()**

  결과가 하나 이상일때 List를 반환하고, 결과가 없으면 Empty List를 반환한다.

- **getSingleList()**

  결과가 정확히 하나일때 단일 객체를 반환한다. 이 API는 안좋은 점이 결과가 없거나 여러개 이상일때 Exception이 발생한다. 
  결과가 없으면 NoResultException, 둘 이상이면 NonUniqueResultException 



### Parameter Binding

JDBC는 위치 기준 파라미터 바인딩만 지원하지만, JPA에서는 이름 기준 파라미터 바인딩도 지원한다. 이름 기준 파라미터는 앞에 이름 앞에 ' : ' 를 붙혀서 사용한다.
되도록이면 위치 기준 파라미터보다 이름 기준 파라미터 바인딩을 이용하자.

```java
em.createQuery("select m from Member as m where m.username = :username").setParameter("username", "kim"); //이름 기준
em.createQuery("select m from Member as m where m.username = ?1").setParameter(1, "kim"); //위치 기준
```



### 프로젝션

프로젝션은 SELECT 절에 조회할 대상을 지정하는것이다. 조회할 대상은 여러개가 존재할 수 있다.

- Entity를 조회 ⇒ Entity Type 프로젝션
- 복합 값 타입을 조회 ⇒ Embedded Type프로젝션
- 기본 값 타입을 조회 ⇒ Scala Type 프로젝션

```java
em.createQuery("select distinct m.team from Member m", Team.class); // Member를 통해 Team을 조회할 수 있다. 또한 DISTINCT 연산을 통해 중복제거가 가능하다. 단 이같은 방식은 좋지않다.
em.createQuery("select distinct t from Member inner join m.team t", Team.class); //위의 방식보다 아래의 방식이 JOIN연산이 수행된다는 사실을 JPQL에서 바로 알 수 있다.
```

만약 프로젝션으로 여러값을 조회한다면, 단순값을 DTO로 변경하여 조회할 수 있다. 이때 프로젝션의 값 순서와 일치하는 생성자가 있어야 한다.

```java
em.createQuery("select new example.package.UserDTO(m.username, m.age) from Member m", UserDto.class);
```



### Paging API

데이터베이스마다 Paging을 하는 방법이 다르다. JPA는 Paging을 두개의 API로 추상화 했다.

- setFirstResult(int startPosition) : 어디서부터 시작할 것인가?
- setMaxResult(int maxResult) : 몇개를 가져올 것인가?

페이징을 할때는 항상 특정한 기준으로 정렬을 하고 가져와야 한다.

```java
em.createQuery("select m from Member m order by m.name desc").setFirstResult(10).setMaxResult(10).getResultList(); //Offset 10으로부터 Limit 10을 가져오는 쿼리
```



### 집합 함수

집합은 일반적으로 통계정보를 구할 때 이용한다. 아래는 순서대로 회원수, 나이 합, 나이 평균, 나이 최대, 나이 최소를 의미한다. DISTINCT도 이용할 수 있다.

```java
em.createQuery("select count(m), sum(m.age), avg(m.age), max(m.age), min(m.age) from Member m");
em.createQeruy("select count (distinct m.age) from Member m"); // distinct를 통한 중복 제거도 가능
```



### GROUP BY, HAVING

GROUP BY는 통계 데이터를 구할때 특정 그룹끼리 묶어준다. 아래는 팀 이름을 기준으로 그룹별로 묶어서 통계 데이터를 구한다. HAVING은 GROUP BY와 함께 사용하는데, GROUP BY로 그룹화한 통계 데이터를 기준으로 필터링한다.

```java
em.createQuery("select t.name count(m), sum(m.age), avg(m.age), max(m.age), min(m.age) from Member m left join m.team t group by t.name");
em.createQuery("select t.name count(m), sum(m.age), avg(m.age), max(m.age), min(m.age) from Member m left join m.team t group by t.name having avg(m.age) >= 10");
```

이런 쿼리를 보통 통계 쿼리라고 하는데, 이런 쿼리를 잘 활용하면 애플리케이션에서 작성되는 코드를 획기적으로 줄일 수 있다. 하지만 일반적으로 통계 쿼리는 전체 데이터를 기준으로 처리하기 때문에 실시간용으로 사용하기에는 적합하지 않다. 결과가 아주 많다면, 통계 결과를 저장하는 테이블을 별도로 만들고 사용자가 적은 새벽에 통계 쿼리를 이용하여 그 결과를 보관하자.



### JPQL 조인

JPQL도 JOIN 연산을 지원한다. 다만 문법만 약간 다르다.

**순서대로 내부 조인, 외부 조인, 세타 조인 이다.**

```java
em.createQuery("select m From Member m inner join m.team t") // Inner 조인을 하게되면 Team이 존재하는 Member만 가져온다.
em.createQuery("select m From Member m left outer join m.team t") //Outer 조인을 하게되면 Team이 존재하지 않아도 모든 Member를 가져온다.
em.createQuery("select m From Member m, Team t where m.team_id= t.team_id") 
```

**JOIN ON절**

JPA2.1부터 ON절을 지원한다. ON을 이용하면 JOIN할 대상을 줄일 수 있다. 

```java
em.createQuery("select m, t from Member m left outer join m.team t on t.name ='A'"); // Team 이름이 A인 Record만 Join연산을 수행한다.
```

**ON절과 WHERE절의 차이**

```java
em.createQuery("select m, t from Member m left outer join m.team t on t.name ='A'"); // ON
em.createQuery("select m, t from Member m left outer join m.team t where t.name ='A'");  // WHERE
```

ON절을 이용하면 Team Table에서 조인할 Record를 필터링 하고나서 연산을 수행한다. 그렇기 때문에 외부 조인의 특성상 팀 이름이 "B"인 멤버도 결과에 포함된다. 단 이때 Team 은 NULL이다.

WHERE절을 이용하면 모든 Record를 조인연산하고나서 이후에 조건에 맞는 레코드를 선택한다. 그렇기 때문에 팀 이름이 "A"인 멤버들만 결과에 포함된다.



### 서브 쿼리

JPQL도 SQL처럼 서브 쿼리를 지원한다. 다만 서브쿼리는 WHERE과 HAVING에서만 이용할 수 있다. (Hibernate에서는 SELECT 에서까지 사용할 수 있다. 다만 표준은 아니다)

```java
em.createQuery("select m from Member m where m.age > (select avg(m2.age) from Member m2)"); //평균보다 나이가 많은 멤버를 선택
```

서브쿼리는 다음 함수들과 같이 이용할 수 있다.

- EXISTS (sub query) ⇒ 서브 쿼리에 결과가 존재하면 참이다.
- [ALL | ANY | SOME] (sub query) ⇒ 비교 연산자와 같이 이용되며, 조건을 만족하면 참이다.
- IN (sub query) ⇒ 서브 쿼리의 결과중 하나라도 같은것이 있으면 참이다.



### 조건식

조건에 따라 Column 값을 반환하는 연산이다. 순서대로 CASE식, COALESCE식, NULLIF식 이다.

```java
em.createQuery("select case m.age <= 10 then '학생요금' when m.age >= 60 then '경로요금' else '일반요금' end from Member m");
em.createQuery("select coalesce(m.username, '이름 없는 회원') from Member m");
em.createQuery("select nullif(m.username '관리자') from Member m");
```



### 경로 표현식

객체는 .(점)을 찍어서 객체 그래프를 탐색할 수 있다. 마찬가지로 JPQL에서도 .(점)을 찍어서 객체의 값 또는 연관 필드를 탐색할 수 있다. 경로 표현식은 필드에 따라 몇가지 용어가 존재한다.

- 상태필드(state field) : 단순히 값을 저장하기 위한 필드이다. ex) m.username
- 연관필드(association field) : 연관관계를 위한 필드이다. ex) @ManyToOne 과 같은 연관관계 매핑으로 연결한 Entity
- 컬렉션 값 연관필드 : @OneToMany와 같이 대상이 컬렉션인 경우이다.

**경로 표현식 특징**

상태 필드와 같은경우 경로표현식에서 경로 탐색의 끝에 해당하며, 더이상 탐색할 수 있는 공간이 존재하지 않는다.

단일 값 연관 필드의 경우 탐색 시 묵시적인 내부 조인(inner join)이 발생한다. 또한 이후에 탐색이 가능하다.

컬렉션 값 연관 경로의 경우 묵시적 내부 조인(inner join)이 발생하지만, 탐색은 불가능하다. 하지만 FROM 절에서 명시적 조인을 통해 별칭을 얻어 탐색이 가능하다.

```java
em.createQuery("select m.username from Member m"); // 상태 필드 경로 탐색
em.createQuery("select m.team from Member m"); // 연관 필드 경로 탐색
em.createQuery("select t.members from Team t"); // 컬렉션 연괄 필드 경로 탐색
```

위의 경우 전부 묵시적인 조인이다. 즉 JPQL에서는 드러나지 않지만 실제 SQL을 보면 JOIN 연산이 수행되는것을 알 수 있다.
하지만 JOIN연산은 애플리케이션에서 정말 중요하다. SQL 튜닝에도 사용되며 개발자가 JPQL만 보더라도 JOIN연산이 발생하는지 쉽게 알 수 있어야 한다. **그렇기 때문에 묵시적인 조인을 사용하는것이 아닌 명시적인 조인을 사용하자.**

```java
em.createQuery("select m from Member m join m.team t");
em.createQuery("select m from Team t join t.members m");
```



### Fetch Join (패치 조인)

패치 조인은 SQL 조인은 아니다. JPQL에서 성능 최적화를 위해 제공하는 기능으로, 연관된 엔티티 또는 컬렉션을 **한 번의 SQL로 모두 조회하는 기능이다.**

```java
em.createQuery("select m from Member m join fetch m.team");
==
[SQL]
SELECT M.*, T.* FROM MEMBER M INNER JOIN TEAM T ON M.TEAM_ID = T.ID
```

위와 같이 Fetch 조인을 이용할 수 있다. 위의 쿼리는 Member뿐만 아니라 Team의 데이터도 함께 조회하게 된다.

그렇기 때문에 위에서 Member를 통해 Team을 접근할때 추가적인 SQL (지연로딩) 이 발생하지 않고 Team의 데이터를 접근할 수 있게 된다.

**Collection Fetch Join**

1:N 관계에서 테이블을 일대다 관계로 조인하면, 결과로 나온 데이터의 레코드는 1개가 아니라 여러개로 복사된다. 그렇기 때문에 실제로 Collection 을 Fetch Join하면 같은 객체가 여러개 나올 수 밖에 없는 구조이다.

```java
List<Team> teams = em.createQuery("select t from Team t inner join t.members where t.name = 'A'");
for(Team team : teams){
	System.out.println(team.getName());
}
```

위와 같은 코드에서 팀에 소속된 멤버가 여러명이라면, 멤버가 속해있는 만큼 Team 객체가 생기게 된다. 이는 관계형 데이터베이스이기 때문에 발생하는 어쩔 수 없는 중복이다.

일반적으로 데이터베이스 SQL에서는 중복 제거를 위해 DISTINCT라는 명령을 제공하는데, 위의 쿼리에서 데이터는 사실 중복된것이 아니기 때문에 DISTINCT가 의미가 없다. 

하지만 Fetch Join에서는 DISTINCT를 이용하면 위의 Entity 중복을 제거해준다. 이는 애플리케이션에서 중복을 제거해주는 기능으로 RDB와는 관련이 없다.

```java
List<Team> teams = em.createQuery("select distinct t from Team t inner join t.members where t.name = 'A'");
for(Team team : teams){
	System.out.println(team.getName());
}
```

위의 코드를 실행하면 중복된 Team Entity는 존재하지 않는다. 

**Fetch Join과 Join의 차이가 무엇인가?**

JPQL에서 Join을 한다고해서 연관된 Entity를 불러오는것이 아니다. 다만 Join을 이용해서 Where절에서 특정한 조건을 부여하여 Record를 선택할 수는 있겠다. 예를들면 아래와 같다.

```java
em.createQuery("select m from Member m join m.team t where t.name ='A'"); // Team 이름이 A인 Member를 가져옴.
```

위의 코드는 Team 이름이 A인 Member를 가져오는것이지, Team에 대한 데이터는 가져오지 않는다. 

JPQL은 결과를 반환할때 연관관계를 고려하지 않는다. 단지 SELECT 절에 지정한 Entity만 조회한다. 여기서는 Member만 조회하는것이고, Team은 조회하지 않는다.

Fetch Join을 사용하면 Member를 포함해서 Team의 데이터도 같이 가져오게 된다(즉시 로딩). **이는 객체 그래프를 SQL 한번으로 조회하는 개념이라고 생각할 수 있다**

**Fetch Join의 특징과 한계**

- Fetch 조인 대상에는 별칭을 줄 수 없다.

  hibernate는 가능하지만 가급적이면 사용하지 않는게 좋다. Fetch Join은 일반적으로 데이터를 한번에 조회하기 위해 사용하는것이다.

- 둘 이상의 Collection 에 대해 Fetch Join 할 수 없다.

- 컬렉션을 Fetch Join하면 페이징 API를 사용할 수 없다.

  이유는 일대다 관계에서는 데이터 레코드가 많아지기 때문이다. 이럴 경우 하이버 네이트는 모든 레코드를 메모리 위로 올리고 (위험) 그 위에서 페이징한다. 

- 연관된 Entity를 SQL 한번으로 조회하기 때문에 성능 최적화를 할 수 있다.

- Entity에 직접 적용하는 Loading 전략보다 우선한다. 즉 LAZY 로딩을 해도 Fetch Join을 통해 연관 Entity를 가져올 수 있다.

- 최적화가 필요한곳에 사용하라. ex) N + 1 쿼리 문제

모든 것을 Fetch Join으로 해결할 수 없다. Fetch Join은 특정한 객체 그래프를 유지할때 사용하면 효과적이며, 여러 테이블을 Join해서 다른 결과를 내야 한다면 필요한 데이터들만 조회해서 DTO로 반환하는것이 효과적이다.



### 다형성 쿼리

JPA는 Entity간 상속이 가능하다. 그렇기 때문에 상위 타입을 기반으로 한 쿼리가 가능하다.

```java
em.createQuery("select i from Item i where type(i) IN (BOOK, MOVIE)");
em.createQuery("select i from Item i where treat(i as Book).author = 'kim'");
```

TYPE 문법은 조회 대상을 특정 자식으로 한정한다.

TREAT는 자바의 타입 캐스팅과 유사하다. 상속 구조에서 부모 타입을 특정 자식 타입으로 다룰 때 사용한다. 위에서 



### Entity 직접 사용

JPQL에서 Entity를 직접 사용하면 기본 키 값을 이용한다. 또한 Where절에서 "=" 비교시 Entity를 이용할 수 있다.



### Named 쿼리

Named 쿼리는 정적 쿼리로 미리 정의해서 이름을 부여해두고 사용하는 JPQL이다. Annotation 또는 XML에 정의해서 이용가능하다.
애플리케이션 로딩 시점에 쿼리를 파싱하기 때문에 컴파일 시점에 문법 오류를 검증할 수 있다. 

```java
@Entity
@NamedQuery(name = "Member.findByUsername", query = "select m from Member m where m.username = :username")
public class Member

em.createNamedQuery("Member.findByUsername", Member.class).setParameter("username", "회원1").getResultList():
```



### Bulk 연산

JPA는 Update시 Dirty Checking 기능이 있긴 하지만, 많은 데이터를 변경 감지 기능으로 실행하는것은 너무 많은 SQL을 실행하게 된다. 이는 변경된 데이터가 100개라면 100개의 SQL을 날리게 되는것과 동일하다.

쿼리 한번으로 여러개의 테이블 Record를 변경한다.

```java
em.createQuery("update Product p set p.price = p.price * 1.1 where p.stockAmount < :stockAmount")
	.setParameter("stockAmount", 10)
	.executeUpdate();
```

벌크 연산은 데이터베이스 영속성 컨텍스트를 무시하고 데이터베이스에 직접 쿼리한다. 물론 벌크 연산을 수행하기전, 영속성 컨텍스트는 Flush되어 쓰기연산이 수행되지만, 벌크 연산이 수행되고 나서의 1차캐시가 최신데이터라는것을 보장할 수 없다.

**즉 벌크 연산 수행 후 이후에 영속성 컨텍스트가 다시 1차캐시에 데이터를 쌓도록 영속성 컨텍스트를 초기화 하자.**
