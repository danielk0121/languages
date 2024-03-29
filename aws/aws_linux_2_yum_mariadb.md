```shell
# 아마존 리눅스에 amazonlinux2 가 있고 amazonlinux 2023 (2023이 3이다) 가 있다
# 아마존 리눅스는 레드햇 기반이지만 레드헷 6,7,8,9 어떤버전인지 잘 봐야 한다
# 그래야 yum repo 에서 리눅스 버전을 정할 수 있는데...
# yum repo 설치를 자동으로 해주는 shell 스크립트를 실행하면 된다 !
#
# 주의할점은 mariadb 가 x86 이 아닌 환경에서는 지원을 중단한다고 한다 !
# 따라서... arm64 기반인 graviton aws cpu 를 사용하고 싶어도 못쓰는 상황이라는거
# arm64에서 설치를 진행하면, 패키지 자동화 도구가 패키지 깨졌다면서 설치가 안된다
# 테스트 환경 : 아마존리눅스3, arm64, mariadb10.4
#
# 아래는 yum repo 설치해주는 shell 스크립트를 다운로드 후 설치하는 명령어들이다
# 아.. yum update 후 reboot 을 해주라는 블로그가 많던데 해주자

# ec2 생성 후 쉘에 접속
yum update
reboot

# 루트 로그인
sudo su - 

# yum repo 생성 해주는 쉘 다운로드
curl -LsS https://r.mariadb.com/downloads/mariadb_repo_setup -O

# 쉘 실행
sudo bash mariadb_repo_setup --os-type=rhel --os-version=7 --mariadb-server-version=10.4

# mariadb 설치
sudo yum install -y MariaDB-client
sudo yum install MariaDB-server MariaDB-client

# rds 접속
mysql --host=testdb.xxxxxxx.ap-northeast-2.rds.amazonaws.com --port=3306 --user='test' --password='test'
```

```text
# 출처 : https://stackoverflow.com/questions/75452027/can-not-install-mariadb-on-amazon-linux-arm-packages-not-found

비 x86_64 플랫폼용 Centos7은 더 이상 사용되지 않습니다. 
참조: https://mariadb.com/kb/en/deprecation-policy/
Centos 7을 제공하는 RHEL7은 비 86_64 플랫폼에 대한 지원이 종료되었습니다. 
Centos7은 여전히 aarch64에 대해 지원된다고 주장 하지만 
버그 수정 (RHEL에보고 된 MariaDB 개발에 영향을 미치고 RHEL8에서만 수정 된 버그 포함)을 
얻지 못하므로 모든 집중적 인 목적을 위해 유지 관리되지 않습니다.

따라서 MariaDB는 더 이상 비 x86_64 플랫폼에서 Centos/RHEL 7 릴리스를 수행하지 않습니다.
참조: MDEV-30636 : https://jira.mariadb.org/browse/MDEV-30636
```