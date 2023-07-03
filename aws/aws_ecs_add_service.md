```
# 아래 순서대로 진행하면 편리함
# 서비스포트: 8080
# 컨테이너 1개

라우트53 > 호스트 주소 생성
타겟그룹 생성 > ip타입, 8080
로드밸런서 > 규칙추가 > 호스트헤더+타겟그룹
=> 타겟그룹을 확인해보면 LB 할당이 없던 부분이 할당됨으로 변경됨

보안그룹 생성 > 인바운드=8080, 아웃바운드=모든트래픽
작업정의 > json 편집으로 수정 > 생성

클러스터 > 서비스생성
클러스터 > 서비스 > 업데이트 > 상태검사유예시간=120초, 서비스개수=1
```

- sample task definition json
```json
{
    "executionRoleArn": "<your ecs role arn>",
    "containerDefinitions": [
        {
            "logConfiguration": {
                "logDriver": "awslogs",
                "options": {
                    "awslogs-group": "<로그 그룹 이름, /ecs/test-myapp>",
                    "awslogs-region": "ap-northeast-2",
                    "awslogs-stream-prefix": "ecs",
                    "awslogs-create-group": "true"
                }
            },
            "portMappings": [ { "hostPort": 8080, "protocol": "tcp", "containerPort": 8080 } ],
            "cpu": 0,
            "ulimits": [ { "name": "nofile", "softLimit": "10240", "hardLimit": "10240" } ],
            "memory": "2048",
            "image": "<your ecr url>",
            "essential": true,
            "name": "batch-crawler",
            "repositoryCredentials": { "credentialsParameter": "" }
        }
    ],
    "memory": "2048",
    "taskRoleArn": "<your ecs role arn>",
    "family": "<작업정의 이름>",
    "requiresCompatibilities": [ "FARGATE" ],
    "networkMode": "awsvpc",
    "runtimePlatform": { "operatingSystemFamily": "LINUX" },
    "cpu": "1024"
}
```