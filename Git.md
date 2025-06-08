# Git

## 사용자 정보 영구 저장

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
