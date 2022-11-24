```java
// 쓰레드풀 빈 등록
@Configuration
@EnableAsync
public class SpringAsyncConfig {
    @Bean(name = "asyncTaskPool")  // @Async 에서 사용할 목적
    public Executor asyncTaskPool() {
        ThreadPoolTaskExecutor taskExecutor = new ThreadPoolTaskExecutor();
        taskExecutor.setCorePoolSize(30);   // 기본 쓰레드 사이즈
        taskExecutor.setMaxPoolSize(100);    // 최대 쓰레드 사이즈
        taskExecutor.setQueueCapacity(50);   // Max 쓰레드가 동작하는 경우 대기하는 queue 사이즈
        taskExecutor.setThreadNamePrefix("asyncTaskPool-");
        taskExecutor.initialize();
        return taskExecutor;
    }

    @Bean(name = "apiTaskPool")  // api task 비동기 처리에 사용할 목적
    public ThreadPoolTaskExecutor searchTaskPool() {
        ThreadPoolTaskExecutor taskExecutor = new ThreadPoolTaskExecutor();
        taskExecutor.setCorePoolSize(30);   // 기본 쓰레드 사이즈
        taskExecutor.setMaxPoolSize(100);    // 최대 쓰레드 사이즈
        taskExecutor.setQueueCapacity(50);   // Max 쓰레드가 동작하는 경우 대기하는 queue 사이즈
        taskExecutor.setThreadNamePrefix("apiTaskPool-");
        taskExecutor.initialize();
        return taskExecutor;
    }
}

// 서비스 클래스 생성 예시
// 
// 여기서 롬복 RequiredArgsConstructor 를 사용해서 생성자에서 DI 를 받고 싶은데...
// asyncTaskPool 이 아니라 apiTaskPool 를 사용하고 싶어서... 
// Qualifier 사용해서 빈을 지정하고 싶다 !
// 
// 롬복은 생성자에서 어노테이션은 기본적을고 무시한다
// 그럼 롬복이 적용된 네이티브 소스를 보자 (인텔리가 해주는 디컴파일 소스를 보고 유추해보자)
// 
@Slf4j
@Service
@RequiredArgsConstructor
public class ProductComplexService {
    private final ProductClientService productClientService;
    private final OrderClientService orderClientService;
    @Qualifier("apiTaskPool")
    private final ThreadPoolTaskExecutor worker;
}
```

```shell
# lombok.config 파일, 프로젝트 루트에 생성
# 어노테이션 Qualifier 를 복사하겠다는 옵션 추가
lombok.copyableAnnotations += org.springframework.beans.factory.annotation.Qualifier
```

```java
// 디컴파일 소스코드
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by FernFlower decompiler)
//

// imports 생략...

@Service
public class ProductComplexService {
    private static final Logger log = LoggerFactory.getLogger(ProductComplexService.class);
    private final ProductClientService productClientService;
    private final OrderClientService orderClientService;
    @Qualifier("apiTaskPool")
    private final ThreadPoolTaskExecutor worker;

    public ProductSearchApiAsyncService(
            final ProductClientService productClientService, 
            final OrderClientService orderClientService, 
            final ThreadPoolTaskExecutor worker
    ) {
        this.productClientService = productClientService;
        this.orderClientService = orderClientService;
        this.worker = worker;
    }
}
```
