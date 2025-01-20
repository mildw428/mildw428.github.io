---
layout: default
title: CICD
parent: 실습
grand_parent: aws
---
# CICD
---

[Foundation labs](https://catalog.us-east-1.prod.workshops.aws/workshops/869f7eee-d3a2-490b-bf9a-ac90a8fb2d36/en-US/4-basic)

  
**Lab 0. CodeCommit**

- 실습을 전부 진행하고 [Repositories]-[Code] 의 hello.txt 내용을 캡쳐
![[스크린샷 2024-04-14 오전 10.36.46.png]]

- 실습을 전부 진행하고 [Repositories]-[Commits]-[Commit visualizer] 의 commit 내용을 캡쳐
![[스크린샷 2024-04-14 오전 10.41.16.png]]

---

**Lab 1. Rolling Update**

- **[1] 레포지토리 만들기**
	- [Repositories]-[Code] 에서 hello-server 캡쳐
![[스크린샷 2024-04-14 오전 10.55.35.png]]
- **[2] ECS 리소스 생성**
	- [AWS ECS]-[Clusters]-staging 선택 후, 서비스 생성 화면 캡쳐
![[스크린샷 2024-04-14 오전 11.01.00.png]]
- **[3] 파이프라인 만들기**
	- [AWS Pipelines]-[Pipeline]-[Pipeline]-hello-server-pipeline 선택 후, 파이프라인 화면 캡쳐
![[스크린샷 2024-04-14 오전 11.21.16.png]]
- **[4] 파이프라인 실행**
	- curl 명령어 실행 후, 정상적으로 서비스가 호출되는지 확인 & 캡쳐
![[스크린샷 2024-04-14 오전 11.22.40.png]]
- **[5] 변경 내용 배포**
	- curl 명령어 실행 후, 정상적으로 서비스가 호출되는지 확인 & 캡쳐
![[스크린샷 2024-04-14 오전 11.23.41.png]]

---

**Lab 2. Blue/Green Deploy**

- **[1] ECS 프로덕션 서비스 생성**
	- curl 명령어 실행 후, 정상적으로 서비스가 호출되는지 확인 & 캡쳐 (웹으로도 확인 가능)
![[Pasted image 20240414164312.png]]
- **[2] 코드 배포 애플리케이션 생성**
	- [AWS Deploy]-[Applications]- hello-server → hello-server-prod 선택 후, 화면 캡쳐
![[Pasted image 20240414164325.png]]
- **[3] 파이프라인 확장**
	- [AWS Pipelines]-[Pipeline]-[Pipeline]-hello-server-pipeline 선택 후, 파이프라인 화면 캡쳐
![[Pasted image 20240414164356.png]]
- **[4] 변경 사항 푸시 및 테스트**
	- curl 명령어 실행 후, 정상적으로 서비스가 호출되는지 확인 & 캡쳐 (웹으로도 확인 가능)
![[Pasted image 20240414164407.png]]
---

**Lab 3. Exploring CodeDeploy**

- **[1-2] 테스트 포트 설정 & 푸시 변경, 테스트 및 롤백**
	- curl 명령어 실행 후, 정상적으로 서비스가 호출되는지 확인 & 캡쳐
		- TEST 포트(8080)에 접속하는 결과와 기존 포트(80)에 접속하는 결과 차이 비교  
![[Pasted image 20240414164443.png]]
		- [Reroute traffic] 클릭 후, 결과 확인  
![[Pasted image 20240414164447.png]]
		- [Stop and roll back deployment] 클릭 후, 결과 확인  
![[Pasted image 20240414164455.png]]
