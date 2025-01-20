---
layout: default
title: Spring Cloud Function
parent: git
---

s설정 코드 : https://github.com/mildw/doraemon/blob/main/spring-cloud-function/src/main/java/com/example/springcloudfunction/LambdaHandler.java

Lambda에 Spring Cloud Function 프로젝트를 배포하기 위한 과정을 다룹니다.

```groovy
  
// For Local Test  
implementation 'org.springframework.cloud:spring-cloud-starter-function-web:3.1.6'  
// Spring Cloud Function  
implementation 'org.springframework.cloud:spring-cloud-function-adapter-aws:3.1.6'

implementation 'com.amazonaws:aws-lambda-java-core:1.2.1'  

implementation 'com.amazonaws:aws-lambda-java-events:3.11.0'

```
spring cloud function에 필요한 의존성을 추가합니다.
lambda에 필요한 의존성을 추가합니다.

```java
@Slf4j
@Component  
public class ExampleFunction implements Function<String, String> {  
  
    @Override  
    public String apply(String request) {  
  
        log.info(request);  
  
        return request;  
    }  
}
```
Funtion, Consumer, Supplier와 같은 함수형 인터페이스를 구현합니다.

> `implementation 'org.springframework.cloud:spring-cloud-starter-function-web:3.1.6'`
> 위에서 선언한 의존성을 통해 로컬환경에서 테스트할 수 있습니다.

```bash
curl -X POST -H "Content-Type: application/json" -d '{"body":"1111"}' http://localhost:8080/exampleFunction 
```

```
- 결과값
{"body":"1111"}
```

해당 프로젝트를 aws에 올리기전에

```java
public class LambdaHandler extends SpringBootStreamHandler {   
}
```
Lamda가 Spring Cloud Function 프로젝트를 다루기 위해서 Spring Cloud Function AWS에서 제공하는 핸들러를 상속받은 클래스가 필요합니다.

다음으로 핸들러 정보 입력이 필요합니다.
`com.example.springcloudfunction.LambdaHandler`

![[Pasted image 20240305213820.png]]

환경변수의 입력도 필요합니다. 

`FUNCTION_NAME`에서 맨앞의글자가 소문자인점을 고려합니다.
`MAIN_CLASS`에는 메인메서드가 있는 클래스 경로를 입력합니다.
![[Pasted image 20240305213813.png]]

이후 테스트를하면 결과값을 확인 할 수 있습니다.
![[Pasted image 20240406125203.png]]
