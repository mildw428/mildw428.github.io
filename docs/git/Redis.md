---
layout: default
title: Redis
parent: git
---

`github` : https://github.com/mildw/doraemon/tree/main/redis

## 레딧스를 어디에 사용하는가

운영 중인 웹 서버에서 키-값 형태의 데이터 타입을 처리해야 하고, I/O가 빈번히 발생해 다른 저장 방식을 사용하면 효율이 떨어지는 경우에 사용합니다.

조회수와 같은 카운트 형태의 데이터일 겁니다. 유튜브를 생각해보면 인기 채널의 신규 동영상을 1시간도 안돼서 100만 조회수를 넘기는 경우도 있습니다. 이때 조회수에 해당하는 데이터를 RDS 형태의 데이터에 저장해 I/O를 반복한다면 엄청난 자원이 사용될 것이란 건 불 보듯 뻔한 일입니다.

일정한 주기에 따라 RDS에 업데이트를 한다면 RDS에 가해지는 부담을 크게 줄이고, 성능은 크게 향상할 수 있습니다.

많은 I/O를 예시로 들었지만 가장 많이 사용되는 건 역시 사용자의 세션 관리입니다. 사용자의 세션을 유지하고, 불러오고, 여러 활동들을 추적하는 게 매우 효과적으로 사용할 수 있습니다. 또한 매우 빠르게 동작한다는 점을 사용해 메시지 큐잉에도 사용할 수 있습니다.

그 밖에도 강력하게 사용할 수 있는게 바로 API 캐싱입니다. 라우트로 들어온 요청에 대해 요청 값을 캐싱해면 동일 요청에 대해 캐싱된 데이터를 리턴하는 방식입니다. 이와 관련된 쉬운 링크가 있어 아래에 첨부합니다. 

같은 요청이 여러 번 들어오는 경우 매번 데이터 베이스를 거치는 것이 아니라 캐시 서버에서 첫 번째 요청 이후 저장된 결괏값을 바로 내려주기 때문에 DB의 부하를 줄이고 서비스의 속도도 느려지지 않는 장점이 있습니다.

## **Redis 사용에 주의할 점으로는**

-   서버에 장애가 발생했을 경우 그에 대한 운영 플랜이 꼭 필요합니다.  
    : 인메모리 데이터 저장소의 특성상, 서버에 장애가 발생했을 경우 데이터 유실이 발생할 수 있기 때문입니다.
-   메모리 관리가 중요합니다.
-   싱글 스레드의 특성상, 한 번에 하나의 명령만 처리할 수 있습니다. 처리하는데 시간이 오래 걸리는 요청, 명령은 피해야 합니다.

쿼리가 길고 복잡한 경우에도 데이터베이스를 조회하는 시간이 오래 걸리는데, 이 쿼리가 자주 사용되는 경우라면 해당 쿼리가 전체 서비스 속도의 병목이 될 수도 있겠죠?
그럴때는 쿼리 결과 자체를 캐싱을 해두고, 쿼리의 결과가 바뀔 수 있는 이벤트가 발생할 때마다 캐시에 적재를 새로한다면 전체 서비스 속도를 향상 시킬 수 있을것입니다.

## 설정 코드

https://www.baeldung.com/spring-boot-redis-cache


```groovy
//캐시
implementation 'org.springframework.boot:spring-boot-starter-data-redis'
//세션
implementation 'org.springframework.session:spring-session-data-redis'

```

