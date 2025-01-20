---
layout: default
title: DDD - 인터넷 정보
parent: memo
---
# DDD - 인터넷 정보
---

- DDD의 강점
	- 도메인 용어를 코드에 반영하지 않으면 개발자는 코드의 의미를 해석해야 한다.
	- 코드의 가독성을 높여 분석과 이해하는 시간을 절약시킨다.
	- 용어가 저의 되 때마다 용어 사전에 이를 기록하고 명확하게 정의 함으로써 추후 다른 사람들도 공통된 언어를 사용할 수 있도록 한다.

- 용어
	- 도메인 : 일반적인 요구사항, 소프트웨어로 해결하고자 하는 문제 영역
	  솔직히 이해가 잘 안가니 나중에 예제를 찾아보자
	- 도메인 모델 : 특정 도메인을 개념적으로 표현한 것
	  이것도 이해가 안되니 다시 찾아보자
	- 엔티티 : 테이블 모델, 고유 식별자를 가짐
	- 애그리게이트 : 연관된 엔티티와 값객체의 묶음, 일관성과 트랜잭션, 분산의 단위
		- JPA에서 사용하는 엔티티라고 보면 될것같다.
		- 다만 order - orderItem 과같은 경우 모두 연관관계 매핑이 되어있어야함.
	- 바운디드 컨텍스트 : 특정한 도메인 모델이 적용되는 제한된 영역
		- 모델은 특정한 컨텍스트(문맥)하에서 완전한 의미를 갖는데, 이렇게 구분되는 경계를 갖는 컨텍스트를 바운디드 컨텍스트라고 한다.
		- 좋은 바운디드 컨텍스트란
		  하나의 바운디드 컨택스트에는 하나의 팀만 할당되어야 한다.
		  하나의 팀은 여러 개의 바운디드 컨택스트를 할당받을 수 있다.
		  둘 이상의 팀이 하나의 바운디드 컨택스트를 같이 관리하는건 안티패턴
		  각각의 바운디드 컨택스트는 각각의 개발환경을 가질 수 있다.
		- 이게 MSA인듯?
	- 컨텍스트 맵 : 바운디드 컨텍스트 간의 관계
		- 상호 교류하는 시스템 목록을 제공하고, 팀 내 원활한 의사소통을 위해 표현되는 방식이다.

