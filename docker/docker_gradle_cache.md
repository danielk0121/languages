gradle 의존성 다운로드 받는 부분이 오래 걸린다
docker cache 가 적용되도록 다운로드 받는 부분을 분리해서 적용

```dockerfile
FROM 992575464873.dkr.ecr.ap-northeast-2.amazonaws.com/n.code-java-base:jdk-11.0.11_9-alpine AS builder

WORKDIR /build

COPY gradlew /build
COPY gradle /build/gradle
COPY build.gradle.kts /build
COPY settings.gradle.kts /build

# 그래들 파일이 변경되었을 때만 새롭게 의존패키지 다운로드 받게함.
RUN chmod +x ./gradlew; sync && ./gradlew
RUN gradle build -x test --parallel --continue > /dev/null 2>&1 || true

# 빌더 이미지에서 애플리케이션 빌드
COPY . /build
RUN ./gradlew clean build -x test --no-daemon --parallel --no-build-cache

FROM 992575464873.dkr.ecr.ap-northeast-2.amazonaws.com/n.code-java-base:jdk-11.0.11_9-alpine
WORKDIR /app
ARG APP_ENV
ARG APM_ENV
ARG APP_NAME
ARG APM_SERVER_URL
ARG APM_SECRET
ARG JAVA_OPTS="-XX:MaxRAMPercentage=75.0 -XX:MinRAMPercentage=75.0"
ARG GIT_COMMIT_ID_ABBREV=$GIT_COMMIT_ID_ABBREV
ARG GIT_BRANCH=$GIT_BRANCH
ARG GIT_COMMIT_TIME=$GIT_COMMIT_TIME
ARG GIT_TAGS=$GIT_TAGS
ARG APM_AGENT_VERSION

ENV APP_ENV=$APP_ENV
ENV APM_ENV=$APM_ENV
ENV APP_NAME=$APP_NAME
ENV APM_SERVER_URL=$APM_SERVER_URL
ENV APM_SECRET=$APM_SECRET
ENV JAVA_OPTS=$JAVA_OPTS
ENV SPRING_PROFILES_ACTIVE=$APP_ENV
ENV GIT_COMMIT_ID_ABBREV=$GIT_COMMIT_ID_ABBREV
ENV GIT_BRANCH=$GIT_BRANCH
ENV GIT_COMMIT_TIME=$GIT_COMMIT_TIME
ENV GIT_TAGS=$GIT_TAGS
ADD https://repo1.maven.org/maven2/co/elastic/apm/elastic-apm-agent/${APM_AGENT_VERSION}/elastic-apm-agent-${APM_AGENT_VERSION}.jar elastic-apm-agent.jar

COPY --from=builder /build/build/libs/*.jar app.jar
EXPOSE 8080
ENTRYPOINT [ "sh", "-c", "java $JAVA_OPTS -javaagent:elastic-apm-agent.jar -Delastic.apm.service_name=$APP_NAME -Delastic.apm.environment=$APM_ENV -Delastic.apm.server_urls=$APM_SERVER_URL -Delastic.apm.secret_token=$APM_SECRET -Delastic.apm.application_packages=dcode -Djava.security.egd=integrationsfile:/dev/./urandom -Duser.timezone=Asia/Seoul -jar ./app.jar" ]

```