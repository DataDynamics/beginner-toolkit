# Ansible

* 다수의 서버를 작업해야 할 때 도움을 주는 자동화 도구
* SSH 접속이 가능해야 함

## Concept

* Inventory - 관리할 호스트 목록 정의 (/etc/ansible/hosts, 동적 inventory 가능)
* Playbook - 작업 절차 정의 파일. YAML 형식. 여러 작업(Tasks)을 순차 실행
* Task - 모듈을 이용해 하나의 작업 단위 실행
* Module - 작업을 수행하는 실행 단위. 예: `yum`, `copy`, `template`
* Role - 태스크, 핸들러, 변수, 템플릿 등을 구조화해서 재사용 가능하도록 구성
* Variable - `vars`, `group_vars`, `host_vars` 등에서 정의 가능
* Handler - 특정 변경이 있을 경우만 실행되는 Task
* Facts - 대상 시스템의 정보 (메모리, OS, 네트워크 등) 자동 수집 데이터

## Installation

```
pip3 install ansible
```

## Usage

### Hostfile

`/etc/ansible/hosts` 파일 또는 별도 호스트 파일을 작성해서 사용하도록 하며 가장 기본적인 포맷은 다음과 같음.

```
[master]
192.168.1.10
192.168.1.11
192.168.1.12

[utility]
192.168.1.20

[worker]
192.168.1.30
192.168.1.31
192.168.1.32
192.168.1.33
192.168.1.34
192.168.1.35

[nifi]
192.168.1.40
192.168.1.41
192.168.1.42
```

다음과 같이 옵션을 통해 좀더 디테일한 설정이 가능

```
[webservers]
web01 ansible_host=192.168.1.101
web02 ansible_host=192.168.1.102 ansible_user=deploy

[dbservers]
db01 ansible_host=192.168.1.201 ansible_port=2222 ansible_user=postgres
```

* `ansible_host` - 실제 접속할 IP/호스트
* `ansible_user` - 	SSH 접속 사용자
* `ansible_port` - SSH 포트 (기본: 22)
* `ansible_ssh_private_key_file` - 접속에 사용할 private key 경로
* `ansible_become` - root 권한 상승 여부 (`yes` 또는 `true`)
* `ansible_python_interpreter` - python 경로 명시 (e.g., `/usr/bin/python3`)

### 자주 사용하는 커맨드

* `ansible all -m ping`
* `anslble all -m shell -a "systemctl restart <서비스>"`
* `ansible all -m copy -a "src=/etc/hosts dest=/etc/hosts`
* `ansible all -m yum -a "name=httpd state=present" --become`
* `ansible all -m shell -a 'ssh-keygen -t rsa -b 4096 -N "" -f ~/.ssh/id_rsa <<< y' --become-user=$USER`
* `ansible all -m authorized_key -a "user=$USER key='{{ lookup(\"file\", \"$HOME/.ssh/id_rsa.pub\") }}'" --become-user=$USER`
