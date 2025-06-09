# Subversion

Gitlab 등을 설치하기 곤란하고, 경량으로 버전을 관리하기 위해서 사용할 수 있는 대안이며 아주 오래된 VCS이므로 웬만하면 사용하지 않도록 합니다.

## 구성 방법

* Subversion Server - 독립 서버로 구성하며 구성하기 용이합니다.
* Apache HTTP + mod_dav_svn - Apache HTTP가 이미 구성되어 있고 80 포트가 개방되어 있을때 용이합니다.

## 설치

### Subversion Server

```
# 패키지 설치
dnf install subversion -y

# Subversion Repository 구성
mkdir -p /svn/repos
svnadmin create /svn/repos/myproject

# 서버 설정
vi /svn/repos/myproject/conf/svnserve.conf
[general]
anon-access = none
auth-access = write
password-db = passwd
authz-db = authz
realm = My SVN Repo

# 사용자 및 패스워드 설정
vi /svn/repos/myproject/conf/passwd
[users]
user1 = password1
user2 = password2

# 권한 설정
vi /svn/repos/myproject/conf/authz
[groups]
devs = user1, user2

[/]
* = 
@devs = rw
```

이제 systemd를 통해 관리할 수 있도록 `/etc/systemd/system/svnserve.service` 서비스 파일을 작성합니다.

```
[Unit]
Description=Subversion server
After=network.target

[Service]
Type=forking
ExecStart=/usr/bin/svnserve -d -r /svn/repos
ExecStop=/bin/kill -9 $MAINPID
PIDFile=/var/run/svnserve.pid

[Install]
WantedBy=multi-user.target
```

이제 다음의 커맨드로 서비스를 시작합니다. Subversion Server의 기본 포트는 3690 포트이므로 필요시 방화벽을 개방합니다.

```
systemctl daemon-reexec
systemctl daemon-reload
systemctl enable --now svnserve
```

이제 생성한 Subversion Repository를 다음의 커맨드로 checkout할 수 있습니다.

```
svn checkout svn://<서버IP>/svn/myproject --username user1
```

### Apache HTTP + mod_dav_svn

```
# 패키지 설치
dnf install subversion -y
dnf install httpd mod_dav_svn -y

# Subversion Repository 구성
mkdir -p /svn/repos
svnadmin create /svn/repos/myproject
chown -R apache:apache /svn/repos/myproject

# Apache HTTP 설정
sudo vi /etc/httpd/conf.d/subversion.conf
<Location /svn>
   DAV svn
   SVNParentPath /svn/repos
   AuthType Basic
   AuthName "Subversion Repository"
   AuthUserFile /etc/svn-auth-users
   Require valid-user
</Location>

# 로그인 사용자 추가
htpasswd -cm /etc/svn-auth-users user1
# 다음 사용자부터는 -c 빼고 추가
htpasswd -m /etc/svn-auth-users user2
```

이제 생성한 Subversion Repository를 다음의 커맨드로 checkout할 수 있습니다.

```
svn checkout http://<서버IP>/svn/myproject --username user1
```

## Subversion Repository 구조

```
<ROOT>
   +-- branches
   +-- tags
   +-- trunk
```

## 커맨드 요약

| 구분       | 명령어                            | 설명                   | 예시                                     |
| -------- | ------------------------------ | -------------------- | -------------------------------------- |
| 🧱 초기화   | `svnadmin create`              | 저장소 생성 (서버 측)        | `svnadmin create /svn/repos/myproject` |
| 📥 체크아웃  | `svn checkout` (`co`)          | 저장소 복사 (로컬 워킹카피 생성)  | `svn checkout svn://host/myproject`    |
| 📤 커밋    | `svn commit` (`ci`)            | 변경사항을 저장소에 반영        | `svn commit -m "설명"`                   |
| 📄 추가    | `svn add`                      | 신규 파일/디렉토리 추가        | `svn add newfile.txt`                  |
| ❌ 삭제     | `svn delete` (`del`, `remove`) | 파일/디렉토리 삭제           | `svn delete file.txt`                  |
| 📝 수정 확인 | `svn status` (`st`)            | 변경사항 요약 보기           | `svn status`                           |
| 🔍 차이 확인 | `svn diff`                     | 변경된 내용 비교            | `svn diff file.txt`                    |
| 🔄 업데이트  | `svn update` (`up`)            | 저장소 변경사항을 로컬에 반영     | `svn update`                           |
| 🔁 되돌리기  | `svn revert`                   | 로컬 변경사항 되돌림          | `svn revert file.txt`                  |
| 📜 로그    | `svn log`                      | 커밋 기록 확인             | `svn log file.txt`                     |
| 📦 정보    | `svn info`                     | 파일 또는 워킹카피 정보        | `svn info`                             |
| 📑 목록    | `svn list` (`ls`)              | 저장소 내 파일 목록 확인       | `svn list svn://host/myproject`        |
| 🔍 검사    | `svnlook`                      | 저장소 내용을 직접 조회 (서버 측) | `svnlook tree /svn/repos/myproject`    |

