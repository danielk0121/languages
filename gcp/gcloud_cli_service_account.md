### 구글 클라우드 cli 설치 리눅스
```shell
# https://cloud.google.com/sdk/docs/install?hl=ko

curl -O https://dl.google.com/dl/cloudsdk/channels/rapid/downloads/google-cloud-cli-432.0.0-linux-x86_64.tar.gz
tar -xf google-cloud-cli-432.0.0-linux-x86_64.tar.gz
./google-cloud-sdk/install.sh

# init 실행 이후 로그인 하려면 y 하라는 메세지가 뜬다, n 를 선택해서 일단 로그인 취소
./google-cloud-sdk/bin/gcloud init

# 서비스 계정 생성 후 생성한 key json 파일을 사용해서 등록
gcloud auth activate-service-account my-service-account@my-project-name.iam.gserviceaccount.com --key-file=/path/key.json--project=my-proejct-name
```