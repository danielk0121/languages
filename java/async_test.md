```java
import lombok.Builder;
import lombok.SneakyThrows;
import lombok.ToString;
import lombok.extern.slf4j.Slf4j;
import org.junit.Test;

import java.util.Arrays;
import java.util.List;
import java.util.Random;
import java.util.concurrent.*;
import java.util.concurrent.atomic.AtomicInteger;

@Slf4j
public class AsyncStudyTest {

    @ToString
    @Builder
    static class ListWrapper {
        List<Long> idList;
    }

    @SneakyThrows
    @Test
    public void test01() {

        long st = System.currentTimeMillis();
//        ExecutorService worker = Executors.newSingleThreadExecutor();
        ExecutorService worker = Executors.newFixedThreadPool(3);
        log.info("create worker ms: " + (System.currentTimeMillis() - st));

        final Random rand = new Random();
        AtomicInteger workerDoneCnt = new AtomicInteger(0);
        AtomicInteger futureGetSuccessCnt = new AtomicInteger(0);
        AtomicInteger futureGetFailCnt = new AtomicInteger(0);

        Callable<ListWrapper> task = new Callable<ListWrapper>() {
            @SneakyThrows
            @Override
            public ListWrapper call() {
                long st = System.currentTimeMillis();
                long waitMills = rand.longs(10, 999).findFirst().getAsLong();
                Thread.sleep(waitMills);  // api call 시뮬레이션
                ListWrapper lw = ListWrapper.builder().idList(Arrays.asList(1L,2L,3L)).build();
                log.info("\twork enddd ms: {}, waitMills: {}", (System.currentTimeMillis() - st), waitMills);
                workerDoneCnt.incrementAndGet();
                return lw;
            }
        };

        int forCnt = 50;
        long futureGetWaitMills = 200;

        for(int i = 0; i < forCnt; i++) {
            long st2 = System.currentTimeMillis();
            Future<ListWrapper> future = worker.submit(task);
            ListWrapper lw;
            try {
                lw = future.get(futureGetWaitMills, TimeUnit.MILLISECONDS);  // 최대 기다리는 시간
            } catch (InterruptedException | ExecutionException | TimeoutException e) {
                log.error("{} {}", e.getClass().getCanonicalName(), e.getMessage());
                futureGetFailCnt.incrementAndGet();
                continue;
            }
            log.info("success ms: {}, {}", (System.currentTimeMillis() - st2), lw);
            futureGetSuccessCnt.incrementAndGet();
        }

        // isGraceShutdown: true, forCnt: 50, workerDoneCnt: 50, futureGetWaitMills: 200, futureGetSuccessCnt: 9, futureGetFailCnt: 41
        // forCnt: 실행시킨 횟수, workerDoneCnt: 쓰레드 작업을 완료한 개수
        // future.get() 에서 TimeoutException 이 발생하더라도, 워커 쓰레드는 실행을 계속함을 확인함
        //
        worker.shutdown();  // 시뮬레이션 종료
        boolean isGraceShutdown = worker.awaitTermination(10, TimeUnit.SECONDS);  // 최대 10초까지 쓰레드 실행 기다림
        log.info("isGraceShutdown: {}, forCnt: {}, workerDoneCnt: {}" +
                        ", futureGetWaitMills: {}, futureGetSuccessCnt: {}, futureGetFailCnt: {}"
                , isGraceShutdown, forCnt, workerDoneCnt
                , futureGetWaitMills, futureGetSuccessCnt, futureGetFailCnt);
    }
}
```
