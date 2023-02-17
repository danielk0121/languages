```bash
# 보통 이렇게 해왔는데...
# while true
# do
# sleep 2
# date
# curl localhost:9001/status
# curl localhost:9001/sample/db02
# echo
# done

# 꽤 좋은 툴이다
# brew install hey
# -z 20s -z 1m, 1분동안, -q 10, 각 워커당 10qps로, -c 4, 4개 worker 사용
hey -z 20s -z 20m -q 5 -c 4 http://localhost:9001/sample/db02
```

- 리포트도 잘 해준다
```bash
Summary:
  Total:	60.0568 secs
  Slowest:	0.1890 secs
  Fastest:	0.0125 secs
  Average:	0.0341 secs
  Requests/sec:	39.7623


Response time histogram:
  0.012 [1]	|
  0.030 [1478]	|■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■
  0.048 [10]	|
  0.065 [803]	|■■■■■■■■■■■■■■■■■■■■■■
  0.083 [21]	|■
  0.101 [3]	|
  0.118 [14]	|
  0.136 [13]	|
  0.154 [23]	|■
  0.171 [13]	|
  0.189 [9]	|


Latency distribution:
  10% in 0.0140 secs
  25% in 0.0148 secs
  50% in 0.0165 secs
  75% in 0.0572 secs
  90% in 0.0588 secs
  95% in 0.0602 secs
  99% in 0.1512 secs

Details (average, fastest, slowest):
  DNS+dialup:	0.0000 secs, 0.0125 secs, 0.1890 secs
  DNS-lookup:	0.0000 secs, 0.0000 secs, 0.0016 secs
  req write:	0.0000 secs, 0.0000 secs, 0.0012 secs
  resp wait:	0.0339 secs, 0.0123 secs, 0.1887 secs
  resp read:	0.0002 secs, 0.0000 secs, 0.0004 secs

Status code distribution:
  [200]	2388 responses
```