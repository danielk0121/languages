```shell

yum update 
yum update kernel
yum install -y gcc make perl bison m4 flex elfutils-libelf-devel kernel kernel-headers kernel-devel

# 버츄얼박스 머신 메뉴 > 게스트확장 이미지 삽입
ll /dev/cd*
mount -r /dev/cdrom /media
/media
media/VBoxLinuxAdditions.run --nox11

mount -t vboxsf localvm_shared_dirs  /mnt/shared

vi /etc/profile
mount -t vboxsf localvm_shared_dirs  /mnt/shared

```