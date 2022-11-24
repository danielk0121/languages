## yaml 파일
```
tcp:
  server:
    host: localhost
    port: 10200
    charset: UTF-8
  client:
    confirm:
      host: localhost
      portOdd: 4517
      portEven: 14517
      charset: UTF-8
    other:
      host: localhost
      portOdd: 4518
      portEven: 14518
      charset: UTF-8
```

## java properties 파일
```
import lombok.Getter;
import lombok.Setter;
import lombok.ToString;
import lombok.extern.slf4j.Slf4j;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

import javax.annotation.PostConstruct;
import javax.validation.constraints.Min;
import javax.validation.constraints.NotNull;
import javax.validation.constraints.Size;

@Slf4j
@Getter
@Setter
@ToString
@Component
@ConfigurationProperties(prefix = "tcp")
public class TcpProperties {

    private ServerInfo server;  // 기본적으로 변수명을 찾아감, 클래스명 무관
    private ClientInfo client;

    @Getter @Setter @ToString
    public static class ServerInfo {  // 여러개 파일로 구분하면 쓸데없이 파일이 많아지므로 내부 클래스 사용
        @NotNull @Size(min=1000, max=65535)  private int port = 10200;
        @NotNull @Min(1) private int bossCount = 1;
        @NotNull @Min(2) private int workerCount = 2;
        @NotNull private int backlog = 128;
        private boolean isKeepAlive = true;
        private int soTimeout = 120000;
    }

    @Getter @Setter @ToString
    public static class ClientInfo {
        private ConnectionInfo confirm;
        private ConnectionInfo other;
    }

    @Getter @Setter @ToString
    public static class ConnectionInfo {
        private String host;
        private int portOdd;
        private int portEven;
        private String charset;
    }

    @PostConstruct
    public void init() {
        log.info("loaded: {}", this.toString());
    }
}
```
