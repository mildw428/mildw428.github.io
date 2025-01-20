---
layout: default
title: JPA - OneToOne 성능 저하
parent: memo
---
# JPA - OneToOne 성능 저하
---

테이블을 설계할 때 정규화 등의 목적으로 1 대 1 관계로 설계했었다.

OneToOne의 N+1문제를 모른체 JPA를 사용하는 프로젝트였던 만큼 당연하게 OneToOne을 사용했다.

fetch join을 통해서 N+1문제는 없앨 수 있었다.

fetch join을 통해서 가져오는 데이터의 양이 너무 방대해서 또 다른 이슈를 발생시켰다.

join으로 인해 너무 많은 데이터로 인해 처리가 오래 걸리고, 프로젝트 간 통신이 끊겼던 것이다.

```java
@OneToOne(fetch = FetchType.LAZY , optional = false)
@JoinColumn(name = "example_parent_property_sn") private ExampleParentProperty exampleParentProperty;﻿
```

핫픽스 건이라 급하게 이슈를 처리해야 했고, 그 대안으로  엔티티를 똑같이 복사해서 OneToOne인 부분만 주석 처리했다.(양방향 관계-> 단방향 관계)

이렇게 해서 부모 엔티티에서 자식 엔티티(대량의 데이터) 관계를 끊었다.

앞으로 1:1 관계의 테이블은 상황을 고려하여 부모 테이블에 fk를 두는것도 고려하자.

### 학습 내용

jpa를 사용하면서 gradle에

```
implementation 'com.github.gavlyukovskiy:p6spy-spring-boot-starter:1.5.8'
```

를 추가해주면 실제로 호출되는 쿼리를 확인할 수 있다.

N+1 문제는 다음과 같은 상황을 말한다.

