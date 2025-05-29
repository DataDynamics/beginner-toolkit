# Linux

## 서비스 관리 (수동)

systemd가 아닌 스크립트를 통해 관리하기 위한 방법

### Start

```shell
#!/bin/sh

APP_NAME="watcher.py"
CONFIG="config.yaml"
PID_FILE="app.pid"
LOG_FILE="console.log"

# Check if already running
if [ -f "$PID_FILE" ]; then
    PID=$(cat "$PID_FILE")
    if ps -p $PID > /dev/null; then
        echo "Application already running with PID $PID"
        exit 1
    fi
fi

# Start the application in background
nohup python3 "$APP_NAME" --config="$CONFIG" > "$LOG_FILE" 2>&1 &
PID=$!
echo $PID > "$PID_FILE"
echo "Started $APP_NAME with PID $PID"
```

### Stop

```shell
#!/bin/sh

PID_FILE="app.pid"

if [ ! -f "$PID_FILE" ]; then
    echo "No PID file found. Is the app running?"
    exit 1
fi

PID=$(cat "$PID_FILE")

if ps -p $PID > /dev/null; then
    echo "Stopping application with PID $PID"
    kill $PID
    rm "$PID_FILE"
else
    echo "No process found with PID $PID"
    rm "$PID_FILE"
fi
```

### Status

```shell
#!/bin/sh

PID_FILE="app.pid"

if [ ! -f "$PID_FILE" ]; then
    echo "Application is not running (no PID file found)."
    exit 1
fi

PID=$(cat "$PID_FILE")

if ps -p $PID > /dev/null; then
    echo "Application is running with PID $PID."
    exit 0
else
    echo "Application is not running (stale PID file found)."
    exit 1
fi
```

## `lsof`

lsof는 List Open Files의 약자로, 리눅스 및 유닉스 계열 시스템에서 열린 파일(파일 디스크립터)을 확인할 수 있는 강력한 명령어입니다. 
리눅스에서는 파일뿐만 아니라 디렉토리, 라이브러리, 네트워크 소켓 등도 모두 "파일"로 취급되므로 lsof는 매우 다양한 상황에서 유용하게 사용됩니다.

* 특정 파일을 사용 중인 프로세스 확인 - `lsof /path/to/file`
* 특정 포트 사용 프로세스 확인 - `lsof -i :포트번호`
* 프로세스 ID(PID)로 열린 파일 확인 - `lsof -p <PID>`
* 특정 유저가 연 파일 보기 - `lsof -u <username>`
* 삭제되었지만 여전히 열려 있는 파일 찾기 - `lsof | grep deleted`
* 특정 디렉토리나 장치에 열린 파일 찾기 - `lsof +D /path/to/directory`
* 특정 네트워크 연결 보기
  * `lsof -i @192.168.0.100:22`
  * `lsof -i tcp`
  * `lsof -i udp`
* 특정 포트를 점유하는 프로세스 강제 종료 - `kill -9 $(lsof -t -i :80)`

## `ulimit`

ulimit은 리눅스 시스템에서 셸(Shell) 및 해당 셸에서 실행된 프로세스들이 사용할 수 있는 리소스의 한계를 설정하는 명령어입니다. 
시스템의 안정성과 보안을 위해 각 사용자에게 할당된 자원(CPU 시간, 메모리, 파일 개수 등)에 제한을 걸 수 있습니다.

* ulimit은 커널이 아닌 셸 레벨에서 설정하는 리소스 제한입니다.
* 제한되는 리소스는 사용자당 적용되며, 해당 사용자가 실행하는 프로세스에도 적용됩니다.
* ulimit 설정은 크게 두 가지로 나뉩니다:
  * soft limit: 현재 적용 중인 값 (사용자가 줄일 수 있음)
  * hard limit: 시스템이 허용하는 최대값 (root만 변경 가능)

### 리소스 항목

| 항목                 | 설명                        | 옵션   |
| ------------------ | ------------------------- | ---- |
| open files         | 열 수 있는 파일 최대 수            | `-n` |
| max user processes | 생성 가능한 프로세스 수             | `-u` |
| file size          | 생성할 수 있는 파일의 최대 크기 (KB)   | `-f` |
| core file size     | core dump 파일 최대 크기        | `-c` |
| stack size         | 스택 크기 제한 (KB)             | `-s` |
| CPU time           | 프로세스가 사용할 수 있는 CPU 시간 (초) | `-t` |
| virtual memory     | 가상 메모리 크기 (KB)            | `-v` |

### 현재 ulimit 값 확인

