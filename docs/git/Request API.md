---
layout: default
title: Request API
parent: git
---

설정 코드 : https://github.com/mildw/doraemon/blob/main/request-api/src/main/java/com/example/requestapi/config/feign/FeignClientConfig.java

백엔드 서버에서 다른 백엔드 서버로 통신 요청할때 사용하는 라이브러리 입니다.

[[Request API - RestTemplate]]
- 다른 서버로 요청할때 사용하는 라이브러리입니다.
- 동기식 호출 방식이기 때문에 단순한 요청 처리에 사용됩니다.
- Spring에서는 Web Client사용을 권고하고있습니다.

[[Request API - Web Client]]
- 다른 서버로 요청할때 사용하는 라이브러리입니다.

[[Request API - Feign Client]]
- 다른 서버로 요청할때 사용하는 라이브러리입니다.
- Spring의 REST Template보다 더 간단하고 직관적인 코드를 작성할 수 있습니다.

