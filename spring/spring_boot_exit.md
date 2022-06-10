- 스프링 부트에서 어플리케이션을 강제 종료 하고 싶은 경우 
- System.exit() 를 사용하면 무시가 된다
- 스프링에서 exit 시 처리하는 빈이 없어서 무시되는거 같음

```
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.SpringApplication;
import org.springframework.context.ApplicationContext;
import org.springframework.http.ResponseEntity;
import org.springframework.util.StringUtils;
import org.springframework.web.bind.annotation.*;

import javax.annotation.PostConstruct;

@Slf4j
@RequiredArgsConstructor
@RequestMapping("/direct-order")
@RestController
public class SomeController {

    private final ApplicationContext applicationContext;
    @PostConstruct
    public void foo() {
        // some logic

        SpringApplication.exit(applicationContext, () -> 0);
        System.exit(0);
    }
}
```