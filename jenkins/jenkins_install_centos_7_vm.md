```shell
### virtualbox centos 7 설치
###

# 방화벽 해제
systemctl stop firewalld
systemctl disable firewalld
systemctl status firewalld

# 랜카드 업, ifconfig
ip addr
vi /etc/sysconfig/network-scripts/ifcfg-ens03
ifup ifcfg-ens03
hostnamectl set-hostname jenkinstest
yum install -y net-tools

# mariadb connector
yum update
reboot
curl -LsS https://r.mariadb.com/downloads/mariadb_repo_setup -O
sudo bash mariadb_repo_setup --os-type=rhel --os-version=7 --mariadb-server-version=10.4
sudo yum install -y MariaDB-client
mysql --host=testdb.xxxxxxx.ap-northeast-2.rds.amazonaws.com --port=3306 --user='test' --password='test'

# java install
yum list java*jdk-devel
yum -y install java-11-openjdk-devel.x86_64

###############################
# jenkins ps -ef

jenkins   4374     1  0 May15 ?        00:24:35 
/etc/alternatives/java 
-Dcom.sun.akuma.Daemon=daemonized 
-Djava.awt.headless=true 
-Dorg.apache.commons.jelly.tags.fmt.timeZone=Asia/Seoul 
-DJENKINS_HOME=/var/lib/jenkins 
-jar /usr/lib/jenkins/jenkins.war 
--logfile=/var/log/jenkins/jenkins.log 
--webroot=/var/cache/jenkins/war 
--daemon 
--httpPort=8080 
--debug=5 
--handlerCountMax=100 
--handlerCountMaxIdle=20
###############################

# install jenkins
yum -y install wget
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key


###############################

# yum centos repository to kakao repository

bzip2 /etc/yum.repos.d/CentOS-*.repo
yum repolist

echo '[base]
name=CentOS-$releasever - Base
baseurl=http://ftp.daumkakao.com/centos/$releasever/os/$basearch/
gpgcheck=0 
[updates]
name=CentOS-$releasever - Updates
baseurl=http://ftp.daumkakao.com/centos/$releasever/updates/$basearch/
gpgcheck=0
[extras]
name=CentOS-$releasever - Extras
baseurl=http://ftp.daumkakao.com/centos/$releasever/extras/$basearch/
gpgcheck=0' > /etc/yum.repos.d/Daum.repo
yum repolist

#########################################

# 젠킨스 버전에 따라 사용하는 java 버전이 다름
Running Jenkins system
Jenkins requires Java 11 or 17 since Jenkins 2.357 and LTS 2.361.1.

# java 설치
sudo yum install java-1.8.0-openjdk-devel
sudo yum -y install java-11-openjdk-devel.x86_64
sudo alternatives --config java

# 젠킨스 설치 계속
curl --silent --location http://pkg.jenkins-ci.org/redhat-stable/jenkins.repo | sudo tee /etc/yum.repos.d/jenkins.repo
sudo rpm --import https://jenkins-ci.org/redhat/jenkins-ci.org.key
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key

# 젠킨스 설치확인, 실행
yum list installed *jenkins*
sudo yum install jenkins
sudo systemctl start jenkins


#########################################

## 젠킨스 원하는 버전 설치

# java 설치
sudo yum list java*jdk-devel
sudo yum install java-1.8.0-openjdk-devel
sudo yum -y install java-11-openjdk-devel.x86_64
sudo alternatives --config java

# jenkins 는 rpm 으로 설치
https://get.jenkins.io/redhat-stable/
젠킨스 다운로드 홈페이지에 들어가서 찾는 버전에 맞는 rpm 파일 다운로드 주소를 복사함
sudo yum install https://get.jenkins.io/redhat-stable/jenkins-2.289.3-1.1.noarch.rpm
sudo yum list *jenkins*
sudo systemctl start jenkins

# centos 7 git 최신버전 설치
# centos 7 저장소에는 git 1.8 까지밖에 없다
yum install http://opensource.wandisco.com/centos/7/git/x86_64/wandisco-git-release-7-1.noarch.rpm
yum remove git
yum install git
git --version

# jenkins 유저로 로그인
# 안될경우 sudo 명령어로 로그인
# passwd 파일보고 사용자의 홈 경로, 기본 사용 쉘 확인
# 
# jenkins:x:998:996:Jenkins Automation Server:/var/lib/jenkins:/bin/bash
# jenkins : user id
# /var/lib/jenkins : home
# /bin/bash : 기본 사용 쉘, /bin/false 인 경우 /bin/sh 나 /bin/bash 로 변경
# 
su - jenkins
sudo -u jenkins -s
whoami
pwd
cat /etc/passwd

# yum 패키지 매니저는 htop 이 기본으로 포함되어있지 않기 때문에
# EPEL repository 를 추가한다.
# htop 설치
$ sudo yum -y install epel-release
$ sudo yum -y install htop

# aws command not found 에러 !
# 
# 젠킨스 ui > /configure > Global Properties
# > Environment variables > 키-값 목록
# 키 : PATH+usr-local
# 값 : /usr/local/bin

# aws cli 1.18.147 설치
# aws profile 설정 (default, dcode-develop-jenkins 2개 프로파일 필수)
# default = 9계정 production
# dcode-develop-jenkins = 6계정 development

# docker 설치

# jq 설치

```