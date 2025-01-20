---
layout: default
title: Swagger UI
parent: git
---

설정 코드 : https://github.com/mildw/doraemon/blob/main/swagger-ui/src/main/java/com/example/swaggerui/config/swagger_ui/SwaggerConfig.java


**Springdoc 설정**

```yaml
springdoc:  
  swagger-ui:  
    path: /swagger-ui  
    disable-swagger-default-url: true  
    display-request-duration: true  
    tags-sorter: alpha  
    operations-sorter: alpha  
    doc-expansion: none  
    urls-primary-name: CEC API  
    persist-authorization: true  
    query-config-enabled: true  
    syntax-highlight:  
      theme: nord  
  pre-loading-enabled: true
```

- **path: /swagger-ui** (기본값): 스웨거 UI에 접근할 수 있는 경로를 정의합니다.
  이 경우 `http://localhost:8080/swagger-ui`에서 접근할 수 있습니다.
  
- **disable-swagger-default-url: true** (기본값: false): 기본 스웨거 UI URL (`/swagger-ui.html`) 자동 생성을 비활성화합니다.

- **display-request-duration: true** (기본값: false): 스웨거 UI에 요청 실행 시간을 표시합니다.

- **tags-sorter: alpha** (기본값: none): API 태그를 알파벳 순으로 정렬합니다.

- **operations-sorter: alpha** (기본값: none): 각 태그 내에서 API 작업을 알파벳 순으로 정렬합니다.

- **doc-expansion: none** (기본값: list): API 문서 확장의 초기 상태를 설정합니다. `none`은 모든 것을 축소된 상태로 유지합니다. 다른 옵션으로는 `list` (최상위 섹션 확장) 및 `full` (모든 항목 확장)이 있습니다.

- **urls-primary-name: TEST API** (기본값: 애플리케이션 이름): 스웨거 UI 제목에 표시되는 기본 이름을 설정합니다.

- **persist-authorization: true** (기본값: false): 스웨거 UI 세션 간에 권한 정보를 기억합니다.

- **query-config-enabled: true** (기본값: false): URL의 쿼리 매개변수를 사용하여 스웨거 UI를 구성할 수 있도록 합니다.

- **syntax-highlight:** 스웨거 UI의 코드 샘플에 대한 구문 강조 표시를 활성화합니다.
    - **theme: nord** (기본값: github): 구문 강조 표시 테마를 "nord"로 설정합니다.

**추가 참고 사항**
- **pre-loading-enabled: true** (기본값: false): 이 옵션은 Springdoc 문서에 명시적으로 설명되어 있지는 않지만, 더 빠른 초기 렌더링을 위해 스웨거 UI를 사전 로딩하는 것과 관련이 있을 수 있습니다. 정확한 확인을 위해서는 사용 중인 Springdoc 버전의 문서를 확인하는 것이 좋습니다.
