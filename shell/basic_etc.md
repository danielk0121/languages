```shell
# tar 압축
tar -cf jenkins_plugins_from_server.tar.gz /var/lib/jenkins/plugins/

# scp 로컬 2 원격
scp testfile2 root@192.168.159.129:/tmp/testclient
# scp 로컬 2 원격, 여러파일
scp tesfile1 testfile2 root@192.168.159.129:/tmp/testclient
# scp 로컬 2 원격, 폴더
scp -r testgogo root@192.168.159.129:/tmp/testclient

# scp 원격 2 로컬
scp root@192.168.159.129:/tmp/testclient/testfile2 /tmp

```