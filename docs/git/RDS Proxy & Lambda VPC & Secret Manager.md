---
layout: default
title: RDS Proxy & Lambda VPC & Secret Manager
parent: git
---

Lambda 함수가 동시 요청을 처리할 때 RDS Proxy를 통해 데이터베이스 연결 풀링이 이루어지므로 데이터베이스에서 발생하는 부하를 분산시키고 성능을 향상시킬 수 있습니다.

RDS Proxy를 사용하려면 RDS DB 인스턴스와 RDSProxy 간에 공통의 Virtual Private Cloud(VPC)가 있어야 합니다.

아래는 AWS 설정 과정입니다.

![[Pasted image 20240406215153.png]]
![[Pasted image 20240406215400.png]]

![[Pasted image 20240406175109.png]]
Lambda 생성 당시 고급 설정을 통해 VPC를 설정할 수 있으며, 생성 이후에도 가능합니다.
![[Pasted image 20240406221410.png]]

![[Pasted image 20240406221429.png]]
VPC설정을 완료하고 나면 Time out이 뜨지 않는 것을 확인할 수 있습니다.

---
## Secrets Manager

환경 변수에 직접 데이터베이스의 ID와 암호를 저장하는 것 대신에 AWS Secrets Manager를 사용을 권장 하고있습니다.

Lambda는 secrets manager 에 접근하기 위해
```
"secretsmanager:GetSecretValue",
"secretsmanager:DescribeSecret"
```
두가지 권한이 필요합니다.

```java
public class AwsSecrets {  
    private String dbInstanceIdentifier;  
    private String engine;  
    private String resourceId;  
    private String username;  
    private String password;  
}
```

```java
@Bean  
public HikariConfig hikariConfig() {  
    if(secretName != null && !secretName.isBlank()) {  
        AwsSecrets awsSecrets = getSecretValue(secretName);  
        dbUsername = awsSecrets.getUsername();  
        dbPassword = awsSecrets.getPassword();  
    }  
  
    HikariDataSource dataSource = new HikariDataSource();  
    dataSource.setJdbcUrl(dbUrl);  
    dataSource.setUsername(dbUsername);  
    dataSource.setPassword(dbPassword);  
    dataSource.setMaximumPoolSize(1);  
    return dataSource;  
}  
  
private AwsSecrets getSecretValue(String secretName) {  
    AWSSecretsManager client = AWSSecretsManagerClientBuilder.standard().build();  
    GetSecretValueRequest request = new GetSecretValueRequest().withSecretId(secretName);  
    GetSecretValueResult result = client.getSecretValue(request);  
    String secretString = result.getSecretString();  
    if(secretString == null) {  
        throw new RuntimeException();  
    }  
  
    return ObjectMapperUtils.readValue(secretString, AwsSecrets.class);  
}
```
