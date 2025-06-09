# Git

## 사용자 정보 영구 저장

사용자 정보를 영구 저장하고, Git Push 등 수행시 재인증을 하지 않도록 하는 방법

```
git config --global user.name 홍길동
git config --global user.email honggildong@gmail.com
git config --global credential.helper store
```

## 초보자를 위한 가장 기본적인 커맨드

| 커맨드             | 설명                         | 예시                                         |
|-------------------|------------------------------|----------------------------------------------|
| `git clone <URL>` | Git Repository를 clone합니다. | `git clone https://github.com/user/repo.git` |
| `git add`         | 변경된 파일을 다음 커밋에 포함시키기 위해 스테이징 영역에 추가합니다. | `git add .` <br/>`git add -m README.md`|
| `git commit -m`   | 변경사항을 커밋합니다. | `git commit -m "Controller 추가"` |
| `git push`        | 커밋한 변경사항을 Git Repository에 push합니다. | `git push` |
| `git pull`        | 변경사항을 가져와서 병합니다. | `git pull` |
| `git remote -v`   | GIt Repository 정보를 표시합니다.| `git remote -v` |

## 관련 용어

| 용어                       | 설명                                                        |
| ------------------------ | --------------------------------------------------------- |
| **Repository (저장소)**     | Git으로 관리되는 프로젝트의 전체 이력 데이터. 로컬 저장소와 원격 저장소로 나뉨.           |
| **Working Directory**    | 사용자가 현재 작업 중인 디렉토리. 실제 파일이 위치함.                           |
| **Staging Area (Index)** | 커밋 대상이 될 파일들을 임시로 저장하는 영역. `git add` 명령으로 여기에 추가함.        |
| **Commit**               | Staging Area에 있는 변경사항을 저장소에 기록하는 명령. 하나의 스냅샷이라고 생각할 수 있음. |
| **Branch**               | 독립적인 작업 흐름을 의미하는 포인터. 기본 브랜치는 일반적으로 `main` 또는 `master`.   |
| **HEAD**                 | 현재 작업 중인 브랜치나 커밋을 가리키는 포인터.                               |
| **Merge**                | 한 브랜치의 변경사항을 다른 브랜치에 통합하는 작업.                             |
| **Rebase**               | 브랜치 기반을 다른 브랜치 위로 재설정하여 커밋 기록을 정리하는 작업.                   |
| **Clone**                | 원격 저장소를 복제하여 로컬 저장소를 생성하는 명령.                             |
| **Pull**                 | 원격 저장소의 변경사항을 로컬로 가져와 병합함. (`fetch` + `merge`)            |
| **Push**                 | 로컬 저장소의 변경사항을 원격 저장소에 업로드함.                               |
| **Fetch**                | 원격 저장소의 변경사항을 로컬로 가져오되, 자동 병합은 하지 않음.                     |
| **Remote**               | 원격 저장소의 URL 정보. 예: `origin`                               |
| **Tag**                  | 특정 커밋에 이름표를 붙이는 기능. 릴리스 버전 등에 사용.                         |
| **Checkout**             | 특정 브랜치나 커밋으로 작업 디렉토리를 전환함.                                |
| **Diff**                 | 변경사항을 비교하여 보여줌.                                           |
| **Reset**                | 커밋/스테이징/작업 디렉토리 상태를 이전으로 되돌림.                             |
| **Revert**               | 이전 커밋의 내용을 취소하는 새로운 커밋을 생성함.                              |
| **Cherry-pick**          | 다른 브랜치의 특정 커밋 하나를 현재 브랜치에 적용함.                            |
| **Conflict**             | 병합 또는 리베이스 중 자동 통합할 수 없는 충돌이 발생한 상태.                      |
| **Stash**                | 현재 작업 상태를 임시로 저장하고 워킹 디렉토리를 깨끗한 상태로 돌림.                   |

## Checkout

`git checkout`은 Git에서 브랜치 이동, 커밋/파일 복원, 작업 디렉토리 상태 변경 등 다양한 상황에서 사용하는 다기능 명령어입니다.

### 주요 기능

