---
layout: default
title: ChainedTransaction
parent: git
---

설정 코드 : https://github.com/mildw/doraemon/blob/main/multitenancy/src/main/java/com/example/multitenancy/config/database/TenantDataSourceConfig.java

동시에 두가지 DB에 연결하여 트랜잭션을 관리합니다.

저희 프로젝트해서는 문항과 선택지를 관리하는 common DB와
각 기업의 데이터를 관리하는 tenant DB가 있습니다.
ChainedTransactionManager는 두개의 트랜잭션잉 연결해주는 역할을 합니다.

```java
@Configuration
public class TransactionConfig {

    @Primary
    @Bean
    public ChainedTransactionManager transactionManager(
            @Qualifier("commonTransactionManager") PlatformTransactionManager commonTransactionManager,
            @Qualifier("tenantTransactionManager") PlatformTransactionManager tenantTransactionManager) {

        return new ChainedTransactionManager(commonTransactionManager, tenantTransactionManager);

    }

}
```
