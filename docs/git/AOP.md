---
layout: default
title: AOP
parent: git
---

설정 코드 : https://github.com/mildw/doraemon/blob/main/aop/src/main/java/com/example/aop/aop/Advice.java

@Before : 메소드가 실행되기 이전에 실행됩니다.
@After : 메소드의 종료 후 무조건 실행됩니다. (try-catch에서 **finally**와 같은 동작)
@After-returning : 메소드가 성공적으로 완료되고 리턴한 다음에 실행됩니다.
@After-throwing : 메소드 실행 중 예외가 발생하면 실행됩니다. (try-catch에서 **catch**와 같은 동작)
@Around : 메소드 호출 자체를 가로채서 메소드 실행 전후에 처리할 로직을 삽입할 수 있습니다.

---

@Around("execution(* com.example.MyClass.myMethod(..))")
	- 위 예시에서는 `com.example.MyClass` 클래스의 `myMethod()` 메서드 호출 시 어드바이스가 실행됩니다.

@Around("within(com.example.MyPackage)")
	- 위 예시에서는 `com.example.MyPackage` 패키지에 속하는 모든 메서드 호출 시 어드바이스가 실행됩니다.

@Around("args(String)")
	- 위 예시에서는 첫 번째 인수가 `String` 타입인 메서드 호출 시 어드바이스가 실행됩니다.

```java
@Slf4j  
@Aspect  
@Order(1)  
@Component  
@RequiredArgsConstructor  
public class TaskAdvice {  
    
	@Around("execution(* com.example.aop..application..*Service.*(..))")  
	public Object testAdvice(ProceedingJoinPoint joinPoint) throws Throwable {  
	  
	    Object o = null;  
	    try {  
	        log.info("메소드 실행 전");  
	        o = joinPoint.proceed();  
	        log.info("메소드 실행 후");  
	    } catch (Exception e) {  
	        log.error(e.getMessage(), e);  
	    }  
	    return o;  
	} 
  
}
```

- @Around("execution(* com.example.aop..application..*Service.*(..))")
	- **타입:** 메서드 실행
	- **패키지:**
	    - `com.example.aop` 패키지 하위의 모든 패키지
	    - `application` 패키지 포함
	- **클래스:**
	    - `Service`로 끝나는 모든 클래스
	- **메서드:**
	    - 모든 메서드 이름
	    - 모든 매개변수

> testAdvice함수가 void라면 함수 호출 후 null을 리턴하므로 주의해야합니다.


```
2024-03-16 23:16:39.928  INFO 10150 --- [nio-8080-exec-8] com.example.aop.aop.TaskAdvice           : 메소드 실행 전
Hibernate: 
    select
        member0_.`id` as id1_0_,
        member0_.`name` as name2_0_ 
    from
        `member` member0_
2024-03-16 23:16:41.461  INFO 10150 --- [nio-8080-exec-8] com.example.aop.aop.TaskAdvice           : 메소드 실행 후
```

---

멀티 테넌시 사용시 루프문을 사용하여 여러 테넌트를 한번에 동작 시킬 수 있습니다.
```java
@Slf4j  
@Aspect
@Order(1)
@Component
@RequiredArgsConstructor  
public class TaskAdvice {
  
    @Around("execution(* com.example..task.tenant..*Task.*(..))")  
    public void tenantTaskAdvice(ProceedingJoinPoint joinPoint) throws Throwable {  

        List<TenantContext> tenantContexts = new ArrayList<>();
		tenantContexts.add(new TenantContext(null,"test1"));
		tenantContexts.add(new TenantContext(null,"test2"));
  
        for (TenantContext tenantContext : tenantContexts) {  
            try {  
                TenantContextHolder.setTenantContext(tenantContext);  
                LocaleContextHolder.setLocale(Locale.KOREA);  
                EnvironmentContextHolder.setEnvironmentContext(environment);  
  
                joinPoint.proceed();  
            } catch (Exception e) {  
                log.error(e.getMessage(), e);  
            } finally {  
                TenantContextHolder.resetTenantContext();  
                LocaleContextHolder.resetLocaleContext();  
                EnvironmentContextHolder.resetEnvironmentContext();  
            }  
        }  
    }  
  
}

```