### util code
```
import java.util.function.Consumer;
import java.util.function.Supplier;

public class LambdaUtil {
public static <T> LambdaHelper<T> runAndGet(Supplier<T> task) {
LambdaHelper<T> helper = LambdaHelper.getInstance();
return helper.runAndGet(task);
}

    public static class LambdaHelper<T> {
        private T t;
        private LambdaHelper() {
        }
        private static <T> LambdaHelper<T> getInstance() {
            return new LambdaHelper<T>();
        }
        private LambdaHelper<T> runAndGet(Supplier<T> supplier) {
            LambdaHelper<T> instance = getInstance();
            this.t = supplier.get();
            return instance;
        }
        public T also(Consumer<T> consumer) {
            consumer.accept(t);
            return t;
        }
    }
}
```

### example
```
// 메소드 호출 결과를 로그로 남김
var resultDataList = runAndGet(() -> someLogic(inputData))
    .also(it -> log.info("" + it));

// 메소드 파라미터로 넘기기 전에 추가 작업(로그) 등을 실행
resultList.addAll(
        runAndGet(() -> inputDataList.stream()
                .filter(Objects::nonNull)
                .map(it -> ResultModel.builder()
                        .productId(it.getProductId())
                        .message("not-exists")
                        .build())
                .collect(toList())
        ).also(it -> log.info("not-exists-list: {}", it))
);
```

### 고찰
- runAndGet().also().also() 체이닝이 안됨
- 코틀린 범위 지정함수 처럼 사용되길 희망했지만, 한계가 있음