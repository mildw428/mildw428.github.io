---
layout: default
title: Async
parent: git
---

```java
  
@EnableAsync  
@SpringBootApplication  
public class AsyncApplication extends SpringBootServletInitializer {  
  
    public static void main(String[] args) {  
       SpringApplication.run(AsyncApplication.class, args);  
    }  
  
}
```

```java
@Service  
@RequiredArgsConstructor  
public class RequestService {  
  
    private final LocalTestClient localTestClient;  
  
    @Async  
    public void request() {  
        System.out.println(localTestClient.test());  
    }  
      
}
```

```java
@Service  
@RequiredArgsConstructor  
public class AsyncTestService {  
  
    private final RequestService requestService;  
  
    public void test() {  
        for(int i=0; i<3; i++) {  
            System.out.println("call");  
            requestService.request();  
        }  
    }  
  
}
```

```
//결과
call
call
call
test
test
test
```


Spring Boot의 기본 스레드 풀 설정은 다음과 같습니다.

- `corePoolSize`: 1
- `maxPoolSize`: Integer.MAX_VALUE
- `queueCapacity`: Integer.MAX_VALUE
- `threadNamePrefix`: "task-"

이러한 기본 설정은 작은 규모의 애플리케이션 또는 간단한 비동기 작업을 처리하는 데 적합합니다. 그러나 상용 서비스 또는 고성능 요구 사항이 있는 애플리케이션의 경우 이러한 기본 설정을 조정하여 적합한 스레드 풀을 구성하는 것이 좋습니다.

따라서 특별한 경우가 아니라면 명시적으로 스레드 풀을 지정하지 않고도 Spring Boot의 기본 스레드 풀을 사용할 수 있습니다.

```
@Configuration
@EnableAsync
public class AsyncConfig {

    @Bean(name = "asyncExecutor")
    public Executor asyncExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(3); // 기본 스레드 풀 크기
        executor.setMaxPoolSize(10); // 최대 스레드 풀 크기
        executor.setQueueCapacity(500); // 작업 대기 큐 크기
        executor.setThreadNamePrefix("Async-"); // 스레드 이름 접두사
        executor.initialize();
        return executor;
    }
}

```

---

### 동기 처리 및 DB 커넥션 관리
```yaml
spring:
  datasource:
    hikari:
      maximum-pool-size: 10

```

- Spring Boot와 HikariCP를 함께 사용할 때 비동기 처리를 위해 `@Async` 애너테이션 또는 `CompletableFuture`와 같은 기술을 사용할 수 있습니다.
- 비동기 메서드가 실행될 때 Spring은 쓰레드를 풀에서 가져와 해당 메서드를 실행하게 됩니다.
- HikariCP는 커넥션 풀을 관리하기 위해 사용됩니다. 이것은 요청이 들어올 때마다 커넥션을 가져와서 사용하고, 처리가 완료되면 해당 커넥션을 풀에 반환합니다.

>**주의 사항**
>비동기 처리를 할 때는 DB 작업이 반드시 비동기로 수행되어야 합니다. 그렇지 않으면 스레드 풀이 막히거나, 불필요한 스레드가 생성될 수 있습니다. 따라서 적절한 비동기 메서드 및 기술을 사용하여 비동기 작업을 구현해야 합니다.
