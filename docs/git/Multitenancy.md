---
layout: default
title: Multitenancy
parent: git
---

설정 코드 : https://github.com/mildw/doraemon/blob/main/core/src/main/java/com/example/core/multitenancy/TenantConnectionProvider.java

설정 코드 : https://github.com/mildw/doraemon/blob/main/multitenancy/src/main/java/com/example/multitenancy/config/database/TenantDataSourceConfig.java

MultiTenantConnectionProvider를 활용하여 기업에 따라 DB 스키마를 변경합니다.
트랜잭션이 생성되기 전인 필터 또는 인터셉터 레이어에서 TenantContextHolder를 통해 연결되는 DB스키마를 변경합니다.

```java
public class TenantConnectionProvider implements MultiTenantConnectionProvider {

    private final DataSource dataSource;

    public TenantConnectionProvider(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    @Override
    public Connection getAnyConnection() throws SQLException {
        return dataSource.getConnection();
    }

    @Override
    public void releaseAnyConnection(Connection connection) throws SQLException {
        connection.close();
    }

    @Override
    public Connection getConnection(String tenantIdentifier) throws SQLException {
        Connection connection = dataSource.getConnection();
        connection.createStatement()
                .execute(String.format("USE `%s`;", tenantIdentifier));

        return connection;
    }
}
```

```java
public class TenantIdentifierResolver implements CurrentTenantIdentifierResolver {

    @Override
    public String resolveCurrentTenantIdentifier() {
        try {
            return TenantContextHolder.getDatabaseName();
        } catch (Exception e) {
            return "common";
        }
    }

    @Override
    public boolean validateExistingCurrentSessions() {
        return false;
    }
}
```

```java
public class TenantContextHolder {

    public static final String TENANT_CONTEXT_KEY = "TenantContext";
    private static final ThreadLocal<TenantContext> tenantContextThreadLocal = new NamedThreadLocal<>(TENANT_CONTEXT_KEY);

    private TenantContextHolder() {
        throw new RuntimeException();
    }

    public static void setTenantContext(TenantContext tenantContext) {
        tenantContextThreadLocal.set(tenantContext);
    }

    public static TenantContext getTenantContext() {
        TenantContext tenantContext = tenantContextThreadLocal.get();

        if (Objects.isNull(tenantContext)) {
            throw new RuntimeException();
        }

        return tenantContext;
    }

    public static void resetTenantContext() {
        tenantContextThreadLocal.remove();
    }

    public static Long getTenantId() {
        return getTenantContext().getId();
    }

    public static String getDatabaseName() {
        return getTenantContext().getDatabaseName();
    }
}
```
