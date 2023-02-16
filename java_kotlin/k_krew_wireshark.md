### kubectl plugin krew sniff

- install
```
(
  set -x; cd "$(mktemp -d)" &&
  OS="$(uname | tr '[:upper:]' '[:lower:]')" &&
  ARCH="$(uname -m | sed -e 's/x86_64/amd64/' -e
 's/\(arm\)\(64\)\?.*/\1\2/' -e 's/aarch64$/arm64/')" &&
  curl -fsSLO "https://github.com/kubernetes-sigs/krew/releases/latest/download/krew.tar.gz" &&
  tar zxvf krew.tar.gz &&
  KREW=./krew-"${OS}_${ARCH}" &&
  "$KREW" install krew
)
```

- use
```
kubectl sniff -n 네임스페이스 팟이름 -c 컨테이너이름 -f "port 8000" -o - | wireshark -i -
kubectl sniff -n scouter scouter-7944b6c9bb-t9mjv -f "port 6100" -o - | wireshark -i -
```

- mac 에서 wireshark 실행하려면 brew 설치가 안되고 홈페이지에서 dmg 파일 다운받아서 설치해야한다
- wireshark 명령어가 없기 때문에... wireshark 명령어 간단 스크립트를 /usr/local/bin 에 추가한다
- code /usr/local/bin/wireshark
```
#!/usr/bin/env bash
WIRESHARK="/Applications/Wireshark.app/Contents/MacOS/Wireshark"
${WIRESHARK} "$@"
exit $?
```

- .zshrc 에 추가
```
export PATH="${KREW_ROOT:-$HOME/.krew}/bin:$PATH"
```
