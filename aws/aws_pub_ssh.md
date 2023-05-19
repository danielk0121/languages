```shell

mkdir ~/tempssh
cd ~/tempssh

# aws 에 보낼 ssh 임시키를 생성한다
ssh-keygen -t rsa -f temp_prod_key -N '' > /dev/null
chmod 400 temp_prod_key*

# pub 를 aws 에 보낸다
aws ec2-instance-connect send-ssh-public-key \
    --instance-id $EC2_JENKINS_ID \
    --availability-zone ap-northeast-2a \
    --instance-os-user ec2-user \
    --ssh-public-key file://temp_prod_key.pub > /dev/null \
    --profile=prod

# ssh 를 비밀키로 접속한다
ssh -i temp_prod_key \
    ec2-user@$EC2_JENKINS_ID \
    -o ProxyCommand="aws ssm start-session --target %h --document-name AWS-StartSSHSession --parameters portNumber=%p --profile=prod"

# scp 도 사용가능
# ~/.ssh/config 파일에 정보를 미리 입력해 놓아야함
scp ec2-user@$EC2_JENKINS_ID:/home/ec2-user/jenkins_plugins_from_server.tar.gz .

```