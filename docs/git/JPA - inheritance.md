---
layout: default
title: JPA - inheritance
parent: git
---

프로젝트를 진행하면서 @inheritance를 사용하려고합니다.

사용하려는 부분은 응답과 진단 부분입니다.
```
응답
├─ 단일 응답
├─ 다중 응답
├─ 순위 응답
└─ 텍스트 응답
```
@inheritance를 사용하여 '응답'이라는 공통적인 메서드가 있으면 좋을것 같습니다.

```
역량진단
└─ 대상자, 진단자 상태 스냅샷
   └─ 대상자, 진단자
      └─ 진단
         ├─ 진단 응답
         └─ 진단 분석 결과

수습진단
└─ 대상자, 진단자
   └─ 회차
      └─ 진단
         ├─ 진단 응답
         └─ 진단 분석 결과
```
진단 타입에 따라 가지고있는 정보가 조금씩 다르지만 공통되는 동작이 있어 상속을 받으면 좋을 것 같습니다.

>짧은 결론
>응답부분에만 상속을 적용했습니다.
>책 또는 인터넷 상의 대부분이 간단한 예시로 장점을 부각하고 있습니다. (부모 - 자식)
>하지만 상속 계층이 깊어지는 경우 (회사 - 부서 - 팀 - 직원)
>쿼리를 작성할때 조인이 깊어지고 복잡성이 증가할 수 있기 때문에 진단에서는 사용하지 않았습니다.
>참고 https://browndwarf.tistory.com/m/55

JPA를 사용하면 상속 관계의 테이블은 어떻게 구성해야하는지 알아보겠습니다.

Inheritance - 상속 관계를 매핑하기 위한 다양한 어노테이션입니다.
주요한 상속 전략에는
- 단일 테이블 전략
- 테이블 별 클래스 전략
- 조인 테이블 전략이 있습니다.

**조인 테이블 전략(Join Table Inheritance)**:
    - `@Inheritance(strategy = InheritanceType.JOINED)`: 각 엔티티 클래스에 대해 별도의 테이블을 생성하고, 부모 클래스와 자식 클래스 사이의 관계를 유지하기 위해 조인 테이블을 사용합니다.
![[Pasted image 20240410154646.png]]

**단일 테이블 전략(Single Table Inheritance)**:
    - `@Inheritance(strategy = InheritanceType.SINGLE_TABLE)`: 부모 클래스와 자식 클래스의 모든 필드를 하나의 테이블에 매핑합니다.
    - `@DiscriminatorColumn(name = "discriminator_column_name")`: 단일 테이블에 저장된 각 엔티티를 식별하기 위한 컬럼을 지정합니다.
    - `@DiscriminatorValue("entity_discriminator_value")`: 각 엔티티의 구분 값을 지정합니다.
![[Pasted image 20240410154702.png]]

**테이블 별 클래스 전략(Table Per Class Inheritance)**:
    - `@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)`: 각 엔티티 클래스마다 별도의 테이블을 생성합니다.
![[Pasted image 20240410154724.png]]

### 선택 가이드라인
- 상속 구조가 단순하고 조회 성능이 중요한 경우 **단일 테이블 전략**을 사용합니다.
- 데이터베이스 정규화가 중요하고 테이블 관리가 용이해야 하는 경우 **조인 전략**을 사용합니다.
- 상속 구조가 복잡하고 각 엔티티의 속성이 독립적인 경우 **구현 클래스마다 테이블 전략**을 사용합니다.
- 코드 중복을 줄이고 유지 관리를 용이하게 하려는 경우 **MappedSuperClass 전략**을 사용합니다.

---

## 조인 테이블 전략
Join Table Inheritance