![](https://blog.kakaocdn.net/dn/oLyde/btrv2ET8WsA/v9eMRSyxK8SIiClY47jAp1/img.png)

부모객체를 findAll()을 통해 조회(1번)를 한다.

이때 부모객체에 속해있는 자식 객체를 사용하기 위해서 또 한 번의 쿼리를 발생하는 것이 N+1문 제이다.

위의 사진에서는 4개의 부모 객체가 있고, 자식 객체를 사용하기 위해 4번의 쿼리를 더 발생시키고 있다.

### N+1문제 발생 원인

JPA를 그냥 사용할 경우 자식 객체를 따로 호출하는데서 발생한다.

```
JPQL은 Java Persistence Query Language의 약자로, DB 테이블이 아니라 엔티티의 객체를 대상으로 검색하는 객체 지향 쿼리이다.

jpaRepository에 정의한 인터페이스 메서드를 실행하면 JPA는 메서드 이름을 분석해서 JPQL을 생성하여 실행하게 된다.

JPQL은 SQL을 추상화한 객체지향 쿼리 언어로서 특정 SQL에 종속되지 않고 엔티티 객체와 필드 이름을 가지고 쿼리를 한다.

그렇기 때문에 JPQL은 findAll()이란 메소드를 수행하였을 때 해당 엔티티를 조회하는 select * from Owner 쿼리만 실행하게 되는것이다.

JPQL 입장에서는 연관관계 데이터를 무시하고 해당 엔티티 기준으로 쿼리를 조회하기 때문이다.

그렇기 때문에 연관된 엔티티 데이터가 필요한 경우, FetchType으로 지정한 시점에 조회를 별도로 호출하게 된다.



출처 - https://incheol-jung.gitbook.io/docs/q-and-a/spring/n+1
```

```java
@Entity
@Data
@NoArgsConstructor
public class ExampleParent {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long sn;

    private String name;

    @OneToMany(fetch = FetchType.LAZY, cascade = CascadeType.ALL)
    @JoinColumn(name = "example_parent_sn")
    private List<ExampleChild> exampleChildList = new ArrayList<>();

    public ExampleParent(String name) {
        this.name = name;
    }

    public void addExampleChild(ExampleChild exampleChild) {
        this.exampleChildList.add(exampleChild);
        exampleChild.setExampleParent(this);
    }
}
```

부모 객체가 자식 객체를 호출하는 방법으로

@OneToMany, @ManyToOne, @OneToOne은 자식 객체에 대해 정의할 때 옵션 값을 통해 호출 시점을 정할 수 있다.

FetchType.EAGER : 객체를 미리 호출한다.

FetchType.LAZY : 객체를 사용할 때 호출한다.

EAGER옵션을 사용하면 무조건 N+1 이슈가 발생하기 때문에 특별한 경우가 아닌 이상 LAZY옵션을 사용하는 걸 권장한다.

두 옵션다 N+1을 발생하는 건 마찬가지이기 때문에 다른 조치가 필요하다.

### N+1문제 해결

### 1. Join Fetch

다음과 같이 부모 객체를 조회할 때 자식 객체를 조인해서 가져올 수 있다.

```java
@Query("select p from ExampleParent p join fetch p.exampleChildList")
List<ExampleParent> findAllJoinFetch();
```

![](https://blog.kakaocdn.net/dn/cey1f5/btrv9KsNNIu/kFgaGnnzF1VKZHApWRfrAk/img.png)

실행 결과로 N+1문제가 발생하지 않는 것을 확인할 수 있다.

### 2. QueryDsl

두 번째는 queryDsl을 사용하는 방법이 있다.

QueryDsl을 사용하여 fetch join하거나 원하는 필드만 조회하여 문제를 해결 할 수 있다.

다음은 fetch join의 예제 코드이다.

```java
@Override
public List<ExampleParent> findAllLeftJoinExampleChild() {
    return from(exampleParent)
            .leftJoin(exampleParent.exampleChildList, exampleChild)
            .fetchJoin()
            .fetch();
}
```

![](https://blog.kakaocdn.net/dn/cVG3pT/btrv9KGkgiJ/Fu9kEwmgnKWEssK5yhTAt1/img.png)

마찬가지로 N+1 문제가 발생하지 않고 로직을 수행하는 것을 볼 수 있다.

아래는 sn, name만을 필드로 가진 ExampleParentDto를 생성하여 그것만 조회할 수 있도록 하는 코드이다.

```java
@Override
public List<ExampleParentDto> getAllExampleParentDto() {
    return from(exampleParent)
            .select(Projections.fields(ExampleParentDto.class,
                    exampleParent.sn,
                    exampleParent.name
                    ))
            .fetch();
}
```

이렇게 필요한 부분만 조회를 하면 자식객체를 불러올 일이 없기 때문에 N+1문제가 발생하지 않는다.

### 추가

다음은 @OneToOne을 추가한 부모 객체 코드이다.

```java
@Entity
@Data
@NoArgsConstructor
public class ExampleParent {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long sn;

    private String name;


    @OneToOne(fetch = FetchType.LAZY, cascade = CascadeType.ALL)
    @JoinColumn(name = "example_parent_property_sn")
    private ExampleParentProperty exampleParentProperty;


    @OneToMany(fetch = FetchType.LAZY, cascade = CascadeType.ALL)
    @JoinColumn(name = "example_parent_sn")
    private List<ExampleChild> exampleChildList = new ArrayList<>();

    public ExampleParent(String name) {
        this.name = name;
        this.exampleParentProperty = new ExampleParentProperty(10000L);
    }

    public void addExampleChild(ExampleChild exampleChild) {
        this.exampleChildList.add(exampleChild);
        exampleChild.setExampleParent(this);
    }
}
```

그리고 위에서 실행한 메서드를 그대로 실행했을 때 쿼리문을 살펴보자.

![](https://blog.kakaocdn.net/dn/sIzpa/btrwaFLojek/LdXbMUZnY0FWQfLRxJWEz0/img.png)

위에서 다뤘던 N+1의 경우는 사용할 경우에만 호출하는 반면, @OneToOne의 경우는 LAZY를 설정하고 사용하지 않았는데도 호출을 하고 있다. 

(lazy load 동작하지 않는 이유 공부)

> ### 2.1.2.1. Lazy Loading - 객체 그래프 탐색 & Proxy
> 
> 그렇다면 왜 Lazy 글로벌 패치 설정이 동작하지 않았을까?
> 
> JPA에선 객체 그래프 탐색이라는 논리적인 개념이 존재한다. 특정 도메인 객체를 조회할 때 연관 관계를 맺은 각각의 도메인의 글로벌 패치 전략을 판단하다. 이때 Lazy 전략으로 설정된 도메인에 대해선 내부적으로 default 생성자를 통해 Proxy 객체를 생성하고([참고 CGLIB](https://github.com/cglib/cglib/wiki)) 조회할 도메인 객체는 생성된 Proxy 객체를 참조하게 된다.
> 
> 즉 nullable한 도메인에 대해선 Proxy 객체 생성을 보장할 수 없다. 따라서 Lazy 패치 전략이더라도 내부적으로 객체간 참조를 보장하기 위해 쿼리를 발생시켜 연관된 객체에 데이터를 매핑시키는 동작 방식을 취하게 된다.  
>   
> Member와 MemberOption의 OneToOne관계로 봤을때,
> 
> 이를 토대로 앞서 본 테스트 결과를 분석하면 다음과 같다.  
> MemberOption 클래스가 Proxy가 아니다.  
>   
> nullable한 OneToOne 관계라면, Proxy 객체를 감싸지 않고 내부적으로 객체를 반환한다. 단. Collection(*ToMany)은 다른 방식을 취한다.  
>   
> MemberOption 필드에 접근하지 않았음에도 MemberOption를 조회 쿼리가 발생한다.  
>   
> 하이버네이트는 객체 그래프 탐색하는 시점에 Lazy로 판단하여 쿼리 생성 시 Member 쿼리만 생성하게 된다.   
>   
> 하지만 MemberOption은 Proxy 객체가 아니므로 MemberOption 쿼리가 발생하여 참조 받을 수 있도록 쿼리를 발생시키게 된다.  
>   

lazy load가 동작하기 위해서는 proxy보장되어야 하고,

Proxy 생성을 보장하기 위해선 non-null 관계가 형성되어야 한다. JPA의 연관관계 관련된 어노테이션에는 optional이라는 속성이 존재한다. 다음 속성을 통해 Proxy 생성을 보장하면 된다.

```java
@OneToOne(fetch = FetchType.LAZY
        , cascade = CascadeType.ALL
        , optional = false) // false - non-null, true - nullable
@JoinColumn(name = "example_parent_property_sn")
private ExampleParentProperty exampleParentProperty;
```

또는 fetchType을 EAGER로 설정하게 되면 자동으로 join 해서 불러올 수 있다. 하지만, 사용하지 않는 데이터까지 무작정 가져오게 되면 메모리 낭비가 될 수 있다.
