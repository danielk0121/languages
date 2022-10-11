- gradle 의존성 다운로드 받는 부분이 오래 걸린다
- docker cache 가 적용되도록 다운로드 받는 부분을 분리해서 적용

Dockerfile
```dockerfile
FROM jdk-11.0.11_9-alpine AS builder

# gradle 빌드를 작업할 폴더로 이동
WORKDIR /build

# 컨테이너 내부에서 gradle 빌드 하기 위한 파일 추가
COPY gradlew /build
COPY gradle /build/gradle
COPY build.gradle.kts /build
COPY settings.gradle.kts /build

# source 파일이 없어서 빌드 실패하도록 유도함, 의존패키지만 다운로드 되도록
# gradle 파일이 변경되었을 때만 새롭게 의존패키지 다운로드 받게함
RUN chmod +x ./gradlew; sync && ./gradlew
RUN gradle build -x test --parallel --continue > /dev/null 2>&1 || true

# 실제 어플리케이션 소스 빌드
COPY . /build
RUN ./gradlew clean build -x test --no-daemon --parallel --no-build-cache

# 멀티 스테이징 사용 => 이미지 용량 감소, 캐시 활성화
FROM jdk-11.0.11_9-alpine
WORKDIR /app
ARG APP_ENV
ARG JAVA_OPTS="-XX:MaxRAMPercentage=75.0 -XX:MinRAMPercentage=75.0"
ARG GIT_COMMIT_ID_ABBREV=$GIT_COMMIT_ID_ABBREV

ENV APP_ENV=$APP_ENV
ENV JAVA_OPTS=$JAVA_OPTS
ENV SPRING_PROFILES_ACTIVE=$APP_ENV
ENV GIT_COMMIT_ID_ABBREV=$GIT_COMMIT_ID_ABBREV

COPY --from=builder /build/build/libs/*.jar app.jar
EXPOSE 8080
ENTRYPOINT [ "sh", "-c", "java $JAVA_OPTS -Djava.security.egd=integrationsfile:/dev/./urandom -Duser.timezone=Asia/Seoul -jar ./app.jar" ]
```

docker 빌드
```shell
docker build -t my-app:0.1 \
--build-arg APP_ENV=local-docker-dev \
--build-arg 'JAVA_OPTS=-Xms1024m -Xmx1024m' \
-f Dockerfile .
```

docker run
```shell
docker run -p 8080:8080 my-app:0.1
```

.dockerignore 파일 => 빌드 속도 개선, 이미지 용량 감소
```shell
.gradle/
.idea/
build/
out/
bin/
```