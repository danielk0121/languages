- spring boot yaml 설정파일에 대해 property 클래스로 맵핑하는 방법
- spring boot kotlin 기준으로 설명

```kotlin
import org.jetbrains.kotlin.gradle.tasks.KotlinCompile

plugins {
    kotlin("jvm") version "1.4.32"
    kotlin("plugin.spring") version "1.4.32"
    id("org.springframework.boot") version "2.4.5"
    id("io.spring.dependency-management") version "1.0.11.RELEASE"
}

version = "0.0.1"
java.sourceCompatibility = JavaVersion.VERSION_11
java.targetCompatibility = JavaVersion.VERSION_11

...

dependencies {
    annotationProcessor("org.springframework.boot:spring-boot-configuration-processor")
}
```

```kotlin
// main 클래스나 다른곳에 선언되어 있어야 한다
@ConfigurationPropertiesScan
```

```yaml
app:
  db:
    url: "jdbc:mysql://127.0.0.1:3303"
    username: asdf
    password: qwer
  es:
    host: asdf
    apiKey: qwer
  redis:
    hostname: "127.0.0.1"
    port: 6379
```

```kotlin
import org.slf4j.LoggerFactory
import org.springframework.boot.context.properties.ConfigurationProperties
import org.springframework.boot.context.properties.ConstructorBinding
import javax.annotation.PostConstruct

@ConstructorBinding
@ConfigurationProperties(prefix = "app")
data class AppYamlProperty(
    var db: AppDbProperty?,
    var es: AppEsProperty?,
    var redis: AppRedisProperty?,
) {
    data class AppDbProperty(
        var url: String?,
        var username: String?,
        var password: String?,
    )
    data class AppEsProperty(
        var host: String?,
        var apiKey: String?,  // 대쉬를 카멜케이스 프로퍼티로 자동 맵핑
    )
    data class AppRedisProperty(
        var hostname: String?,
        var port: Int?,
    )

    private val log = LoggerFactory.getLogger(AppYamlProperty::class.java)
    @PostConstruct
    fun init() {
        log.info("${AppYamlProperty::class.java.simpleName} init:\n$this")
    }
}
```