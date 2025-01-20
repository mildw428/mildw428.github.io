---
layout: default
title: Serverless
parent: 실습
grand_parent: aws
---
# Serverless
---

https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop


## 1. 데이터로 시작

### 1.1. DynamoDB 만들고 결과 캡쳐
![[Pasted image 20240420172733.png]]
### 1.2. Lambda 함수를 수행하고 결과 캡쳐
![[Pasted image 20240420172727.png]]
## 2. DynamoDB에 연결

### 2.1. DynamoDB 연결하고 Lambda 결과 화면 캡쳐
![[Pasted image 20240420172843.png]]
### 2.2. Lambda 수행하고 DynamoDB 결과 화면 캡쳐
![[스크린샷 2024-04-19 오후 10.29.20.png]]
## 3. API 구축

### 3.1. API GW를 구축하고 api 호출 결과 화면 캡쳐
![[Pasted image 20240420172903.png]]

# M1. 동기 호출

## 1. 데이터 저장소 만들기

### 1.1. DynamoDB 만들고 화면 캡쳐
![[Pasted image 20240420173017.png]]
## 2. 비즈니스 로직 추가

### 2.1. Lambda 만들고 화면 캡쳐
![[Pasted image 20240420173051.png]]


## 3. API 연결

### 3.1. API GW 만들고 화면 캡쳐
![[Pasted image 20240420173107.png]]

### 3.2. API GW 만들고 API 호출 화면 캡쳐
![[Pasted image 20240420173122.png]]

## 3-1. 사용자 풀 생성

### 3-1.1. Amazon Cognito 를 만들고 화면 캡쳐
![[Pasted image 20240420173128.png]]
## 3-2. API 보안

### 3-2.1. 보안에 필요한 Lambda를 만들고 화면 캡쳐
![[Pasted image 20240420173135.png]]
## 3-3. 승인 확인

### 3-3.1. API GW 요청을 보낼 때 승인된 API 요청 화면을 캡쳐
![[Pasted image 20240420173222.png]]
