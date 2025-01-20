---
layout: default
title: port forwarding
parent: memo
---
# port forwarding
---

특정 포트 번호를 허용하여 내 pc에 접근할 수 있도록 할 수 있다.

ipconfig 명령어를 통해 내부 ip주소를 알 수 있다.

결과 : 192.168.219.184

공유기 ip 주소 : 192.168.219.1

id : admin
password : mac주소 끝의 6자리 + `_admin`

접속 후 포트포워드 가서 설정하고,
제어판의 보안설정에 접근하여 방화벽 설정을 풀어주자.

근데 왜안됨?

확인해보니 방화벽은 잘 풀린상태였다.
원격 접속을 위해 윈도우기능으로 원격 기능을 키고 skb설정에서 3389 번 포트를 허용하니 잘 동작함.
그리고 윈도우의 원격접속 기능을 끄면 포트포워딩이 fail이 뜬다.
즉 포트만 여는게 아니라 받아주는 서버가 있어야 한다는것.

마찬가지로 mysql서버를 설치 후 동작시키면 ture가 뜬다.

하지만 내가 하고싶은건 도커컨터네이너 동작이기 떄문에 아직 헤매는중이다

그에대한 해결 방법 : https://codeac.tistory.com/118

```shell
netsh interface portproxy show all
```
적용 확인

포트 포워딩을 리눅스에서 ifconfig를 통해 정보를 찾는다.
en0으로 시작하는 부분에서 inet주소가 연결되어야 하는 주소이다.

```shell
netsh interface portproxy add v4tov4 listenport=$port listenaddress='0.0.0.0' connectport=$port connectaddress=$my_wsl_address
```
`$my_wsl_address` 는 정확하게는 리눅스에서 ifconfig를 통해 나온주소를 입력해야한다.
파워쉘에서 ipconfig를 통해 나온 주소로 입력해서 계속 갸우뚱했다.

해결!
