---
layout: default
title: Mybatis
parent: git
---

설정 코드 : https://github.com/mildw/doraemon/blob/main/mybatis/src/main/java/com/example/mybatis/config/database/mybatis/MybatisConfig.java

mysql, auto increment, JPA를 사용할때 bulk insert에 어려움이 있습니다.
때문에 mybatis를 같이 사용하여 대량의 insert를 실행할 수 있습니다.
jdbcTemplate라는 선택지도 있습니다.

```java
@Mapper
public interface OrderMapper {
    @InsertProvider(type = OrderSqlProvider.class, method = "insertOrders")
    void insertOrders(@Param("orders") List<Order> orders);
}
```

```java
public class OrderSqlProvider {
    public String insertOrders(Map<String, Object> params) {
        @SuppressWarnings("unchecked")
        List<Order> orders = (List<Order>) params.get("orders");
        StringBuilder sql = new StringBuilder();
        sql.append("INSERT INTO orders (user_id, product, quantity) VALUES ");
        for (int i = 0; i < orders.size(); i++) {
            Order order = orders.get(i);
            sql.append("(")
               .append(order.getUserId()).append(", ")
               .append("'").append(order.getProduct()).append("', ")
               .append(order.getQuantity()).append(")");
            if (i < orders.size() - 1) {
                sql.append(",");
            }
        }
        return sql.toString();
    }
}
```
