SearchProperties
```kotlin
package xxx.service

import org.slf4j.LoggerFactory
import org.springframework.boot.context.properties.ConfigurationProperties
import org.springframework.boot.context.properties.ConstructorBinding
import javax.annotation.PostConstruct

// @ConstructorBinding 를 사용할경우, @Bean 이나 @Component 등을 사용해서 직접적으로 빈 설정하면 에러발생함
//@org.springframework.stereotype.Component

@ConstructorBinding // <- yaml 값 맵핑을 위해 생성자를 사용함, 이게 없으면 setter 가 필요함, 불변객체를 위해 사용
@ConfigurationProperties(prefix = "xxx.search")
data class SearchProperties (
    val productIndexEnv: String,
    val elastic: ElasticProperties,
) {
    data class ElasticProperties(
        val host: String,
        val apiKey: String,
        val schema: String,
        val port: Int,
    )

    private val log = LoggerFactory.getLogger(javaClass)

    @PostConstruct
    fun init() {
        log.info("${javaClass.simpleName} loaded, properties: $this")
    }
}
```

ConfigurationPropertiesConfig
```kotlin
package xxx.config

import xxx.SearchProperties

import org.springframework.boot.context.properties.EnableConfigurationProperties
import org.springframework.context.annotation.Configuration

// @ConfigureProperties 와 관련 없음
//@org.springframework.boot.autoconfigure.EnableAutoConfiguration

// @EnableConfigurationProperties 에서 명시하지 않으면 필요함
// @EnableConfigurationProperties -> 하나씩 명시적으로 설정
// @ConfigurationPropertiesScan -> 빈 스캔
//@org.springframework.boot.context.properties.ConfigurationPropertiesScan

// 추천 설정 방법
@Configuration // <- 엄청 빠른 우선순위 빈 로딩을 실행함
@EnableConfigurationProperties(SearchProperties::class) // <- 배열로 여러개 등록 가능
class ConfigurationPropertiesConfig
```

SearchPropertiesTest
```kotlin
package xxx.service

import xxx.ConfigurationPropertiesConfig

import org.junit.jupiter.api.Assertions.*
import org.junit.jupiter.api.Test
import org.junit.jupiter.api.extension.ExtendWith
import org.springframework.beans.factory.annotation.Autowired
import org.springframework.boot.context.properties.EnableConfigurationProperties
import org.springframework.boot.test.context.SpringBootTest
import org.springframework.test.context.ActiveProfiles
import org.springframework.test.context.TestConstructor
import org.springframework.test.context.TestPropertySource
import org.springframework.test.context.junit.jupiter.SpringExtension

// case1) springboottest 어노테이션 사용
// boot application 이 시작되는 클래스파일을 기준으로 boot 어플리케이션 컨텍스트 전체가 로딩된다
//
// ex)
//@SpringBootTest

// case2) springboottest 어노테이션에 클래스 명시
// 
// ConfigurationPropertiesConfig 에서
// @EnableConfigurationProperties(SearchProperties::class) 때문에
// SearchProperties 클래스의 빈이 한번 생성되고
//
// @SpringBootTest classes 에 선언된
// SearchProperties::class 이거 때문에 클래스 빈이 한번더 로딩된다
//
// 따라서 이름이 같은 빈이 2개 로딩되어서 에러가 발생함
//
// ex)
//@SpringBootTest(classes = [SearchProperties::class, ConfigurationPropertiesConfig::class])

// case3) springboottest 어노테이션에 클래스를 configuration 클래스만 명시
// case2 의 내용에 의해서 SearchProperties 빈이 한번만 로딩된다
//
// ex)
@SpringBootTest(classes = [ConfigurationPropertiesConfig::class])

// @ExtendWith junit5 프레임워크에서 spring 을 사용하겠다는 어노테이션
// 인텔리제이에서 실행할때는 없어도 되는것 같은데...
// 
// ex)
//@ExtendWith(SpringExtension::class)

// @ConstructorBinding 때문에 사용하는것 같은데, 없어도 동작함
// ex)
//@TestConstructor(autowireMode = TestConstructor.AutowireMode.ALL)

@TestPropertySource("classpath:application.yml")
@ActiveProfiles(*["local-jakarta","local-log"])

// kotlin+spring+boot+test 환경에서 주입받으려면 @Autowired constructor 로 주입받아야 함
//
internal class SearchPropertiesTest @Autowired constructor (
    val searchProperties: SearchProperties,
) {

// org.junit.jupiter.api.extension.ParameterResolutionException: No ParameterResolver registered for parameter
// junit 테스트는 스프링프레임워크가 주가 아니라 junit 프레임워크가 스프링 프레임워크를 실행하는 구조임
// 따라서 junit 프레임워크는 적절한 ParameterResolver 가 선언되지 않았다고 판단하고 에러를 발생시킴
// 
// org.junit.jupiter.api.extension.ParameterResolutionException:
//   No ParameterResolver registered for parameter [xxx.SearchProperties searchProperties] in constructor
// ex)
//internal class SearchPropertiesTest (
//      val searchProperties: SearchProperties,
//) {

    // 레이지 로딩으로 빈 주입이 되지 않음 !
//    @Autowired
//    private lateinit var searchProperties: SearchProperties

    @Test
    fun `search property load test`() {
//        assertEquals("local", searchProperties.productIndexEnv)
//        assertEquals("https", searchProperties.elastic!!.schema)
//        assertEquals(443, searchProperties.elastic!!.port)
//        assertEquals("search-dev.es.ap-northeast-2.aws.elastic-cloud.com", searchProperties.elastic!!.host)
//        assertEquals("xxxxxxxxxx", searchProperties.elastic!!.apiKey)
        assertNotNull(searchProperties)
    }
}
```