| 기능                      | 설명                                  |
| ----------------------- | ----------------------------------- |
| 브랜치 전환                  | 다른 브랜치로 이동하여 작업 디렉토리를 해당 브랜치 상태로 변경 |
| 브랜치 생성 후 전환             | 새 브랜치를 만들고 동시에 이동                   |
| 파일 복원                   | 특정 커밋에서 파일을 현재 작업 디렉토리로 되돌림 (취소)    |
| 커밋 체크아웃 (detached HEAD) | 브랜치가 아닌 커밋으로 이동 (읽기 전용 작업 등)        |

### 사용예

| 명령어                        | 설명                           |
| -------------------------- | ---------------------------- |
| `git checkout 브랜치명`        | 해당 브랜치로 이동                   |
| `git checkout -b 새브랜치명`    | 새 브랜치 생성 후 바로 이동             |
| `git checkout 커밋해시`        | 해당 커밋 상태로 이동 (detached HEAD) |
| `git checkout HEAD~1`      | 이전 커밋으로 이동                   |
| `git checkout -- 파일명`      | 파일을 마지막 커밋 상태로 복원 (변경사항 취소)  |
| `git checkout 커밋해시 -- 파일명` | 해당 커밋에서 특정 파일만 복원            |

```
# 브랜치 전환
$ git checkout develop

# 브랜치 생성과 동시에 전환
$ git checkout -b feature/new-api

# 특정 커밋에서 파일 복원
$ git checkout a1b2c3d -- src/app.py

# 현재 파일 변경사항 취소 (Staging 전)
$ git checkout -- README.md

# 이전 커밋으로 이동 (detached HEAD)
$ git checkout HEAD~1
```

### 주의사항

| 상황               | 주의할 점                                       |
| ---------------- | ------------------------------------------- |
| 변경사항 있을 때        | 변경사항이 커밋되거나 stash 되지 않으면 checkout이 실패할 수 있음 |
| detached HEAD    | 이 상태에서 작업하면 브랜치 없이 커밋이 존재함 → 브랜치 생성 필요      |
| `checkout -- 파일` | **주의!** 현재 변경된 내용을 되돌리므로 복구 불가할 수 있음        |

## Revert (되돌리기)

### 파일 삭제

Git으로 추적(관리)하고 있는 파일을 삭제하고, 그 삭제 기록을 다음 커밋에 반영하고 싶을 때 사용합니다.

```
# main.js 파일을 삭제하고, 삭제 내역을 스테이징
git rm main.js
git commit -m "Remove: 불필요한 main.js 파일 삭제"
```

패스워드 파일 등의 민감한 파일을 관리 대상에서 제외하되 현재 디렉토리는 그대로 유지합니다.

```
# config.env 파일을 Git 추적 대상에서만 제외 (파일은 남아있음)
git rm --cached config.env
```

### 변경 사항 되돌리기 (커밋하지 않은 변경)

특정 파일의 수정 사항을 마지막 커밋 상태로 완전히 되돌리고 싶을 때 사용합니다. 주의: 되돌린 내용은 복구할 수 없습니다.

```
# style.css 파일의 모든 수정 사항을 취소하고 마지막 커밋 상태로 복원
git checkout -- style.css

# 최신 Git에서는 restore를 권장
git restore style.css
```

### 커밋 되돌리기 (이미 커밋한 변경)

#### Git Reset

* 로컬 저장소의 커밋을 취소하고 싶을 때 사용합니다.
* 아직 원격 저장소(remote)에 push 하지 않은 커밋을 되돌릴 때 유용합니다.
* HEAD 포인터를 지정한 커밋으로 이동시키며, 옵션에 따라 스테이징 영역과 작업 디렉토리의 상태를 변경합니다.

```
# 가장 최신 커밋 1개를 취소하고, 해당 변경 내용은 커밋할 준비가 된 상태(staged)로 둠
git reset --soft HEAD~1

# 가장 최신 커밋 1개를 취소하고, 변경 내용은 unstaged 상태로 작업 디렉토리에 보존
git reset --mixed HEAD~1
# 또는 간단히
git reset HEAD~1

# 가장 최신 커밋 1개를 포함한 모든 변경 내용을 완전히 삭제하고 이전 커밋 상태로 복원
git reset --hard HEAD~1
```

