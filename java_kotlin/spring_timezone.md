- mysql, springboot timezone 타임존 통합

```mysql
# mysql 테이블 중 타임존에 관련된 테이블 모두 조회
SELECT TABLE_SCHEMA, TABLE_NAME, COLUMN_NAME, DATA_TYPE, COLUMN_TYPE
FROM INFORMATION_SCHEMA.COLUMNS
WHERE DATA_TYPE in ( 'timestamp', 'date', 'datetime', 'time')
AND TABLE_SCHEMA  like '%myapp%'
```

```mysql
-- now() 조회가 UTC 로 취급하고, Asia/Seoul 로 변경하라 -> +9 됨
SELECT @@system_time_zone, @@global.time_zone, @@session.time_zone, now(), CONVERT_TZ(now(), 'UTC',  'Asia/Seoul') ;

-- now() 조회가 Asia/Seoul 로 취급하고, Asia/Seoul 로 변경하라 -> +0 됨
SELECT @@system_time_zone, @@global.time_zone, @@session.time_zone, now(), CONVERT_TZ(now(), 'Asia/Seoul',  'Asia/Seoul') ;
```

```java
public abstract class SomeResponse {
    // ... 중략 ...

    @JsonProperty("displayBeginAt")
    @JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "yyyy-MM-dd'T'HH:mm:ss.SSSXXX", timezone = "Asia/Seoul")  // server to client, response body 에 사용됨
//    @DateTimeFormat(iso = DateTimeFormat.ISO.DATE_TIME)  // client to server, request body 에 사용됨
    private OffsetDateTime displayBeginAt;

    @JsonProperty("displayEndAt11")
    @JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "yyyy-MM-dd'T'HH:mm:ssXXX", timezone = "Asia/Seoul")
    private OffsetDateTime displayEndAt;

    @JsonProperty("createdAt11111")
    @JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "yyyy-MM-dd'T'HH:mm:ssXXX", timezone = "UTC")
    private OffsetDateTime createdAt;

    @JsonProperty("updatedAt11111")
    private OffsetDateTime updatedAt;
}
```

```json
{
  "displayBeginAt": "2021-10-20T19:00:00.000+09:00",
  "displayEndAt11": "2021-10-20T19:00:00+09:00",
  "createdAt11111": "2021-10-20T10:00:00Z",
  "updatedAt11111": "2021-10-20T10:00:00Z"
}
```