```java
@Entity
@Inheritance(strategy = InheritanceType.JOINED)
@DiscriminatorColumn(name = "DTYPE")
public abstact class Item() {

	@Id
	@GeneratedValue
	@Column(name = "itme_id")
	private Long id;
	private String name;
	private int price;
}

@Entity
@DiscriminatorValue("A")
public class Album extends Item {
	private String artist;
}

@Entity
@DiscriminatorValue("M")
public class Movie extends Item {
	private String director;
	private String actor;
}

@Entity
@DiscriminatorValue("B")
@PrimaryKeyJoinColumn(name = "book_id") // id 재정의
public class Book extends Item {
	private String author;
	private String isbn;
}
```

- 장점
	- 테이블 정규화
	- 외래 키 참조 무결성 제약조건 활용
	- 저장 공간 효율적 사용
- 단점
	- 조회할 때 조인으로 인한 성능 저하
	- 복잡한 조회 쿼리
	- 1번의 Insert 쿼리가 N번으로 늘어날 수 있음


## 단일 테이블 전략
Single Table Inheritance


```java
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "DTYPE")
public abstact class Item() {

	@Id
	@GeneratedValue
	@Column(name = "itme_id")
	private Long id;
	private String name;
	private int price;
}

// joined와 동일하다
@Entity
@DiscriminatorValue("A")
public class Album extends Item {
	private String artist;
}

@Entity
@DiscriminatorValue("M")
public class Movie extends Item {
	private String director;
	private String actor;
}

@Entity
@DiscriminatorValue("B")
public class Book extends Item {
	private String author;
	private String isbn;
}

```

- 장점
	- 조인이 필요없으므로 성능이 빠름
	- 조회 쿼리가 단순함
- 단점
	- 자식 엔티티가 매핑한 컬럼은 모두 null을 허용해야함
	- 단일 테이블에 모든것을 저장하므로 테이블이 커짐, 그러므로 상황에 따라 성능이 느려질 수 있음
- 특징
	- 구분 컬럼을 꼭 사용해야함 (@DiscriminatorValue 필수)
	- DiscriminatorValue이름을 지정해주지않는다면 기본으로 엔티티 이름을 사용함

## 테이블 별 클래스 전략
Table Per Class Inheritance

```java
@Entity
@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)
public abstact class Item() {

	@Id
	@GeneratedValue
	@Column(name = "itme_id")
	private Long id;
	private String name;
	private int price;
}

// joined와 동일하다
@Entity
public class Album extends Item {
	private String artist;
}

@Entity
public class Movie extends Item {
	private String director;
	private String actor;
}

@Entity
public class Book extends Item {
	private String author;
	private String isbn;
}
```

해당 전략은 자식 엔티티마다 테이블을 만든다. 일반적으로 추천하지 않는 전략

- 장점
	- 서브 타입을 구분해서 처리할 때 효과적
	- not null 제약 조건을 사용할 수 있음
- 단점
	- 여러 자식 테이블을 함께 조회할 때 성능이 느림 (UNION 쿼리 불가피)
	- 자식 테이블을 통합해서 쿼리하기 어려움
- 특징
	- 구분 컬럼을 사용하지 않음

---

## MappedSuperclass

활용 예제 [[Entity Listener]]

```java
@Getter  
@MappedSuperclass  
@EntityListeners(AuditingEntityListener.class)  
public abstract class BaseEntity {  
  
    @CreatedDate  
    @Column(updatable = false)  
    private LocalDateTime createDateTime;  
  
    @LastModifiedDate  
    private LocalDateTime updateDateTime;  
  
}
```
BaseEntity는 테이블 매핑이 필요없고, 자식 엔티티에게 공통으로 사용되는 매핑 정보만 제공하면 됨

```java
@Entity
@AttributeOverrides({
	@AttributeOverride(name = "createDateTime", @Column(name = "create_at")),
	@AttributeOverride(name = "updateDateTime", @Column(name = "update_at"))
})
public class Member extends BaseEntity {

//...

}
```
`@MappedSuperclass`로 지정한 클래스는 엔티티가 아니므로 em.find()나 JPQL에서 사용할 수 없음

---
