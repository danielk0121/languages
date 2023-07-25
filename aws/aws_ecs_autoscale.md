- ecs auto scale 설정
- ecs 오토스케일은 desire 개수만 변경 가능하다
- 2개를 설정해야 됨
  - scalable-targets : 등록, 제거 설정을 함
  - scheduled-actions : cron 등으로 스케줄링을 설정함

```shell
# aws config 기본 프로파일 변경 및 조회
export AWS_DEFAULT_PROFILE=dev && aws configure list

# 오토스케일링 상태 조회
aws --no-cli-pager application-autoscaling describe-scalable-targets --service-namespace=ecs
aws --no-cli-pager application-autoscaling describe-scheduled-actions --service-namespace=ecs

# ecs 클러스터, 서비스 목록 조회
aws ecs list-clusters
aws --no-cli-pager ecs list-services --cluster mycluster

----

# 변수
ECS_SERVICE_RESOURCE_ID="service/mycluster/api-myapp"

# scalable-target 조회
aws --no-cli-pager application-autoscaling describe-scalable-targets --service-namespace=ecs

# scalable-target 등록
aws --no-cli-pager application-autoscaling register-scalable-target \
--service-namespace ecs --scalable-dimension ecs:service:DesiredCount --resource-id $ECS_SERVICE_RESOURCE_ID \
--min-capacity 1 --max-capacity 1

# scalable-target 제거
aws --no-cli-pager application-autoscaling deregister-scalable-target \
--service-namespace ecs --scalable-dimension ecs:service:DesiredCount --resource-id $ECS_SERVICE_RESOURCE_ID

----

# 스케줄액션 조회
#SCHEDULED_ACTION_NAME="api-myapp-scaleout-test"
#SCHEDULED_ACTION_NAME="api-myapp-scalein-test"
aws --no-cli-pager application-autoscaling describe-scheduled-actions --service-namespace=ecs --scheduled-action-names $SCHEDULED_ACTION_NAME

# 스케줄액션 등록수정
SCHEDULED_ACTION_NAME="api-myapp-scaleout-test"
SCHEDULED_ACTION_RESOURCE_ID="service/mycluster/api-myapp"
SCHEDULED_ACTION_SCHEDULE="cron(0 7 * * ? *)"
SCHEDULED_ACTION_TIMEZONE="Asia/Seoul"
SCHEDULED_ACTION_MIN=3
SCHEDULED_ACTION_MAX=3

SCHEDULED_ACTION_NAME="api-myapp-scalein-test"
SCHEDULED_ACTION_RESOURCE_ID="service/mycluster/api-myapp"
SCHEDULED_ACTION_SCHEDULE="cron(0 18 * * ? *)"
SCHEDULED_ACTION_TIMEZONE="Asia/Seoul"
SCHEDULED_ACTION_MIN=1
SCHEDULED_ACTION_MAX=1

aws application-autoscaling put-scheduled-action \
--service-namespace ecs --scalable-dimension ecs:service:DesiredCount \
--resource-id $SCHEDULED_ACTION_RESOURCE_ID \
--scheduled-action-name $SCHEDULED_ACTION_NAME \
--schedule $SCHEDULED_ACTION_SCHEDULE \
--timezone $SCHEDULED_ACTION_TIMEZONE \
--scalable-target-action MinCapacity=$SCHEDULED_ACTION_MIN,MaxCapacity=$SCHEDULED_ACTION_MAX

# 스케줄액션 json 으로 등록 !!
echo '{
    "ServiceNamespace": "ecs",
    "Schedule": "cron(0 18 * * ? *)",
    "Timezone": "Asia/Seoul",
    "ScheduledActionName": "api-myapp-scalein-test2",
    "ResourceId": "service/mycluster/api-myapp",
    "ScalableDimension": "ecs:service:DesiredCount",
    "ScalableTargetAction": {
        "MinCapacity": 1,
        "MaxCapacity": 1
    }
}' | xargs -0 aws application-autoscaling put-scheduled-action --cli-input-json

# 스케줄액션 삭제
aws application-autoscaling delete-scheduled-action \
--service-namespace=ecs \
--scalable-dimension=ecs:service:DesiredCount \
--resource-id=$SCHEDULED_ACTION_RESOURCE_ID \
--scheduled-action-name=$SCHEDULED_ACTION_NAME
```