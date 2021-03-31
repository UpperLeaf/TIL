# JPA 최적화 방법

### JPA 최적화

**JPA는 개발자 대신 SQL문을 생성하여 데이터베이스에 쿼리를 날리므로 내부에서 어떻게 동작하는지 정확히 알고 사용할 필요가 있다.**

Spring Data JPA는 기본적으로 Entity끼리의 연관관계에서 **Lazy Loading**을 한다. Eager 과 같은 즉시로딩도 있지만, 즉시로딩을 사용하는것은 많은 경우에 성능에 도움을 주지 못한다. 즉시로딩이라고 해서 N + 1 쿼리 문제가 해결되는것은 아니며, N + 1 쿼리문제는 JPQL에서 제공하는 Fetch Join을 이용해서 해결해야한다.

JPA는 기본적으로 Entity를 대상으로 쿼리하고, Fetch Join을 사용하지 않으면 쿼리하는 Entity에 대한 데이터만 가져온다. 이 Entity가 여러 Entity와 연관관계를 가진다면, 내부적으로 해당 Entity에 대한 프록시 객체를 가지고 있으며, 프록시 객체에서 필드에 접근할때 Lazy Loading이 동작하면서 쿼리가 발생하게 된다.

이것이 N + 1 쿼리를 일으키는 현상이다. 한개의 쿼리로 N개의 Entity를 가져왔을때 Entity와 연관된 Entity를 가져오기 위해 N개의 쿼리를 추가로 날리게 된다.

JPA는 위의 문제의 해결방법으로 Fetch Join이라는것을 제공한다. Fetch Join을 사용하면 연관된 Entity를 단 1개의 쿼리만으로 가져올 수 있다. 즉 N + 1 쿼리문제를 깔끔하게 한개의 쿼리로 변경한다는 의미다.

```java
//단 한개의 쿼리만으로 모든 Member와 연관된 Team의 데이터도 가져올 수 있다.
entityManager.createQuery("select m from Member m join fetch m.Team t");
```

@ManyToOne, @OneToOne 관계에서 이러한 Fetch Join은 정말로 유용하고 쉽게 사용가능하다.

### @OneToMany 관계에서의 Fetch Join

@ManyToOne, @OneToOne과 @OneToMany의 차이가 무엇인가? 데이터베이스에 쿼리를 날렸을때 OneToMany 관계의 경우 Row가 N개가 된다. 다시 말해 1 : N 관계에서 1 을 통해 쿼리를 날리면, 쿼리의 결과로 N개에 해당하는 Row가 반환된다는 의미다.

사실 너무나 당연한 결과이다. 관계형 데이터베이스는 List를 저장할 수 없으며, 모든 관계를 FK로 관리하기 때문에, N개의 결과가 나와야하는것이다. 사실 중복된 결과처럼 보이지만 사실 전부 다른 Row인것이다.

**하지만 JPA로 넘어오면서 이것은 확실한 중복이 된다.** JPA는 관계형 데이터 베이스와는 다르게 데이터를 Collection으로 가질 수 있다. 즉 데이터베이스를 곧이곧대로 객체로 만들어서 매핑한다는것은 사실상 중복된 객체를 Collection에서 지니는것이라고 할 수 있다.

실제로 JPA를 이용해서 1 : N 쿼리를 날려보면, 중복된 데이터가 담기는것을 확인할 수 있을것이다.

### Distinct를 통한 중복 제거

JPA는 위와 같은 중복을 제거하기 위해서 Distinct 키워드를 제공한다. 데이터베이스를 공부한 사람이라면 Distinct 키워드가 데이터베이스에서 중복을 제거하기 위한 키워드라는것을 알 것이다. 하지만 JPA의 Distinct와 Database의 Distinct는 비슷하면서도 다르다.

아까 썻다시피, Database의 Row는 사실 중복이 아닌 전부 다른 Row이다. 즉 Distinct 키워드를 쓴다고해서 Database의 결과가 달라지는것은 아니다. 하지만 JPA에서 Distinct 키워드를 사용하면 중복되는 객체를 전부 제거하고, 결과를 반환한다. 즉 JPA의 Distinct는 Entity의 중복을 제거해주는것이다.

### @OneToMany Paging

@OneToMany의 경우 Distinct Fetch Join을 한다고 하더라도, 데이터베이스에서 데이터의 중복을 걸러서 JPA에게 주지는 않는다. Entity 중복은 JPA에서 걸러진다는 의미이다.

페이징이란 특정지점부터 얼마나 가져올것인가? 에 대한 쿼리이다. 만약 Limit이 10이라면 10개의 개별적인 Entity를 원한다는 뜻이다. 

그렇다면 1:N 관계에서 Fetch Join Limit 10 쿼리를 데이터베이스에 보내 받은 데이터는 10개의 개별적인 Entity일까? 아니다.
이는 **조금 생각해본다면 알 수 있다.** @OneToMany의 경우 결과 Column이 1개당 N개로 바뀐다. 즉 데이터베이스로부터 받은 10개의 Column이 전부 사실상 한개의 Entity일수도 있는것이다.

**그래서 JPA는 OneToMany를 Paging할때 모든 데이터를 읽고 메모리에서 조인하여, 결과를 페이징한다. 이것은 정말로 위험하다.**

### O**neToMany 관계에서 어떻게 Paging을 할까?**

Fetch Join을 하지 않는다. 그렇다면 Paging을 수행할 수 있다. 그리고 Lazy Loading을 해서 OneToMany 관계에 있는 데이터를 가져온다. 이때 지연로딩으로 인해 N + 1 쿼리문제가 발생한다.

이 N + 1 쿼리문제는 @BatchSize를 통해 최적화 할 수 있다.

지연로딩 성능 최적화를 위해서 Hibernate는 BatchSize를 설정할 수 있게 해준다.

```yaml
hibernate.default_batch_fetch_size = 100 
# 100 ~ 1000 사이를 이용한다. 
# 숫자가 높을 수록 데이터베이스에 큰 부하가 가지만, 쿼리 수를 줄일 수 있다는 장점이 있다.
```

위의 뜻은 컬렉션 또는 Proxy 객체를 조회할때 한꺼번에 설정한 Size만큼 Where IN (?, ?, ?) 쿼리로 조회하는것을 의미한다.

즉 1 + N 쿼리가 아니라 1 + 1 쿼리로 수행할 수 있는것이다. 이것이 OneToMany에서 Paging을 하면서 쿼리를 최적화 할 수 있는 방법이다.

개별적으로 BatchSize를 설정하고 싶다면 @BatchSize를 지정하여 사용한다.