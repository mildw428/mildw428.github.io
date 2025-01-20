---
layout: default
title: CompletableFuture
parent: git
---

설정 코드 : https://github.com/mildw/doraemon/blob/main/async/src/main/java/com/example/async/application/AsyncTestService.java

CompletableFuture는 Future의 확장된 형태입니다.

비동기 계산의 결과를 나타내는데 사용되며, 작업이 완료되면 결과를 포함하는 CompletableFuture를 반환합니다.

CompletableFuture 는 Future와 달리 비동기 작업의 조합, 합성, 처리 결과에 따른 다양한 조작을 지원합니다.

여러 CompletableFuture를 조합하여 병렬로 실행하고 모든 작업이 완료될 때까지 기다릴 수 있습니다. 

또한 작업이 완료되면 다른 작업을 수행하거나 예외 처리를 할 수 있습니다.

CompletableFuture는 콜백 함수를 통해 비동기 작업의 성공 또는 실패를 처리할 수 있습니다.

```java
    
public void test() throws ExecutionException, InterruptedException {  
    ExecutorService executor = Executors.newCachedThreadPool();  
  
    CompletableFuture<String>[] futures = new CompletableFuture[3];  
    for(int i=0; i<3; i++) {  
        futures[i] = CompletableFuture.supplyAsync(() -> {  
            System.out.println("call");  
            return requestService.request();  
        }, executor);  
    }  
    CompletableFuture<Void> allOfFuture = CompletableFuture.allOf(futures);  
    allOfFuture.join();  
    System.out.println("done");  
}
```

```java
public String request() {  
    String result = localTestClient.test();  
    System.out.println("method : "+result);  
    return result;  
}
```

```
call
call
call
method : test
method : test
method : test
done
```

- `.join()`: 이 메서드는 CompletableFuture가 완료될 때까지 기다린 후 결과를 반환합니다. 만약 CompletableFuture가 예외로 완료된 경우에는 해당 예외를 throw합니다. 이는 checked exception이 아니기 때문에 예외를 처리할 필요가 없습니다.

- `.get()`: 이 메서드는 CompletableFuture가 완료될 때까지 기다린 후 결과를 반환합니다. 하지만 `.join()`과 다르게 CompletableFuture가 예외로 완료된 경우에는 ExecutionException으로 감싸서 throw합니다. 이는 checked exception이기 때문에 예외 처리가 필요합니다.

upplyAsync():
    - 비동기 작업을 실행하고 **결과를 반환**하는 CompletableFuture를 생성합니다.

runAsync():
    - 비동기 작업을 실행하고 **결과를 반환하지 않는** CompletableFuture를 생성합니다.

 thenApply():
    - 이전 작업의 **결과를 가공하여 새로운 값을 반환**하는 작업을 연결합니다.

thenAccept():
    - 이전 작업의 결과를 인자로 받아서 해당 **결과를 소비하는 작업**을 연결합니다.

thenRun():
    - 이전 작업의 **결과에 의존하지 않고 단순히 다른 작업을 실행**하는 작업을 연결합니다.

thenCombine():
    - **두 개의 CompletableFuture 작업이 모두 완료되었을 때 결과를 조합하는 작업**을 연결합니다.

thenCompose():
    - 이전 작업의 **결과에 기반하여 비동기적으로 다른 작업을 실행하고 결과를 제공**하는 작업을 연결합니다.
    - Function을 인자로 받아서 이전 결과에 따라 새로운 작업을 수행합니다.

exceptionally():
    - 예외가 발생하면, 해당 예외를 처리하고 새로운 값을 반환할 수 있습니다.
```java
CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> {
    // 예외 발생
    throw new RuntimeException("Exception occurred");
}).exceptionally(ex -> {
    System.out.println("Handled exception: " + ex.getMessage());
    return 0; // 예외 처리 후 반환할 값
});

System.out.println("Result: " + future.join()); // Result: 0

```

handle():
    - 작업이 성공적으로 완료되거나 예외를 발생시켰을 때 결과나 예외를 처리하는 작업을 연결합니다.
    - exceptionally()와 달리, 정상적인 결과나 예외 모두를 처리합니다.

```java
CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> {
    // 예외 발생
    throw new RuntimeException("Exception occurred");
}).handle((result, ex) -> {
    if (ex != null) {
        System.out.println("Handled exception: " + ex.getMessage());
        return 0; // 예외 처리 후 반환할 값
    } else {
        return result; // 정상적인 결과
    }
});

System.out.println("Result: " + future.join()); // Result: 0

```

allOf():
    - 여러 개의 CompletableFuture가 모두 완료될 때까지 기다린 후 새로운 CompletableFuture를 반환합니다.
    - 모든 작업이 완료될 때까지 기다린 후 결과를 처리할 수 있습니다.

anyOf():
    - 여러 개의 CompletableFuture 중 하나라도 완료될 때까지 기다린 후 새로운 CompletableFuture를 반환합니다.
    - 가장 빨리 완료되는 작업의 결과를 처리할 수 있습니다.


## Executor

작업의 종류에 따른 Executor 선택

CompletableFuture를 사용할 때는 작업의 종류에 따라 다른 Executor를 선택할 수 있습니다.

CompletableFuture를 생성할 때 Executor를 명시적으로 지정하지 않으면 기본적으로 ForkJoinPool.commonPool()이 사용됩니다. 이는 보통 CPU 코어의 수에 기반하여 크기가 조정되는 스레드 풀입니다.

예를 들어, CPU 집약적인 계산을 수행하는 작업에는 고정 크기의 스레드 풀이 적합할 수 있습니다.
반면에 I/O 작업을 수행하는 경우에는 CachedThreadPool이나 WorkStealingPool이 더 적합할 수 있습니다.

CachedThreadPool은 동적 크기의 스레드 풀로, I/O 및 CPU 바운드 작업에 사용될 수 있습니다. 반면에 WorkStealingPool은 ForkJoinPool을 기반으로 하며, CPU 바운드 작업을 처리하는 데 사용됩니다. WorkStealingPool은 작업을 분산하여 처리하기 때문에 특히 병렬 처리에 유리합니다.
