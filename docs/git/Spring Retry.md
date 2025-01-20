---
layout: default
title: Spring Retry
parent: git
---

가끔 로직 수행하다 실패한 경우 재시도가 필요한 경우가 있습니다.

다양한 방법이 있지만 Spring Retry를 사용해서 해결할 수 있습니다.

```groovy
// Spring Retry  
implementation 'org.springframework.retry:spring-retry'
// AOP
implementation 'org.springframework:spring-aspects'
```

```java
@EnableRetry  
@SpringBootApplication  
public class AsyncApplication extends SpringBootServletInitializer {  
  
   public static void main(String[] args) {  
      SpringApplication.run(AsyncApplication.class, args);  
   }  
  
}
```
@EnableRetry 추가

```java
@Retryable(maxAttempts = 2, backoff = @Backoff(delay = 3000))  
public Boolean runtimeException() {  
    String result = localTestClient.runtimeException();  
    System.out.println("method : "+result);  
    return true;}
```
@Retryable(maxAttempts = 2, backoff = @Backoff(delay = 3000))  추가
maxAttempts 기본값은 3입니다.

RetryTemplate Configuration 설정방법도 있으니 좀 더 알아보기
