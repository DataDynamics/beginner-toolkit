# Linux

- [Linux](#linux)
  - [서비스 관리 (수동)](#서비스-관리-수동)
    - [Start](#start)
    - [Stop](#stop)
    - [Status](#status)
  - [`lsof`](#lsof)
  - [`ulimit`](#ulimit)
    - [리소스 항목](#리소스-항목)
    - [현재 ulimit 값 확인](#현재-ulimit-값-확인)
    - [일시적으로 ulimit 설정 변경](#일시적으로-ulimit-설정-변경)
    - [영구적으로 ulimit 설정 변경](#영구적으로-ulimit-설정-변경)
    - [systemd 서비스용 ulimit 변경](#systemd-서비스용-ulimit-변경)
    - [/etc/security/limits.d/ 경로 사용 (권장)](#etcsecuritylimitsd-경로-사용-권장)
    - [확인방법](#확인방법)
  - [메모리  사용량 체크](#메모리--사용량-체크)
  - [`systemctl` 및 systemd](#systemctl-및-systemd)
    - [기본 명령 구조](#기본-명령-구조)
    - [서비스 제어 명령어](#서비스-제어-명령어)
    - [예제](#예제)
    - [서비스 단위 파일 (Unit File)](#서비스-단위-파일-unit-file)
    - [서비스 파일을 수정한 후 적용](#서비스-파일을-수정한-후-적용)
    - [서비스 목록 보기](#서비스-목록-보기)
    - [서비스 로그 확인](#서비스-로그-확인)
    - [시스템 자체 제어](#시스템-자체-제어)
    - [유용한 옵션 요약](#유용한-옵션-요약)
    - [참고용 서비스 파일](#참고용-서비스-파일)
      - [Spring Boot Application](#spring-boot-application)
      - [Python Application](#python-application)

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

## 메모리  사용량 체크

`top` 커맨드를 통해서 점유 메모리의 크기를 확인하도록 합니다.

```
top - 09:16:10 up 13 days, 23:34,  1 user,  load average: 0.04, 0.09, 0.18
Tasks: 333 total,   2 running, 331 sleeping,   0 stopped,   0 zombie
%Cpu(s):  2.8 us,  0.9 sy,  0.0 ni, 95.8 id,  0.1 wa,  0.2 hi,  0.1 si,  0.1 st
MiB Mem :  31164.7 total,    436.0 free,  16840.0 used,  13888.8 buff/cache
MiB Swap:  20479.0 total,  20476.0 free,      3.0 used.  13733.4 avail Mem 

    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
   3297 clouder+  20   0   10.1g   4.0g 443604 S  15.3  13.0   2031:32 java
   1005 root      20   0 2380840  85184  18180 S   9.3   0.3 218:55.19 cmagent
   1076 clouder+  20   0   10.3g   5.8g  27924 S   3.0  18.9   1420:51 java
   2156 root      20   0 5563000 102464  17144 S   1.3   0.3 227:44.60 python3.8
   3332 clouder+  20   0 8356828   2.3g 494756 S   1.3   7.5 382:26.32 java
   3273 clouder+  20   0 7375612   1.3g  51588 S   1.0   4.3  88:22.09 java
   3252 clouder+  20   0 7123744   1.4g  19740 S   0.7   4.7  55:33.67 java
... 생략
```

`top` 커맨드는 다음의 정보를 표시하며 이중에서 `RES` 항목에 집중하도록 합니다.

| 컬럼          | 설명|
| ----------- | -------------------------------------------- |
| **PID**     | 프로세스 ID (Process ID)|
| **USER**    | 프로세스를 실행한 사용자 이름  |
| **PR**      | 프로세스의 우선순위 (Priority) <br> 낮을수록 우선순위가 높음 |
| **NI**      | nice 값 (Niceness) <br> -20(가장 높은 우선순위) \~ 19(가장 낮은 우선순위)|
| **VIRT**    | 가상 메모리 사용량 (Virtual Memory) <br> 프로세스가 사용하거나 예약한 전체 메모리 (swap 포함)|
| **RES**     | 실제 메모리 사용량 (Resident Set Size) <br> 실제로 RAM에 올라가 있는 메모리|
| **SHR**     | 공유 메모리 크기 (Shared Memory) <br> 다른 프로세스와 공유하는 라이브러리 등|
| **S**       | 프로세스 상태 (State) <br> <ul><li>`R` = 실행 중(Running)</li><li>`S` = 슬립(Sleeping)</li><li>`D` = 디스크 I/O 대기</li><li>`Z` = 좀비(Zombie)</li><li>`T` = 일시 중지(Traced/Stopped)</li></ul> |
| **%CPU**    | CPU 사용률 (%) <br> 단일 CPU 기준|
| **%MEM**    | 물리 메모리(RAM) 사용률 (%) |
| **TIME+**   | 누적 CPU 사용 시간 (초 단위 + 소수점: mm\:ss.hh) |
| **COMMAND** | 실행 중인 명령 또는 프로세스 이름 (경로 또는 커맨드라인 일부 포함 가능) |

서버의 Core 별로 CPU 사용량을 확인하려면 `htop`을 사용하도록 합니다.

![htop](https://htop.dev/images/htop-2.0.png)

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

### 참고용 서비스 파일

#### Spring Boot Application

```
[Unit]
Description=Spring Boot Application - MyApp
After=network.target

[Service]
User=springuser
WorkingDirectory=/opt/myapp

# 환경변수 파일 지정 (예: /etc/myapp/myapp.env)
EnvironmentFile=/etc/myapp/myapp.env

# 실행 명령
ExecStart=/usr/bin/java -jar /opt/myapp/myapp.jar

# 최대 open file 수 설정
LimitNOFILE=65535

# Java 앱의 종료 코드 143은 SIGTERM 처리시 사용됨
SuccessExitStatus=143

# 재시작 정책
Restart=on-failure
RestartSec=5

# 로그 설정 (journal 사용시)
StandardOutput=journal
StandardError=journal

# 로그 설정 (파일)
# StandardOutput=append:/var/log/myapp_stdout.log
# StandardError=append:/var/log/myapp_stderr.log

[Install]
WantedBy=multi-user.target
```

환경변수 파일을 별도로 지정하는 경우 다음과 같이 작성합니다.

```
# 환경 변수 예시
SPRING_PROFILES_ACTIVE=prod
JAVA_OPTS=-Xms512m -Xmx2g -Dfile.encoding=UTF-8
```

#### Python Application

```
[Unit]
Description=Python App - pyapp
After=network.target

[Service]
User=pyuser
WorkingDirectory=/opt/pyapp
ExecStart=/opt/pyapp/venv/bin/python /opt/pyapp/main.py
Restart=on-failure
RestartSec=5
StandardOutput=append:/var/log/pyapp.log
StandardError=append:/var/log/pyapp.log
Environment="PYTHONUNBUFFERED=1"

[Install]
WantedBy=multi-user.target
```

