---
layout: default
title: DTO pattern
parent: memo
---
# DTO pattern
---

https://climbtheladder.com/10-dto-best-practices/
https://www.baeldung.com/java-dto-pattern

The solution is to create a Data Transfer Object that can hold all the data for the call.


Use a builder pattern
- you can use a builder pattern, which allows you to set the required fields in the constructor and the optional fields using setter methods.
- setter를 사용하지말고, builder를 사용해서 생성자를 만들때도 setter를 사용할 필요가 없다.

Make the class immutable (수정x)
- Just make all of the fields in the DTO final
- don’t provide any setter methods.
- Instead, provide a constructor that takes all of the required data as parameters.
- dto 내부로직 구현, 사용 금지

Don’t use inheritance (상속x)
- It’s much better to simply use composition when creating your DTOs.
- 상속받아서 만들지말고, dto를 여러개 만드는게 수정에 용이하다.

Avoid using getters and setters
- Getters and setters are a code smell because they indicate that your class is doing too much.
- A data transfer object should only be responsible for transferring data
- Instead of using getters and setters, make your fields public and final.
- getter와 setter가 있다는건 클래스가 많은 일을하고있다는것을 증명하고, 이는 악취나는 코드이다.
- dto는 응답을 위해서만 존재해야한다.
- 변수를 final로 선언해여 클래스를 간결하고 이해하기 쉽게 만들자.

Keep it simple
- A DTO should only contain the fields that are absolutely necessary to complete the task at hand.
- dto 내부의 변수는 절대적으로 필요한 변수만 넣는다. 여분의 변수는 지워져야한다.
- 이는 클래스를 가볍게 만들어줄뿐아니라 민감데이터가 유출되는것을 막아준다.
- dto 내부에는 절대 비즈니스 로직이 들어가선 안된다.

Validate input data
- dto는 값을 담는 컨테이너일 뿐이란것을 잊지 말아야한다.
- 컨테이너에 들어가는 값들은 매우 중요하며 이것들은 검증되어야한다.
- 모르겠다면 공부하자,,, (test code를 돌리라는 소리일까?)

Consider serialization
- 데이터 직렬화를 고려하자. (공부 결과 직렬화에는 변수 변경에대해 위험이 따르며, 잘 알고사용해야한다.)
- 아직 지식이 부족한 나는 지양 해도 좋을거같다.

Be careful with equals() and hashCode()


Prefer composition over inheritance
- 부모클래스의 변경이 자식클래스에게 영향을 끼칠 수 있기떄문에, 상속에 대한 유혹을 뿌리쳐야 한다.
- 상속보다는 새로 구성하는쪽을 선호해야한다.

Leverage Java 8 features
-  Date and Time API, lambdas, and Streams을 활용하자
- Date Time 은 날짜 포멧변경을 쉽게 만들어준다. 람다는 필터와 변환에 도움을주며 스트림은 병렬처리를 가능하게 해준다.


getter는 직렬화할 때 getter를 사용합니다. 즉 DTO들을 JSON 데이터로 다시 가공할 때 getter를 사용한다고 이해


https://www.baeldung.com/java-dto-pattern

Fowler explained that the pattern's main purpose is to reduce roundtrips to the server by batching up multiple parameters in a single call. This reduces the network overhead in such remote operations.
여러 데이터를 한번에 묶어 처리함으로써 서버와의 통신을 줄임. 


DTOs normally are created as [POJOs](https://www.baeldung.com/java-pojo-class). They **are flat data structures that contain no business logic.** They only contain storage, accessors and eventually methods related to serialization or parsing.


-   Dto는 사용에 따라 최대한 쪼개야 합니다. 예를 들어 하나의 Entity를 용도에 따라 보여주는 부분이 다를 때, 화면에서의 리스트 컬럼과 엑셀 다운로드 시 보여줄 컬럼들이 다를 때 필드가 1-2개 달랐지만 하나의 Dto로 사용했었습니다. 각각의 목적에 따라 Dto를 만들어야 합니다.
