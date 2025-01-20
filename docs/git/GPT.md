---
layout: default
title: GPT
parent: git
---

설정 코드 : https://github.com/mildw/doraemon/blob/main/gpt/src/main/java/com/example/gpt/application/gpt/GptService.java

GPT와 대화할때의 사용자 경험은
모든 텍스트가 완성된 후에 한번에 출력되는 것이 아닌, 스트림 방식으로 출력되게 된다.

이는 WebFlux를 사용하여 스트림 방식으로 데이터를 받아 출력하면 된다.
아래는 예제 코드이다.

```java
public Flux<String> streamResponse(String prompt) {  
    GptRq request = new GptRq("gpt-3.5-turbo", true, prompt);  
    return webClient  
            .post()  
            .uri(getGptUri())  
            .header("Authorization", "Bearer " + apiKey)  
            .contentType(MediaType.APPLICATION_JSON)  
            .bodyValue(request)  
            .retrieve()  
            .bodyToFlux(OpenAIResponse.class)  
            .takeWhile(response -> response.getChoices() != null  
                    && response.getChoices().stream().noneMatch(choice -> "stop".equals(choice.getFinish_reason())))  
            .flatMap(response -> Flux.fromIterable(response.getChoices()))  
            .map(choice -> choice.getDelta().getContent())  
            .filter(content -> content != null && !content.isEmpty())  
            .onErrorResume(e -> Flux.empty());  
}
```

일정하게 오는 객체를 파악하여 OpenAIResponse.class를 정의하여`.bodyToFlux(OpenAIResponse.class)`코드를 통해 변객체로 변환한다.

전달을 마무리하는 객체에는 response.getChoices().get(0)에 stop이라는 String값이 온다. 대게 0번째로 전달온다고 하지만 안정성을 위해 null처리 또는  stream().anyMatch()를 통해 처리한다.



