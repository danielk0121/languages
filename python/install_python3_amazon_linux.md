### 아마존 리눅스 파이썬 설치
```shell
sudo yum install gcc openssl-devel bzip2-devel libffi-devel

mkdir temp
cd temp
wget https://www.python.org/ftp/python/3.8.10/Python-3.8.10.tgz
tar -xvf Python-3.8.10.tgz
cd Python-3.8.10

sudo ./configure --enable-optimizations
sudo make install

# 다시 로그인

which python3
# /usr/local/bin/python3

python3 --version
# Python 3.8.10
```

### 아마존 리눅스 버전 확인
```shell
[root@ip-172-31-64-219 ~]# cat /etc/os-release
NAME="Amazon Linux"
VERSION="2"
ID="amzn"
ID_LIKE="centos rhel fedora"
VERSION_ID="2"
PRETTY_NAME="Amazon Linux 2"
ANSI_COLOR="0;33"
CPE_NAME="cpe:2.3:o:amazon:amazon_linux:2"
HOME_URL="https://amazonlinux.com/"
```