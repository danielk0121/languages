```shell
# ssh config file

Host i-*
    # User ubuntu
    ProxyCommand sh -c "aws ssm start-session --target %h --document-name AWS-StartSSHSession --parameters 'portNumber=%p'"
    IdentityFile /Users/myusername/tempssh/tempdevkey
```

```shell
fun ec2list() {
  aws ec2 describe-instances \
    --filters Name=tag-key,Values=Name \
    --query 'Reservations[*].Instances[*].{Instance:InstanceId,AZ:Placement.AvailabilityZone,Name:Tags[?Key==`Name`]|[0].Value,Status:State.Name}' \
    --output=table --no-cli-pager
}

fun ec2sendssh() {
  aws ec2-instance-connect send-ssh-public-key \
    --region ap-northeast-2 \
    --availability-zone ap-northeast-2a \
    --instance-id $1 \
    --instance-os-user $2 \
    --ssh-public-key file://~/tempssh/tempdevkey.pub > /dev/null
  echo "OK => ec2-instance-connect send-ssh-public-key file://~/tempssh/tempdevkey.pub"
}

# 방법1 ssm -> ssh config 무관, ssm iam 권한 필요
~ export AWS_DEFAULT_PROFILE=dev && ec2list
~ aws ssm start-session --target i-xxx

# 방법2 send ssh pub key -> ssh config 설정 필요, scp 사용가능
~ ec2sendssh i-xxx ec2-user
~ ssh ec2-user@i-xxx
```
