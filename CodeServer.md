# Visual Studio Code Server

Visual Studio Code Server는 RHEL/CentOS/Ubuntu 환경에서 원격에서 웹 브라우저를 기반으로 편집 등이 가능하므로 개발 환경 구성시 장점이 많이 발생하나 보안 문제가 발생할 수 있습니다.

## Install

https://github.com/coder/code-server/releases 에서 OS에 맞는 바이너리를 다운로드드합니다.

```
# yum install code-server-4.102.1-amd64.rpm
```

## Systemd 등록

```
# systemctl enable --now code-server@$USER
```

## Configuration

설정 파일은 앞서 설정한 사용자로 로그인하여 생성하도록 합니다.

```
# vim ~/.config/code-server/config.yaml
bind-addr: 0.0.0.0:8090
auth: password
password: @123qwe
cert: false
```
