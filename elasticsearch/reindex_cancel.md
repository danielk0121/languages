- reindex 명령어 예시
```shell
POST /_reindex
{
  "source": {
    "index": "heartbeat-7.15.0"
  },
  "dest": {
    "index": "heartbeat-7.15.0-000001"
  }
}
```

---

- es reindex 는 인덱스를 새로 복사하는 과정을 거치네
- 그래서 20g 정도 되는 인덱스를 reindex 작업하면 시간이 오래 걸리고
- 취소하고 싶은 상황도 발생한다
- es 가 task 실행 목록 보는 명령어가 있군  
```shell
GET /_cat/tasks?v
GET _tasks?detailed=true&actions=*reindex
GET _tasks/G_cfldrFT52zEJe32BfBjA:303186337
POST _tasks/G_cfldrFT52zEJe32BfBjA:303186337/_cancel
```

- task api 사용 결과 예시
```json
{
  "nodes": {
    "G_cfldrFT52zEJe32BfBjA": {
      "name": "instance-0000000009",
      "transport_address": "10.9.16.30:19682",
      "host": "10.9.16.30",
      "ip": "10.9.16.30:19682",
      "roles": [
        "data_content",
        "data_hot",
        "ingest",
        "master",
        "remote_cluster_client",
        "transform"
      ],
      "attributes": {
        "xpack.installed": "true",
        "region": "ap-northeast-2",
        "instance_configuration": "aws.data.highio.i3",
        "logical_availability_zone": "zone-1",
        "availability_zone": "ap-northeast-2c",
        "server_name": "instance-0000000009.4722f9daccbd44479f522bd3c54e0104"
      },
      "tasks": {
        "G_cfldrFT52zEJe32BfBjA:303186337": {
          "node": "G_cfldrFT52zEJe32BfBjA",
          "id": 303186337,
          "type": "transport",
          "action": "indices:data/write/reindex",
          "status": {
            "total": 12109725,
            "updated": 46257,
            "created": 4743,
            "deleted": 0,
            "batches": 52,
            "version_conflicts": 0,
            "noops": 0,
            "retries": {
              "bulk": 0,
              "search": 0
            },
            "throttled_millis": 0,
            "requests_per_second": -1,
            "throttled_until_millis": 0
          },
          "description": "reindex from [heartbeat-7.15.0] to [heartbeat-7.15.0-000001]",
          "start_time_in_millis": 1675157270794,
          "running_time_in_nanos": 30652486454,
          "cancellable": true,
          "cancelled": false,
          "headers": {
            "trace.id": "82640f4136236b55af8ce62359142aeb"
          }
        }
      }
    }
  }
}
```
