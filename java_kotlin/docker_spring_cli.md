```shell
# SPRING_PROFILES_ACTIVE 도커 내부의 환경변수를 덮어씀
docker run -p 8080:8080 --name batch-search -e SPRING_PROFILES_ACTIVE=local-jakarta batch-search:local

# SPRING_APPLICATION_JSON 환경변수를 선언해서 spring 에서 property 를 덮어쓰도록 설정
docker run -p 8080:8080 --name batch-search -e SPRING_APPLICATION_JSON='{"spring":{"datasource":{"url":"jdbc:mysql://host.docker.internal:3303/mydb"}}}' batch-search:local
```