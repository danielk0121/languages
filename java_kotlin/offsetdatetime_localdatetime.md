```java
import java.time.OffsetDateTime;
import java.time.ZoneId;
import java.time.ZoneOffset;
import java.time.format.DateTimeFormatter;

public class SimpleTest {
    public static void main(String[] args) {
//        OffsetDateTime odtB = OffsetDateTime.parse("2018-03-26T06:00:00Z");
//        odtB = odtB.withOffsetSameInstant(ZoneOffset.ofHours(2));  // convert to offset +02:00

//        OffsetDateTime now = OffsetDateTime.now();  // system clock default timezone
//        OffsetDateTime now00 = now.withOffsetSameInstant(ZoneOffset.ofHours(0));
//        OffsetDateTime now02 = now.withOffsetSameInstant(ZoneOffset.ofHours(2));
//        OffsetDateTime kstNow = now.withOffsetSameInstant(ZoneOffset.ofHours(9));
//        System.out.println(now);
//        System.out.println(now00);  // 2023-06-21T08:59:39.304Z
//        System.out.println(now02);  // 2023-06-21T10:59:39.404+02:00
//        System.out.println(kstNow); // 2023-06-21T17:59:39.404+09:00
//
//        System.out.println(OffsetDateTime.now(ZoneId.of("UTC")));         // 2023-06-21T08:59:39.304Z
//        System.out.println(OffsetDateTime.now(ZoneId.of("Asia/Seoul")));  // 2023-06-21T17:59:39.404+09:00
//        System.out.println(1);

        OffsetDateTime utcNow = OffsetDateTime.now(ZoneId.of("UTC"));
        OffsetDateTime kstNow = utcNow.withOffsetSameInstant(ZoneOffset.ofHours(9));
        OffsetDateTime utcNowPlus = utcNow.plusMinutes(30);

        String utcNowYMD01 = utcNow.format(DateTimeFormatter.ofPattern("yyyy/MM/dd"));
        String utcNowYMD02 = utcNow.format(DateTimeFormatter.ofPattern("yyyy-MM-dd"));
        String utcNowPlusISODateTime = utcNowPlus.format(DateTimeFormatter.ISO_DATE_TIME);  // 2023-06-21T08:59:39.304Z
        String kstNowDateTimeCustom = kstNow.format(DateTimeFormatter.ofPattern("yyyyMMdd_HHmmss")) + "_KST";
    }
}
```