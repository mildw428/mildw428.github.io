---
layout: default
title: Design Pattern - Getter Setter Constructor
parent: memo
---
# Design Pattern - Getter Setter Constructor
---

## setter 없이 개발
[setter 없이 get post 통신](https://jojoldu.tistory.com/407)

[setter없이 개발 1](https://www.inflearn.com/questions/80680/setter-%EC%97%86%EC%9D%B4-%EC%98%81%EC%86%8D%EC%84%B1-%EC%BB%A8%ED%85%8D%EC%8A%A4%ED%8A%B8%EC%97%90%EC%84%9C-update-%ED%95%98%EB%8A%94-%EB%B0%A9%EB%B2%95%EC%9D%B4-%EC%9E%88%EC%9D%84%EA%B9%8C%EC%9A%94)
[setter없이 개발2](https://velog.io/@langoustine/setter%EB%A5%BC-%EC%93%B0%EC%A7%80%EB%A7%90%EB%9D%BC%EA%B3%A0)
setter 없이 어떻게 데이터를 수정할까?
setter의 경우 JPA의 Transaction 안에서 Entity 의 변경사항을 감지하여 Update 쿼리를 생성한다. 즉 setter 메소드는 update 기능을 수행한다.

여러 곳에서 Entity를 생성하여 setter를 통해 update를 수행한다면 복잡한 시스템일 경우 해당 update 쿼리의 출처를 파악하는 비용은 어마어마할 것이다.

그렇다면 어떻게 setter를 배제할까? 아니 setter를 어떤 방식으로 대체하는지 알아보자.

사용한 의도나 의미를 알 수 있는 메서드를 작성하자.
```java
@Getter
@Entity
public class Post {
	
    private Long id;
    private String userId;
    private String title;
    private String cont;
    
    public void updatePost(Long id, String title, String cont) {
        this.id = id;
        this.title = title;
        this.cont = cont;
    }
}
```

```java
post.updatePost(1L, "수정할 제목입니다.", "수정할 내용입니다.");
```

---

## getter 없이 개발

```java
public class Game {

	public void run(Car car) {
		int number = car.getNumber();
        
		if (number >= 4) {
        	car.update(number+1);
		}
	}
    
	public void run2(Car car) {
    	car.move(4);
	}

}
```

위와 같이 Car클래스와 Car클래스로부터 객체를 만들어 게임을 실행하는 Game클래스가 있다.

Game의 run메소드는 getter를 이용하여 로직을 처리하고 있고,

run2메소드는 객체에 메시지를 보내서 로직을 처리하고 있다.

두 로직 모두 car객체의 number가 4이상이면 number를 1만큼 증가시키는 로직이다.

주목할 점은, 1만큼 증가시키는 **판단을 누가 하는가 이다.**

run2같은경우, 숫자를 넘겨줄 뿐, 상태값을 변경시키는 **판단**을 객체에게 맡기고 있다.

하지만 run같은 경우, getter를 통해 값을 받은 후 그 값을 이용하여 객체의 상태값을 변화시키는 판단을 하게되고, update메소드를 호출함으로서 객체의 상태값을 바꾼다.

**즉, 객체의 상태값을 바꾼다는 판단을 외부에서 하고 있는 것이다.**

getter메소드가 직접적으로 상태값을 바꾸지는 않지만, 이것이 사용되는것은 외부에게 '상태값 변경에 대한 판단권'을 줘버리게 될 수 있다.

이것은 계속해서 되새기고 있는 '독립적인 객체'설계에 위배되는 행위이다.



## Constructor 접근 제한

NoArgsConstructor 접근권한을 주지 않고, 정적 팩토리 메서드를 사용한다면,
JPA에서는 프록시를 생성을 위해서 기본 생성자를 반드시 하나를 생성해야한다.
이때 접근 권한을 AccessLevel.PROTECTED로 설정하여 JPA에서의 Entity 클래스 생성만 허용해줍니다.
