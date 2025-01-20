---
layout: default
title: csrf
parent: memo
---
# csrf
---

## cross site request forgery attack

CSRF 공격이란, 인터넷 사용자(희생자)가 자신의 의지와는 무관하게
공격자가 의도한 행위(등록, 수정, 삭제 등)를 특정 웹사이트에 요청하도록 만드는 공격

---
### get

```
GET http://bank.com/trasnfer?accountNumber=1122&amount=10000
```

위 예시는 로그인을 한 사용자가 **특정 계좌**(**"1122"**)로 **금액**(**10000**)을 이체하기 위해 사용하는 GET 요청

```html
<a href="http://bank.com/transfer?accountNumber=5678&amount=10000">
Pictures@
</a>
```
Link(링크): 공격자는 이체를 실행하도록 하기 위해 사용자(희생자)가 링크를 클릭하도록 유도

```html
<img src="http://bank.com/transfer?accountNumber=5678&amount=10000"/>
```
  Image(이미지): 이미지 태그를 사용하여 클릭하지 않더라도 페이지가 로드되면 요청은 자동적으로 실행

### post

```html
<form action="http://bank.com/transfer" method="POST">
    <input type="hidden" name="accountNumber" value="5678"/>
    <input type="hidden" name="amount" value="10000"/>
    <input type="submit" value="Pictures@"/>
</form>
```
post의 경우 클릭을 유도할 수 도, 이미지로 로드할 수도 없다.
공격자는 `from` 태그가 필요

```html
<body onload="document.forms[0].submit()">

<form>
```

자바스크립트를 사용한다면 양식을 자동으로 전송(submit) 하도록 설정할 수 있다.


## Spring Security CSRF Token

임의의 토큰을 발급한 후 자원에 대한 변경 요청일 경우 Token 값을 확인한 후 클라이언트가 정상적인 요청을 보낸것인지 확인

만약 CSRF Token이 존재하지 않거나, 기존의 Token과 일치하지 않는 경우 4XX 상태코드를 리턴

Spring Security에서는 `@EnableWebSecurity` 어노테이션을 지정할 경우 자동으로 CSRF 보호 기능이 활성화된다.

따라서 CSRF 비활성화가 필요할 경우 아래와 같이 csrf().disable() 설정을 추가합니다.

### Rest api에서의 CSRF

그래서 이렇게 보안 수준을 향상시키는 CSRF를 왜 disable 하였을까? spring security documentation에 non-browser clients 만을 위한 서비스라면 csrf를 disable 하여도 좋다고 한다.

![[Pasted image 20230116111026.png]]

이 이유는 rest api를 이용한 서버라면, session 기반 인증과는 다르게 stateless하기 때문에 서버에 인증정보를 보관하지 않는다. rest api에서 client는 권한이 필요한 요청을 하기 위해서는 요청에 필요한 인증 정보를(OAuth2, jwt토큰 등)을 포함시켜야 한다. 따라서 서버에 인증정보를 저장하지 않기 때문에 굳이 불필요한 csrf 코드들을 작성할 필요가 없다.
