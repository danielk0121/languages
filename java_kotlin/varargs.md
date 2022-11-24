참고 : https://zion830.tistory.com/124

가변인자 사용
* 연산자 = 펼치기연산자 = spread operator

```kotlin
// 호출하는 부분, 펼치기 연산자를 사용
val httpHosts = arrayOf(HttpHost(hostname, port, schema))
val restClientBuilder: RestClientBuilder = RestClient.builder(*httpHosts)

// 함수 구현부, 가변인자를 사용한다
public static RestClientBuilder builder(HttpHost... hosts) {
    ...
}
```
