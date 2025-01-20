---
layout: default
title: JPA - Batch insert 성능 저하
parent: memo
---
# JPA - Batch insert 성능 저하
---

벌크성 저장 로직 성능저하 원인에 대해 알아보자.

jpa를 사용하면서 엔티티를 정의할때 아래와 같이 `IDENTITY` 를 많이 사용하고 있었다.

`GenerationType.IDENTITY` : 기본키 생성을 DB에 위임한다
```java
@Id  
@GeneratedValue(strategy = GenerationType.IDENTITY)  
private Integer sn;
```


mysql을 사용하면서 pk를 autoincrement 를 사용하면 자연스럽게 `IDENTITY` 를 사용하게된다.

jpa에서는 saveAll( { any_object } ) 와 같은 벌크성 저장을 할 것 같은 함수를 지원하는데,

이는 `IDENTITY` 를 사용하게되면 지원하지 않게된다.

이는 people.size()가 100,000 의 데이터를 saveAll(people) 함수를 통해 저장할 경우 쿼리가 10만개 날아간다

이유는 새로 할당할 Key 값을 미리 알 수 없는 IDENTITY 방식을 사용할 때 Batch Support를 지원하면 Hibernate가 채택한 flush 방식인 `Transactional Write Behind`와 충돌이 발생하기 때문에, IDENTITY 방식에서는 Batch Insert를 비활성화 한다는 얘기다. 따라서 그냥 일상적으로 가장 널리 사용하는 IDENTITY 방식을 사용하면 Batch Insert는 동작하지 않는다.
[stackoverflow](https://stackoverflow.com/questions/27697810/why-does-hibernate-disable-insert-batching-when-using-an-identity-identifier-gen/27732138#27732138)

---

## 해결방법

### `SEQUENCE` 또는 `TABLE` 전략 사용

`SEQUENCE` 와 `TABLE` 를 사용하는 방법은 잘못 사용할 경우 본인 생각대로 작동하지 않을 위험이 있기때문에 채택하지 않았다.
( `SEQUENCE` 는 mysql에서 지원하지 않음 )


### UUID 사용

uud를 id 값으로 사용하는 방법이 있다.
이는 보안상으로 좋을 수 있다.
채택하지 않은 단점으로는
- 16 byte 를 사용하여 기존의 int형의 컬럼보다 큰 용량을 차지한다
- 외래키로 사용될때 16 byte를 사용한다는 단점은 극대화된다.
- insert를 실행할때 uuid를 만들어야 하기때문에 성능에 저하가 있다.
- 확률은 적지만 uuid가 중복될 수 있는점을 고려해야한다.
- 정렬하는데 사용할 수 없다.


### jdbc 사용
또는 jooq사용

jdbc를 사용하는 방향으로 채택했다.
이유는 성능저하가 느껴지는 대량의 insert 기능은 우리 서비스에 많지 않다.
또한 서비스의 규모가 분산 DB를 사용할정도가 아니기 때문에 auto increment를 사용하는데 제한적이지 않다.