```
ulimit -a        # 모든 설정 확인
ulimit -n        # 열린 파일 수 확인
ulimit -u        # 최대 프로세스 수 확인
```

### 일시적으로 ulimit 설정 변경

현재 쉘에서만 유효하고, 리부팅 및 재접속시 초기화

```
ulimit -n 65535
ulimit -u 4096
```

### 영구적으로 ulimit 설정 변경

```
vi /etc/security/limits.conf
# 형식: <사용자> <타입> <리소스> <값>
kim    soft    nofile    65535
kim    hard    nofile    65535
kim    soft    nproc     4096
kim    hard    nproc     4096
```

### systemd 서비스용 ulimit 변경

```
systemctl edit <서비스명> 또는 vi /etc/systemd/system/<서비스>.service
[Service]
LimitNOFILE=65535
LimitNPROC=4096
```

### /etc/security/limits.d/ 경로 사용 (권장)

```
vi /etc/security/limits.d/99-custom.conf
* soft nofile 65535
* hard nofile 65535
```

### 확인방법

```
# 데몬의 경우
cat /proc/<pid>/limits
```

## `systemctl` 및 systemd

systemctl은 systemd 시스템 및 서비스 관리자로, Linux 시스템에서 서비스(데몬) 시작/중지/재시작, 부팅 시 자동 실행 설정, 시스템 상태 확인, 로그 관리, 시스템 재부팅 등 다양한 작업을 수행할 수 있는 명령어입니다.
RHEL 7 이상, Ubuntu 16.04 이상, CentOS 7 이상 등 대부분의 최신 리눅스에서 init 시스템을 대체합니다.

### 기본 명령 구조

```
systemctl [옵션] [명령] [서비스명/타겟]
systemctl start nginx.service
```

### 서비스 제어 명령어

| 명령           | 설명                       |
| ------------ | ------------------------ |
| `start`      | 서비스 시작                   |
| `stop`       | 서비스 중지                   |
| `restart`    | 서비스 재시작                  |
| `reload`     | 설정 다시 읽기 (프로세스 재시작 없이)   |
| `status`     | 현재 상태 확인                 |
| `enable`     | 부팅 시 자동 시작 설정            |
| `disable`    | 부팅 시 자동 시작 해제            |
| `is-active`  | 현재 실행 여부 확인              |
| `is-enabled` | 부팅 시 자동 시작 여부 확인         |
| `mask`       | 서비스 실행 자체를 막음 (start 불가) |
| `unmask`     | mask 해제                  |

### 예제

```
# nginx 시작
sudo systemctl start nginx

# nginx 상태 확인
systemctl status nginx

# nginx 재시작
sudo systemctl restart nginx

# nginx 부팅시 자동 시작
sudo systemctl enable nginx

# 부팅시 자동 시작 해제
sudo systemctl disable nginx
```

### 서비스 단위 파일 (Unit File)

서비스 정의 파일 경로:
* `/etc/systemd/system/` : 사용자 정의
* `/lib/systemd/system/` 또는 `/usr/lib/systemd/system/` : 패키지 제공
* `/run/systemd/system/` : 런타임 생성

### 서비스 파일을 수정한 후 적용

```
systemctl daemon-reload
```

### 서비스 목록 보기

```
systemctl list-units --type=service --all
systemctl --type=service
systemctl | grep ssh
```

### 서비스 로그 확인

```
journalctl -u nginx         # nginx 서비스 전체 로그
journalctl -u nginx -b      # 현재 부팅 이후 로그
journalctl -u nginx --since "1 hour ago"
```

실시간으로 확인하는 방법

```
journalctl -u nginx -f
```

### 시스템 자체 제어

| 명령          | 설명      |
| ----------- | ------- |
| `reboot`    | 시스템 재부팅 |
| `poweroff`  | 시스템 종료  |
| `halt`      | 시스템 중지  |
| `suspend`   | 절전 모드   |
| `hibernate` | 하이버네이션  |

```
sudo systemctl reboot
```

### 유용한 옵션 요약

| 명령어                          | 설명                   |
| ---------------------------- | -------------------- |
| `systemctl list-unit-files`  | 모든 서비스의 활성/비활성 상태 보기 |
| `systemctl is-active <서비스>`  | 실행 중인지 확인            |
| `systemctl is-enabled <서비스>` | 부팅 시 시작 여부 확인        |
| `systemctl show <서비스>`       | 내부 설정 정보 보기          |
| `systemctl cat <서비스>`        | 단위 파일 내용 보기          |




