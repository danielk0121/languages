```shell
# git user, email 위치

global, 모든 레포 : ~/gitconfig
local, 해당 레포 : .git/config 
```

```shell
# git config 설정 명령어

git config [local or global] [category].[parameter] [value]

# git config 확인
git --no-pager config -l
git --no-pager config --list 
```

```shell
# 해당 레포에만 git commit user 변경 적용하기

cd <git repository clone folder>
git config --local user.name "danielk0121"
git config --local user.email "maedk10@gmail.com"
cat .git/config
```
