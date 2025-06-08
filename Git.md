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
| `git commit -m`   | 변경사항을 커밋합니다. | `git commit -m "Controller 추가"` |
| `git push`        | 커밋한 변경사항을 Git Repository에 push합니다. | `git push` |
| `git pull`        | 변경사항을 가져와서 병합니다. | `git pull` |
| `git remote -v`   | GIt Repository 정보를 표시합니다.| `git remote -v` |

## 되돌리기

### 파일 삭제

Git으로 추적(관리)하고 있는 파일을 삭제하고, 그 삭제 기록을 다음 커밋에 반영하고 싶을 때 사용

```
# main.js 파일을 삭제하고, 삭제 내역을 스테이징
git rm main.js
git commit -m "Remove: 불필요한 main.js 파일 삭제"
```

패스워드 파일 등의 민감한 파일을 관리 대상에서 제외하되 현재 디렉토리는 그대로 유지

```
# config.env 파일을 Git 추적 대상에서만 제외 (파일은 남아있음)
git rm --cached config.env
```

### 변경 사항 되돌리기 (커밋하지 않은 변경)

특정 파일의 수정 사항을 마지막 커밋 상태로 완전히 되돌리고 싶을 때 사용합니다. 주의: 되돌린 내용은 복구할 수 없음

```
# style.css 파일의 모든 수정 사항을 취소하고 마지막 커밋 상태로 복원
git checkout -- style.css

# 최신 Git에서는 restore를 권장
git restore style.css
```

### 커밋 되돌리기 (이미 커밋한 변경)

#### Git Reset

* 로컬 저장소의 커밋을 취소하고 싶을 때 사용
* 아직 원격 저장소(remote)에 push 하지 않은 커밋을 되돌릴 때 유용
* HEAD 포인터를 지정한 커밋으로 이동시키며, 옵션에 따라 스테이징 영역과 작업 디렉토리의 상태를 변경

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

* 원격 저장소에 이미 push 한 커밋의 변경 사항을 되돌리고 싶을 때 사용
* 협업 시 안전하게 커밋을 되돌리는 방법
* 지정한 커밋에서 이루어진 변경 사항을 거꾸로 수행하는 새로운 커밋을 생성하며 기존의 커밋 내역은 그대로 유지

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
| `git checkout -- <file>` `git restore <file>`	 | 커밋 전, 파일 수정 내용 취소	 | 마지막 커밋 상태로 파일 복원 (복구 불가) | 
| `git clean`	| 추적하지 않는(untracked) 파일 삭제	 | 빌드 산출물 등 정리 시 유용 | 
| `git reset`	| 로컬 커밋 되돌리기 (push 전)	 | 커밋 히스토리를 과거로 되돌림 (강제성) | 
| `git revert`	| 원격 커밋 되돌리기 (push 후)	 | 되돌리는 내용을 담은 새 커밋을 생성 (안전함) | 



