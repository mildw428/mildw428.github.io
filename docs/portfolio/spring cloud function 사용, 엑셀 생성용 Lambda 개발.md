---
layout: default
title: spring cloud function 사용, 엑셀 생성용 Lambda 개발
parent: portfolio
---
# spring cloud function 사용, 엑셀 생성용 Lambda 개발
---

[[Spring Cloud Function]]
[[RDS Proxy & Lambda VPC & Secret Manager]]

- **상황**
	- 진단의 응시 결과 PDF 또는 Excel파일을 생성하는 파일함 서버 존재
	- 타 유닛에서 pdf생성 람다개발 이후 기존의 파일함 서버는 엑셀 파일 생성만을 위해 존재
	- 엑셀 파일은 상시 사용되지 않음
	- 파일함 서버는 항상 구동 중이지만 엑셀 파일 생성 요청은 드문 편이어서 비효율적인 서버 자원 사용 문제가 발생함.
- **과제**
	- 엑셀 파일 생성 기능의 비효율성을 해결하고, 서버 자원을 절감할 수 있는 방법 모색
	- 기존의 엑셀 생성 로직에 Java와 QueryDsl을 사용한 코드가 있어 이를 최대한 활용해야함
- **해결**
	- Excel 파일도 생성 요청 시점에만 자원을 사용할 수 있도록 Lambda로 전환
	- **Spring Cloud Function**을 사용하여 서버리스 환경에서 기존 Spring Boot 구조를 유지하며 Excel 파일 생성 기능을 서버리스로 전환
		- **문제 1: Cold Start 지연**
		    - Spring Cloud Function을 사용해 Spring Boot 기반 코드를 Lambda로 전환할 때 Cold Start로 인해 응답 시간이 길어짐
		- **해결**
		    - Cold Start 시간을 줄이기 위해 Spring Cloud Function 설정을 경량화하고, 불필요한 빈(bean)들을 제거하여 성능 최적화
		- **문제 2: DB 커넥션 과부하**
		    - 여러 Lambda 인스턴스가 동시에 실행되면 데이터베이스 연결 수가 과도하게 증가할 가능성 발생
		- **해결**
		    - **RdsProxy**를 통해 데이터베이스 연결을 관리하고, 다수의 Lambda 요청 시에도 안정적으로 DB 연결을 처리할 수 있도록 설정
- **결과**
	- 파일 생성 요청이 적은 기간에도 서버를 유지할 필요가 없어, 효율적인 리소스 관리가 가능해짐
