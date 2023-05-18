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
yum list installed
sudo yum install jenkins
sudo systemctl start jenkins


```