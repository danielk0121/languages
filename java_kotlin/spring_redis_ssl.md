- redis 서버가 ssl 연결을 사용하는 경우 커넥션 풀 설정
```java
package com.ylw.newevertimeserver.config;

import lombok.RequiredArgsConstructor;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.connection.RedisStandaloneConfiguration;
import org.springframework.data.redis.connection.lettuce.LettuceClientConfiguration;
import org.springframework.data.redis.connection.lettuce.LettuceConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.StringRedisSerializer;

@Configuration
@RequiredArgsConstructor
public class RedisConfig {

    @Value("${spring.redis.host}")
    private String redisHost;

    @Value("${spring.redis.port}")
    private int redisPort;

    /**
     * # 레디스 서버가 ssl 을 사용하는 경우 redis-cli 에서 tls 옵션을 사용해줘야 한다
     * # 참고, redis-cli 6 몇버전 부터 기본적으로 tls 옵션이 있지만, 이전 버전 redis-cli 는 tls 옵션이 없다
     * redis-cli -h 127.0.0.1 -p 56379 --tls
     */
    @Bean
    public RedisConnectionFactory redisConnectionFactory() {
        // 레디스 싱글모드 (센티널) 인 경우
        RedisStandaloneConfiguration redisConfiguration = new RedisStandaloneConfiguration(redisHost, redisPort);
        LettuceClientConfiguration lettuceClientConfiguration = LettuceClientConfiguration.builder()
                .useSsl()  // ssl 을 사용하는 redis 의 경우
//                .startTls()
                .disablePeerVerification()
                .build();
        
        // 레디스 설정이랑 레튜스 설정이랑 둘 다 넣어줘야 한다
        LettuceConnectionFactory connectionFactory = new LettuceConnectionFactory(redisConfiguration, lettuceClientConfiguration);
        return connectionFactory;
    }

    @Bean
    public RedisTemplate<String, Object> redisTemplate() {
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(redisConnectionFactory());
        
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        redisTemplate.setValueSerializer(new StringRedisSerializer());

        return redisTemplate;
    }
}

```