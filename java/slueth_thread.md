```java
import brave.Tracer;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.cloud.sleuth.log.Slf4jCurrentTraceContext;
import org.springframework.stereotype.Service;
import org.springframework.util.StopWatch;

import java.util.Objects;
import java.util.concurrent.*;

@Slf4j
@Service
@RequiredArgsConstructor
public class MyAsyncService {

    private final Slf4jCurrentTraceContext traceContext;
    private final Tracer tracer;

    private static final long WAIT_MILLIS = 200;

    public String asyncAndAwaitOrNull() {

        Callable<String> task = () -> {
            // traceId propagation from parent <-- !!
            tracer.startScopedSpanWithParent("callableTask", traceContext.get());
            
            StopWatch watch = new StopWatch();
            watch.start();
            Thread.sleep(1000 * 2);  // do something
            watch.stop();
            log.info("task done, ms: {}", watch.getLastTaskTimeMillis());
            return "done";
        };

        ExecutorService worker = Executors.newSingleThreadExecutor();
        Future<String> future = worker.submit(task);
        
        try {
            return future.get(WAIT_MILLIS, TimeUnit.MILLISECONDS);
        } catch (TimeoutException e) {
            log.info("", e);
        } catch (Exception e) {
            log.error("", e);
        }

        worker.shutdown();
        awaitTerminationAfterShutdown(worker);

        return null;
    }

    private static void awaitTerminationAfterShutdown(ExecutorService threadPool) {
        threadPool.shutdown();
        try {
            if (!threadPool.awaitTermination(20, TimeUnit.SECONDS)) {
                log.error("threadPool awaitTermination false ! shutdownNow");
                threadPool.shutdownNow();
            }
        } catch (InterruptedException ex) {
            threadPool.shutdownNow();
        }
    }
}
```