- DDD를 하기위해 지켜야할것
	- 참고 : [링크](https://velog.io/@gowjr207/도메인-주도-설계DDD를-설명해보다)
- DDD의 단점
- 사용하지 말아야하는 경우
- 사용시 유의 점





---

https://jaehoney.tistory.com/248

-   상품 애그리거트 : 구매하는 상품의 가격이 필요하다. 또한 상품에 따라 배송비가 추가되기도 한다.
-   주문 애그리거트 : 상품별로 구매 개수가 필요하다.
-   할인 쿠폰 애그리거트 : 쿠폰별로 지정한 할인 금액이나 비율에 따라 주문 총 금액을 할인한다.
-   회원 애그리거트 : 회원 등급에 따라 추가 할인이 가능하다.

위 4개의 애그리거트중 결제 금액 계산을 수행하는 애그리거트는?
(나는 주문 이라고 생각)

```java
public class Order {  
    ...  
    private Orderer orderer;  
    private List<OrderLine> orderLines;  
    private List<Coupon> usedCoupons;  
  
    private Money calculatePayAmounts() {  
        Money totalAmounts = calculateTotalAmounts();  
        // 쿠폰 별로 할인 금액 계산  
        Money discount = counts.stream()  
                .map(coupon -> calculateDiscount(coupon)  
                        .reduce(Money(0), (v1, v2) -> v1.add(v2));  
        // 회원에 따른 추가 할인 계산  
        Money membershipDiscount = calculateDiscount(orderer.getMember().getGrade());  
        // 실제 결제 금액 계산  
        return totalAmount.minus(discount).minus(membershipDiscount);  
    }  
  
    private Money calculateDiscount(Coupon coupon) {  
        // orderLines의 각 상품에 대해 쿠폰을 적용해서 할인 금액을 계산하는 로직  
        // 쿠폰의 적용 조건 등을 확인하는 코드        // 정책에 따라 복잡한 if-else와 계산 코드        ...  
    }  
  
    private Money caculateDiscount(MemberGrade grade) {  
        // 등급에 따라 할인 금액 계산  
        ...  
    }  
}
```

이렇게 **한 애그리거트에 넣기 애매한 도메인 기능을 억지로 특정 애그리거트에 구현하면 안된다.** 이는 코드가 길어지고 외부에 대한 의존이 높아지게 되며 코드를 복잡하게 만들고 도메인 개념이 가려지는 요인이 된다.

이러한 문제를 해결할 수 있는 좋은 방법은 도메인 기능을 별도 서비스로 구현하는 것이다.

### 도메인 서비스

도메인 서비스는 **도메인 영역에 위치**한 도메인 로직을 표현할 때 사용한다. 주로 다음 상황에서 도메인 서비스를 사용한다.

```java
public class DiscountCalculationService {  
    public Money calculateDiscountAmounts (List<OrderLine> orderLines, List<Coupon> coupons, MemberGrade grade) {  
		  
        Money couponDiscount = coupons.stream()  
                .map(coupon -> calculateDiscount(coupon))  
                .reduce(Money(0), (v1, v2) -> v1.add(v2));  
  
        Money membershipDiscount = calculateDiscount(grade);  
  
        return couponDiscount.add(menbershipDiscount);  
    }  
    private Money calculateDiscount(Coupon coupon) { ... }  
    private Money calculateDiscount(MemberGrade grade) { ... }  
}
```
---

도메인은 **실세계에서 사건이 발생하는 집합**이라고 생각하면 쉽습니다
옷 쇼핑몰을 한번 예로 들어볼까요?
옷 쇼핑몰에서는 손님들이 주문하는 도메인(Order Domain)이 있을 수 있고,
점주입장에서 옷들을 관리하는 도메인(Manage Domain)이 있을 수 있고,
쇼핑몰 입장에서 월세, 관리비 등 건물에 대한 관리를 담당하는 도메인(Building Domain)이 있을 수 있습니다.
주문 도메인에서의 옷은 손님들에게 팔기 위한 객체 정보(name, price .. etc)들을 담고 있지만,
옷을 관리하는 도메인에서는 옷은 점주 입장에서 관리하기 위한 객체 정보(madeTime, size, madeCountry ... etc)들을 위주로 담고 있습니다.
다시 말해서 Context에 따라 객체(Object)의 역할은 바뀐다는 것이에요!
흔히 이것을 **Bounded Context**라고 부르기도 한답니다
그래서 이를 외부로(public)으로 노출시키지 않고, package-private으로 내부에서만 알 수 있게 하는 것이에요

이러한 관점을 더 나아가서 직접 서비스에 적용시킨 것이 바로 **MircoService(마이크로서비스)**입니다

Domain Layer : Entity, VO를 활용하여 도메인 로직이 진행되는 계층
Entity는 고유 식별자(primary Key)를 바탕으로 객체의 정체성을 부여하고,
VO(Value Object)는 상태(Attribute)를 바탕으로 객체의 정체성을 부여해요!

Aggregate
정말 정말 정말 쉽게 말하면, 각각의 도메인 영역을 대표하는 객체가 바로 Aggregate입니다.
그렇게 되면, 각각의 **도메인에 Repository로 묶어야 하는 객체(Entity)**가 명확해지기 때문이죠
Top-Down 방식으로 계층을 타고타고 내려갈 때 어떤 Entity를 만들기 위해 도메인이 이루어져 있는지
명확하게 들어오기도 한답니다!

DDD의 핵심 목표
- 모듈간 의존성 최소화
- 응집성 최대화

내부 구현을 프라이빗하게 가져간다. (높은 응집력, 캡슐화)
내부 구현의 변경이 인터페이스에 의존하는 객체에 영향이 없도록 한다.(낮은 결합도)

도메인 지식을 구현 코드에 반영한다.

[aggregate](https://happycloud-lee.tistory.com/94#:~:text=Aggregate%EB%8A%94%20%EB%85%BC%EB%A6%AC%EC%A0%81%EC%9D%B8%20DB,%EC%9D%98%20%EC%86%8D%EC%84%B1%EC%9D%B4%20VO%EC%9E%85%EB%8B%88%EB%8B%A4.) : entity 또는 vo에 접근할때 aggregate를 통해 하라는데용?
- **Primary key**-참조는 Primary key 사용: 다른 Aggregate를 참조할때 객체자체를 레퍼런스하지 말고 객체의 primary key로 참조해라. Order는 Consumer 객체 자체를 참조하지 않고, consumerId값으로 Consumer aggregate를 참조함.
- **One to One** -한 개의 트랜잭션은 한 개의 Aggregate만 Writing

---

https://yoojin99.github.io/cs/DDD/

## 값 객체

값의 성질과 값 객체 구현
값의 성질을 이해하는 것은 값 객체를 이해하는 데 중요하다. 값의 성질은 아래와 같다.

- 변하지 않는다. (불변)
- 주고받을 수 있다.
- 등가성을 비교할 수 있다.

따라서 값을 바꾸는 것은 권장되지 않는다. 이를 염두해서 위에서 언급했던 `FullName` 클래스를 다시 보자. 그리고 만약에 이 클래스에 이름을 수정하는 동작이 있다고 해보자.

```
var person = FullName(firstName: "철수", lastName: "김")
person.changeLastName(lastName: "이")
dump(person)
```

위와 같이 코드를 작성하면 부자연스러운 부분이 생긴다. **위의 코드는 값이 수정되기 때문에 좋은 코드가 아니다. FullName은 값 객체이고, 값이기도 하다. 따라서 1이 0으로 변하면 안되는 것처럼 변해선 안된다. 따라서 `changeLastName`처럼 값을 수정하는 기능이 클래스에 정의되면 안된다.**

> 불변하는 값의 장점
> 
> > 상태가 변화하지 않게 하면 프로그램을 단순하게 만들 수 있다. 병렬 처리가 일어나는 프로그램에서는 상태가 변화할 수 있는 객체를 다루는 방법이 관건이기 때문에, 객체의 상태가 변하지 않는다면 병렬/병행 처리를 쉽게 구현할 수 있다. **물론 객체의 일부만 바꾸고 싶다고 해도 객체를 아예 새로 생성해야 한다는 단점이 있다.** 하지만 가변 객체 -> 불변 객체로 바꾸는 작업보다 불변 객체 -> 가변 객체 로 바꾸는 작업이 노력이 적게 들기 때문에 일단 불변 객체를 적용하는 것이 낫다.

### 2. 교환 가능하다.

값 객체를 수정하는 방법은 새로운 객체를 만들어 대입을 통해 교환하는 방식으로 구현한다.

```kotlin
var person = FullName(firstName: "철수", lastName: "김")
person = FullName(firstName: "철수", lastName: "이")
```


### 3. 등가성 비교 가능

프로그래밍의 원시타입끼리는 같은 값끼리 비교할 수 있다. 이처럼 값 객체도 속성을 통해 비교할 수 있다. **하지만 값 객체에서 값을 꺼내서 비교하는 것은 자연스럽지 못하다.**

```kotlin
// 원시 타입
print(0 == 0)
print("hi" == "hi")

let person1 = FullName(firstName: "철수", lastName: "김")
let person2 = FullName(firstName: "철수", lastName: "김")

// 값 객체의 값을 또 꺼내서 비교한다. 값에서 값을 꺼낸다? 부자연스럽다.
let result = person1.firstName == person2.firstName && person1.lastName == person2.lastName
print(result)
```

값과 마찬가지로 값 객체도 값끼리 비교하는 것이 자연스럽다. 이를 위해서 **값 객체를 비교하는 메서드를 정의한다.**

```kotlin
class FullName: Equatable {
    private(set) var firstName: String
    private(set) var lastName: String
    
    init(firstName: String, lastName: String) {
        self.firstName = firstName
        self.lastName = lastName
    }
    
    func equals(fullName: FullName) -> Bool {
        guard self === fullName
                || self.firstName == fullName.firstName
                && self.lastName == fullName.lastName else {
            return false
        }
        return true
    }
    // Equatable protocol
    static func == (lhs: FullName, rhs: FullName) -> Bool {
        guard lhs === rhs
                || lhs.firstName == rhs.firstName
                && lhs.lastName == rhs.lastName else {
            return false
        }
        return true
    }
}

// 비교는 이렇게 할 것이다.
let person1 = FullName(firstName: "철수", lastName: "김")
let person2 = FullName(firstName: "민수", lastName: "김")
let person3 = person1

print(person1.equals(fullName: person2))
print(person1.equals(fullName: person3))
print(person2.equals(fullName: person3))
print(person1 == person3)
```


### 값 객체가 되기 위한 기준

위에서 언급한 예시에서 FullName안의 속성 또한 값 객체로 만들 수 있다. 이에 대한 기준은 사람마다 다른데, 기준을 아래와 같이 잡을 수 있다.

1.  규칙이 존재하는가
2.  낱개로 다루어야 하는가
