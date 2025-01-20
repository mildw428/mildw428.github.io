---
layout: default
title: Request API - Feign Client
parent: git
---

설정 코드 : https://github.com/mildw/doraemon/blob/main/request-api/src/main/java/com/example/requestapi/application/FeignClientService.java

다른 서버로 요청할때 사용하는 라이브러리입니다.
Spring의 REST Template보다 더 간단하고 직관적인 코드를 작성할 수 있습니다.

exception 처리시 참고
```java
  
private static FeignClientException clientErrorStatus(int status,  
                                                      String message,  
                                                      Request request,  
                                                      byte[] body,  
                                                      Map<String, Collection<String>> headers) {  
  switch (status) {  
    case 400:  
      return new BadRequest(message, request, body, headers);  
    case 401:  
      return new Unauthorized(message, request, body, headers);  
    case 403:  
      return new Forbidden(message, request, body, headers);  
    case 404:  
      return new NotFound(message, request, body, headers);  
    case 405:  
      return new MethodNotAllowed(message, request, body, headers);  
    case 406:  
      return new NotAcceptable(message, request, body, headers);  
    case 409:  
      return new Conflict(message, request, body, headers);  
    case 410:  
      return new Gone(message, request, body, headers);  
    case 415:  
      return new UnsupportedMediaType(message, request, body, headers);  
    case 429:  
      return new TooManyRequests(message, request, body, headers);  
    case 422:  
      return new UnprocessableEntity(message, request, body, headers);  
    default:  
      return new FeignClientException(status, message, request, body, headers);  
  }  
}  
  
private static FeignServerException serverErrorStatus(int status,  
                                                      String message,  
                                                      Request request,  
                                                      byte[] body,  
                                                      Map<String, Collection<String>> headers) {  
  switch (status) {  
    case 500:  
      return new InternalServerError(message, request, body, headers);  
    case 501:  
      return new NotImplemented(message, request, body, headers);  
    case 502:  
      return new BadGateway(message, request, body, headers);  
    case 503:  
      return new ServiceUnavailable(message, request, body, headers);  
    case 504:  
      return new GatewayTimeout(message, request, body, headers);  
    default:  
      return new FeignServerException(status, message, request, body, headers);  
  }  
}
```
