```shell

su - jenkins

# install nvm with version
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.38.0/install.sh | bash

```

```shell

# yarn 설치

su - jenkins

# 저장소 등록
curl --silent --location https://rpm.nodesource.com/setup_10.x | sudo bash

# nodejs 설치
sudo yum -y install nodejs

# 저장소 최신 버전 유지 활성화
curl --silent --location https://dl.yarnpkg.com/rpm/yarn.repo | \
sudo tee /etc/yum.repos.d/yarn.repo

sudo rpm --import https://dl.yarnpkg.com/rpm/pubkey.gpg

# yarn 설치
sudo yum -y install yarn
yarn --version

```