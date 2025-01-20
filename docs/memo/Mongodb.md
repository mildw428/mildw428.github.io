---
layout: default
title: Mongodb
parent: memo
---
# Mongodb
---

## 정의?

## 활용사례?

## 장점?

## 단점?

https://docs.spring.io/spring-data/mongodb/docs/current/reference/html/

https://www.mongodb.com/compatibility/spring-boot

https://ckddn9496.tistory.com/102

## 예제 코드

```java
//
@Configuration  
@RequiredArgsConstructor  
@EnableMongoRepositories(basePackages = "com.jainwon.cec.examinee.domain.cec")  
public class MongoConfig {  
  
  private final MongoProperty mongoProperty;  
  private static final String DB_NAME = "cec";  
  private static final String DEFAULT_AUTH_DB_NAME = "admin";  
  
  @Bean  
  public MongoDatabaseFactory mongoDatabaseFactory() {  
  
    ConnectionString connectionString = new ConnectionString(mongoProperty.getUrl());  
  
    String authDatabase = connectionString.getDatabase() != null ? connectionString.getDatabase() : DEFAULT_AUTH_DB_NAME;  
  
    MongoClientSettings settings = MongoClientSettings.builder()  
            .applyConnectionString(connectionString)  
            .credential(MongoCredential.createCredential(mongoProperty.getUsername()  
                    , authDatabase, mongoProperty.getPassword().toCharArray()))  
            .build();  
  
    MongoDriverInformation driverInformation = MongoDriverInformation.builder().build();  
  
    MongoClient mongoClient = new MongoClientImpl(settings, driverInformation);  
    return new SimpleMongoClientDatabaseFactory(mongoClient, DB_NAME);  
  }  
  
  
  @Bean  
  public MongoTemplate mongoTemplate() {  
  
    MongoTemplate mongoTemplate = new MongoTemplate(mongoDatabaseFactory());  
  
    MappingMongoConverter converter = (MappingMongoConverter) mongoTemplate.getConverter();  
    converter.setTypeMapper(new DefaultMongoTypeMapper(null));  
  
    return mongoTemplate;  
  
  }  
  
}
```

```java
@ToString  
@Document(collection = "user_answer")  
@NoArgsConstructor(access = AccessLevel.PROTECTED)  
public class UserAnswer {  
  
    @Id  
    public String id;  
    public Integer questionId;  
    public Integer answer;  
  
    public UserAnswer(Integer questionId, Integer answer) {  
        this.questionId = questionId;  
        this.answer = answer;  
    }  
  
}
```

