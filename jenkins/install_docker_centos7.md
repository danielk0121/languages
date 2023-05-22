```shell

# Docker 설치
# https://docs.docker.com/engine/install/centos/ 사이트 자료를 참고하여 설치한다.
yum -y update
yum install -y yum-utils

# Docker repository 시스템에 추가
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum-config-manager --enable docker-ce-nightly
 
# 최신버전의 Docker 설치(Install Docker Engine)
yum -y install docker-ce docker-ce-cli containerd.io
 
# Docker 데몬 시작 및 부팅 시 Docker 데몬 자동 시작
systemctl start docker
systemctl enable docker
 
# Docker 실행중인지 확인
systemctl status docker

# docker sock permission denied
# 666 으로 권한 주거나, chown 으로 변경, 또는 특정 사용자 권한 추가
sudo chmod 666 /var/run/docker.sock
sudo chown root:docker /var/run/docker.sock

```