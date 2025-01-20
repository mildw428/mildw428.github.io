---
layout: default
title: OIDC
parent: memo
---
# OIDC
---

OIDC 와 Oauth2에 대해 알아보자.

필자의 회사에서는 여러 웹 서비스를 운영 하고있다.
회사가 커지고 구상하는 사업이 많아 질 수록, 접근할 수 있는 웹서비스는 많아졌다.
이는 반복적인 로그인 로직 개발을 하게 만들었고,
사용자들이 회사의 제품을 계약해서 사용할 때 마다 각각의 서비스별로 로그인을 해야하는 불편함을 야기했다.
그래서 OIDC를 이용한 통합 로그인을 도입하게 되어, 본인 또한 공부하고 적용하려고한다.
### OIDC란?

OIDC를 검색하면 OAuth2.0에 대한 글이 많이 보인다.
OIDC가 OAuth2.0 기반으로 동작하기 때문에 많이 헷갈렸다.
요약하면 OAuth2.0은 3rd Party의 API 사용"인가"가 목적이고,
OIDC는 통합 "인증", SSO(single-sign-on)가 목적이다.

> SSO : 구글에 로그인 함으로써 구글 드라이브, Gmail, You Tube에 각각 로그인 필요없이 접근할 수 있다.

사용자는 OIDC를 통해 인증을 하여 인증 코드(authentication code)를 반환받고,
해당 코드를 통해 OAuth2.0를 기반으로 접근 권한 토큰(access token)을 발급받아
이를통해 서비스를 사용할 수 있는 권한을 인가 받아 자유롭게 서비스를 사용한다.

OIDC가 OAuth2.0 기반이므로 플로우는 유사하다.
OIDC는 access_token외에 id token이라는 것을 추가로 받게 된다.
id token에는 사용자 정보가 들어있다.
때문에 백엔드 서버(자사 또는 본인의)에서는 사용자 정보를 가져오기 위해
사용자의 정보를 가지고있는 Resource Server로 한번더 api를 호출하지 않아도 된다.

id token은 jwt 토큰형식을 가지며,
```json
{
   "iss": "https://server.example.com",
   "sub": "24400320",
   "aud": "s6BhdRkqt3",
   "exp": 1311281970,
   "iat": 1311280970
}
```
위와같은 정보 또는 공식문서에 여러 값들을 가질 수 있다.
또한 사용자가 원하는 값들 넣을 수 있다. (ex. email)
[참고용 openid 공식 문서](https://openid.net/specs/openid-connect-basic-1_0.html)

---

### 알아두면 좋은 용어

Resource Owner
서비스 이용자 - 개인정보(자원-Resource)를 소유하는 자

Client (Relying Party)
자사 또는 개인이 만든 애플리케이션 서버
클라이언트 라는 이름은 client가 Resource server에게 필요한 자원을 요청하고 응답하는 관계여서 그렇다고 한다.

Authorization Server
권한을 부여(인증에 사용할 아이템을 제공주는)해주는 서버다. 
사용자는 이 서버로 ID, PW를 넘겨 Authorization Code를 발급 받을 수 있다.  

Resource Server
사용자의 개인정보를 가지고있는 애플리케이션 (Google, Facebook, Kakao 등) 회사 서버  
Client는 Token을 이 서버로 넘겨 개인정보를 응답 받을 수 있다.

Access Token
자원에 대한 접근 권한을 Resource Owner가 인가하였음을 나타내는 자격증명
Refresh Token
Client는 Authorization Server로 부터 access token과 refresh token을 함께 부여 받는다.

token은 가지고만 있어도 자격이 증명되기때문에, 백엔드 서버는 프론트 서버와 통신할때
상대적으로 짧은 만료 기간을 가진 access token으로만 주로 통신하며,
프론트 서버는 access token의 만료기간이 다가옴에 따라 refresh token을 통해 access token을 새로 발급받는다.
만료된 access token를 사용하면 웹 서비스 사용자는 로그아웃 처리 될 것이다.

---

### Flow

![[OIDC-flow.png]]

OpenId Provider = Authorization Server(인증 서버)
Token Endpoint = Resource Server 또는 구조에 따라 token 발급 서버를 따로 둔다.
UserInfo Endpoint = Resource Server
Callback URL은 네이버 로그인 인증 과정에서 사용자 인증 결과를 전달받아 처리하기 위한 URL이다.



