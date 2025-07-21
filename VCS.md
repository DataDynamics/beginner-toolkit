# Version Control System

## Gitea

Gitea는 중국에서 만든 경량의 Github 같은 시스템입니다. 설치는 Docker가 가장 편리합니다.

다음과 같이 `docker-compose.yml` 파일을 작성합니다.

```yaml
version: "3"

networks:
  gitea:
    external: false

services:
  server:
    image: docker.gitea.com/gitea:1.24.3
    container_name: gitea
    environment:
      - USER_UID=1000
      - USER_GID=1000
      - GITEA__database__DB_TYPE=postgres
      - GITEA__database__HOST=db:5432
      - GITEA__database__NAME=gitea
      - GITEA__database__USER=gitea
      - GITEA__database__PASSWD=gitea
    restart: always
    networks:
      - gitea
    volumes:
      - ./gitea:/data
      - /etc/timezone:/etc/timezone:ro
      - /etc/localtime:/etc/localtime:ro
    ports:
      - "8080:3000"
      - "2221:22"

    depends_on:
      - db
  db:
    image: docker.io/library/postgres:14
    restart: always
    environment:
      - POSTGRES_USER=gitea
      - POSTGRES_PASSWORD=gitea
      - POSTGRES_DB=gitea
    networks:
      - gitea
    volumes:
      - ./postgres:/var/lib/postgresql/data
```

이제 다음의 커맨드로 Docker 컨테이너를 생성합니다.

```
# docker-compose up -d
```


