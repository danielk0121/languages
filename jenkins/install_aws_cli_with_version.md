```shell

# ver1
# https://docs.aws.amazon.com/cli/v1/userguide/install-linux.html

# 기존 명령어 삭제 (사실상 심볼릭링크만 삭제)
rm -rf /usr/local/bin/aws

# 버전에 맞게 다운로드 후 설치, 1.18.147 버전 숫자만 변경하면 됨
curl "https://s3.amazonaws.com/aws-cli/awscli-bundle-1.18.147.zip" -o "awscli-bundle.zip"
unzip awscli-bundle.zip
sudo ./awscli-bundle/install -i /usr/local/aws -b /usr/local/bin/aws

# ver2
# https://docs.aws.amazon.com/cli/latest/userguide/getting-started-version.html

```