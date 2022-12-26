- 로그가 출력되는 stacktrace 가 아니라 로그 메세지에서 method 이름과 라인번호를 출력하고 싶었는데 ? 가 출력되는 문제가 발생했다

- line number, method 정보 등은 처리 속도가 느릴수 있다.
- 왜냐면, stacktrace 에서 정보를 파싱하기 때문....
- 그래서, stacktrace 이기 때문에 AsyncAppneder 는 기본값으로 callder data 를 가져오지 않는다
- 밑에 라인번호를 파싱하는 logback 디버깅 코드를 보면...
```java
package ch.qos.logback.classic.pattern;

import ch.qos.logback.classic.spi.CallerData;
import ch.qos.logback.classic.spi.ILoggingEvent;

public class LineOfCallerConverter extends ClassicConverter {

    public String convert(ILoggingEvent le) {
        StackTraceElement[] cda = le.getCallerData();
        if (cda != null && cda.length > 0) {
            return Integer.toString(cda[0].getLineNumber());
        } else {
            return CallerData.NA;  // cda 배열값이 없으니까, 여기에 걸리는 것 ! NA = ? 문자열이다
        }
    }
}
```

- 코드 디버깅을 하던중... 스택형 글에서 겨우 힌트를 얻었다
- You need to set AsyncAppender's includeCallerData property to true. Here is the modified config file:
```xml
<configuration>
    <appender name="FILE" class="ch.qos.logback.core.FileAppender">
      <file>myapp.log</file>
      <encoder><pattern>%logger{35} - [%F:%L] - %msg%n</pattern></encoder>
    </appender>

    <appender name="ASYNC" class="ch.qos.logback.classic.AsyncAppender">
      <appender-ref ref="FILE" />
      <!-- add the following line -->
      <includeCallerData>true</includeCallerData>
    </appender>

    <root level="DEBUG"><appender-ref ref="ASYNC" /></root>
 </configuration>
```