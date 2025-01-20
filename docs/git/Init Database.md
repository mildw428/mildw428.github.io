---
layout: default
title: Init Database
parent: git
---

설정 코드 : https://github.com/mildw/doraemon/blob/main/init-table-liquibase/src/main/java/com/example/inittableliquibase/application/database/DatabaseService.java


liquibase 사용

스키마 멀티테넌시를 사용하여 하나의 rds에 여러 기업의 스키마 존재

개발시 모든 테넌트의 동기화 작업에 불편함을 느낌

기존에는 단순 루프문을 통해 각 테넌트별 쿼리를 수행했다면

리큐베이스를 통해 모든 테넌트에 쿼리를 적용시키고 작동 여부를 각 테넌트에 저장함

리큐베이스는 로컬에 있는 sql파일을 읽어 수행하므로, 배포되어있는 각 환경에 적용하려면 리큐베이스를 작동시키는 프로젝트에 sql문을 추가하고 배포까지 해야함

-> s3에 sql파일을 저장하여 쿼리 추가시 s3로부터 파일을 다운받아 추가 작성하고 다시 S3에 저장

기업에서 계약시 테이블을 초기화해주는데에도 해당 기능으로 테이블을 초기화할 수 있도록 변경


각 환경별로 적용되어있는 테이블 형상이 다르다. 이 때 리큐베이스를 도입하기전에 고려해야할것은?

리큐베이스는 버저닝을 통해 해당 쿼리를 수행했는지를 체크한다.

테이블 형상은
pr, st 환경이 같다
dv, qa 환경이 같다 (같게 만들 수 있다)

pr, st버전이 더 낮기 때문에 해당 환경 기준으로 동기화 버전이 시작되어야한다

-> pr기준의 테이블 형상으로 dv환경에 생성한 후 리큐베이스를 실행시켜 동기화 로그를 생성한다. 각 테넌트의 dv, qa환경 db에 해당 로그를 추가하여, 일치시킨다

변경 쿼리를 따로 관리해서 간단히 할 수 있을 것 같다.

따로 관리하지 않았더라고 해도, 인텔리제이의 db compare기능이 있으니 문제 없을 듯 하다


코드

```groovy
// Liquibase dependency  
implementation 'org.liquibase:liquibase-core:4.6.2'
```

```java
Database database = null;  
Connection connection = null;

connection = DriverManager.getConnection("jdbc:mariadb://localhost:3306/" + schema, "root", "1111");  
database = DatabaseFactory.getInstance()  
        .findCorrectDatabaseImplementation(new liquibase.database.jvm.JdbcConnection(connection));  
  
String changeLogPath = String.format("database/tenant/table_init.sql");  
Liquibase liquibase = new Liquibase(changeLogPath, new ClassLoaderResourceAccessor(), database);  
liquibase.update(new Contexts());

//try - catch - finally
database.close();
connection.close();

```

```java
  
public Boolean initSchemas() {  
    Bucket bucket = new Bucket("bucketName","accessKey","secretKey");  

	//sql 파일 다운로드
    S3Utils.downloadToLocal(bucket,s3ChangeLogPath+tableInitFileName);  
    S3Utils.downloadToLocal(bucket,s3ChangeLogPath+tableUpdateFileName);  
    boolean result = initTables("test");  

	//sql 파일 삭제
    deleteFile(changeLogPath+tableInitFileName);  
    deleteFile(changeLogPath+tableUpdateFileName);  
    return result;  
}  
  
public Boolean updateTables(String author, String query) {  
    Bucket bucket = new Bucket("bucketName","accessKey","secretKey");  

	//Sql 파일 다운로드
    S3Utils.downloadToLocal(bucket,s3ChangeLogPath+tableUpdateFileName);  

	//Pattern pattern = Pattern.compile("-- changeset\\s+\\w+:(\\d+)");
	//마지막 id값 추출
    int lastId = getLastChangeSetId(changeLogPath+tableUpdateFileName);  

	//file에 query 추가
    updateSqlFile(author, query, lastId);  
  
    List<String> schemas = List.of("test");  
    boolean result = true;  
    for (String schema : schemas) {  
        if(!updateTable(schema)) {  
            return false;  
        }  
    }  
    if (result) {  
		// 쿼리가 잘 실행 됐다면 S3에 파일 업로드
        S3Utils.uploadFromLocal(bucket, s3ChangeLogPath + tableUpdateFileName);  
    }  
  
    return result;
}
```