#### Git Revert

* 원격 저장소에 이미 push 한 커밋의 변경 사항을 되돌리고 싶을 때 사용합니다.
* 협업 시 안전하게 커밋을 되돌리는 방법입니다.
* 지정한 커밋에서 이루어진 변경 사항을 거꾸로 수행하는 새로운 커밋을 생성하며 기존의 커밋 내역은 그대로 유지합니다.

```
# 특정 커밋(예: a1b2c3d)의 변경 내용을 되돌리는 새로운 커밋을 생성
git revert a1b2c3d

# revert 커밋 메시지를 편집기 없이 바로 생성
git revert --no-edit HEAD
```

### 요약

| 명령어	| 주 사용 상황 | 특징 | 
|-------|-------------|------|
| `git rm`	| Git으로 관리하는 파일 삭제	 | 실제 파일과 Git 추적을 동시에 제거 | 
| `git checkout -- <file>` <br/> `git restore <file>`	 | 커밋 전, 파일 수정 내용 취소	 | 마지막 커밋 상태로 파일 복원 (복구 불가) | 
| `git clean`	| 추적하지 않는(untracked) 파일 삭제	 | 빌드 산출물 등 정리 시 유용 | 
| `git reset`	| 로컬 커밋 되돌리기 (push 전)	 | 커밋 히스토리를 과거로 되돌림 (강제성) | 
| `git revert`	| 원격 커밋 되돌리기 (push 후)	 | 되돌리는 내용을 담은 새 커밋을 생성 (안전함) | 

## Stash

* stash는 작업 디렉토리와 스테이징 영역(index)의 변경사항을 임시 저장하는 공간입니다.
* 스택(stack) 구조로 작동하며, 여러 개의 stash를 저장할 수 있습니다.
* stash한 변경사항은 나중에 다시 꺼내어(복원하여) 작업을 계속할 수 있습니다.

### 명령어 정리

| 명령어                         | 설명                           |
| --------------------------- | ---------------------------- |
| `git stash`                 | 변경사항을 임시로 저장하고 워킹 디렉토리를 초기화  |
| `git stash save "메시지"`      | (구버전) 메시지를 포함하여 stash 저장     |
| `git stash push -m "메시지"`   | (신버전) 메시지를 포함하여 stash 저장     |
| `git stash list`            | 저장된 stash 목록 확인              |
| `git stash show`            | 최근 stash의 변경사항 요약 보기         |
| `git stash show -p`         | 최근 stash의 상세 diff 보기         |
| `git stash apply`           | 가장 최근 stash를 적용 (stash는 유지됨) |
| `git stash apply stash@{1}` | 특정 stash를 적용                 |
| `git stash pop`             | 가장 최근 stash를 적용하고 삭제         |
| `git stash drop stash@{0}`  | 특정 stash 항목 삭제               |
| `git stash clear`           | 모든 stash 항목 삭제               |

### 사용 예시

```
# 코드 변경 중 브랜치를 변경해야 할 때
$ git stash          # 변경사항 임시 저장
$ git checkout dev   # 다른 브랜치로 이동
$ git stash apply    # 작업 복원
```

### 옵션 정리

| 옵션                            | 설명                           |
| ----------------------------- | ---------------------------- |
| `-k` 또는 `--keep-index`        | 스테이징 영역은 유지하고 워킹 디렉토리만 stash |
| `-u` 또는 `--include-untracked` | 추적되지 않은 파일도 함께 stash         |
| `-a` 또는 `--all`               | 무시된 파일까지 모두 stash            |

### 주의사항

* stash는 어디까지나 임시 저장용입니다. 장기간 보관 용도는 적절하지 않습니다.
* stash apply는 내용을 유지하고 적용하고, stash pop은 내용을 적용 후 삭제합니다.
* 충돌(conflict)이 발생할 수 있으므로 적용 후 꼭 확인해야 합니다.














