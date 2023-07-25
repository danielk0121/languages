- ecr 이미지에서 pull 받아서 컨테이너 이미지 내용 확인
```
# docker login
aws ecr get-login-password --region ap-northeast-2 --profile prod | \
docker login --username AWS --password-stdin <ecr 주소>

# docker pull
docker pull <ecr image 주소>

# docker container create
docker create --name=<컨테이너 이름> <ecr image 주소>:<이미지 태그>

# copy docker container file
mkdir -p ~/tempwork/<컨테이너 이름>
rm -rf ~/tempwork/<컨테이너 이름>
docker cp <컨테이너 이름>:/ ~/tempwork/<컨테이너 이름>
```