```java
@Configuration
@EnableCaching
@RequiredArgsConstructor
public class RedisConfig {

    private final RedisProperty redisProperty;

    @Bean
    public RedisConnectionFactory redisConnectionFactory() {
        RedisStandaloneConfiguration redisStandaloneConfiguration =
                new RedisStandaloneConfiguration(redisProperty.getHost(), redisProperty.getPort());
        redisStandaloneConfiguration.setDatabase(redisProperty.getDatabase());
        redisStandaloneConfiguration.setPassword(redisProperty.getPassword());

        return new JedisConnectionFactory(redisStandaloneConfiguration);
    }

    @Bean
    public RedisOperations<String, Object> redisOperations() {
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();

        redisTemplate.setConnectionFactory(redisConnectionFactory());
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        redisTemplate.setHashKeySerializer(new StringRedisSerializer());
        redisTemplate.setValueSerializer(new GenericJackson2JsonRedisSerializer());

        return redisTemplate;
    }

    @Bean
    public CacheManager cacheManager(RedisConnectionFactory connectionFactory) {
        CacheManager cacheManager = RedisCacheManager.builder(connectionFactory)
                .cacheDefaults(defaultCacheConfiguration())
                .withInitialCacheConfigurations(cacheConfigurationMap())
                .build();

        return cacheManager;
    }

    private RedisCacheConfiguration defaultCacheConfiguration() {
        return RedisCacheConfiguration.defaultCacheConfig()
                .serializeKeysWith(fromSerializer(new StringRedisSerializer()))
                .serializeValuesWith(fromSerializer(new GenericJackson2JsonRedisSerializer()))
                .entryTtl(Duration.ofMinutes(20));
    }

    private Map<String, RedisCacheConfiguration> cacheConfigurationMap() {
        return new HashMap<>();
    }

}
```

`redisConnectionFactory()` : redis 서버 연결

`redisTemplate()` : redis 의 명령을 도와줄 템플릿
(캐시 저장에는 필요 없음 / 참고 : https://www.baeldung.com/spring-boot-redis-cache)

`cacheManager()` : 캐시 매니저를 통해 redis cache config 설정

`redisCacheConfiguration()` : default redis cache config 설정

`redisCacheManagerBuilderCustomizer()` : name space에 따른 만료 기간 처리

>`RedisTemplate` 를 이용해서 실제 레디스를 스프링에서 사용하는데 중요한 것은 `setKeySerializer()`, `setValueSerializer()` 메소드들이다. 이 메소드를 빠트리면 실제 스프링에서 조회할 때는 값이 정상으로 보이지만 `redis-cli`로 보면 key값에 `\xac\xed\x00\x05t\x00\x0` 이런 값들이 붙는다.




```java
//SSL 적용 경우
@Bean
    public RedisConnectionFactory redisConnectionFactory() {
        LettuceClientConfiguration lettuceClientConfiguration = LettuceClientConfiguration.builder().useSsl().build();

        RedisStandaloneConfiguration redisStandaloneConfiguration =
                new RedisStandaloneConfiguration(redisProperty.getHost(), redisProperty.getPort());
        redisStandaloneConfiguration.setDatabase(redisProperty.getDatabase());
        redisStandaloneConfiguration.setPassword(redisProperty.getPassword());

        return new LettuceConnectionFactory(redisStandaloneConfiguration, lettuceClientConfiguration);
}

@Bean  
public ConfigureRedisAction configureRedisAction() {  
	return ConfigureRedisAction.NO_OP;  
}
```

`configureRedisAction()` : `ConfigureRedisAction.NO_OP` 는 레디스 세션 만료시 destroy event가 진행되어야 하는데, 이 destroy event는 기본적으로 자동 설정이 되어있지만 보안의 redis 환경인 경우 동작하지 않는다고 한다.


```java
@Component  
@RequiredArgsConstructor  
public class RedisComponent {  
  
    private final StringRedisTemplate redisTemplate;  
  
    public void saveSession(Long memberSn, String sessionId) {  
  
        final int TIMEOUT = 60 * 6;  
  
        String sessionIdKey = getMemberSessionIdKey(memberSn,sessionId);  
        redisTemplate.opsForValue().set(sessionIdKey, sessionId);  
        redisTemplate.expire(sessionIdKey, TIMEOUT, TimeUnit.MINUTES);  
  
        String allowedSessionKey = getMemberAllowedSessionKey(memberSn);  
        redisTemplate.opsForValue().set(allowedSessionKey, sessionId);  
        redisTemplate.expire(allowedSessionKey, TIMEOUT, TimeUnit.MINUTES);  
  
    }
}
```

`opsForValue()` 에 대한 참고 : https://sabarada.tistory.com/105
