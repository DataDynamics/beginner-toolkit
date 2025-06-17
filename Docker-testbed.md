# Docker Testbed

## Apache Ranger

Apache Ranger를 실행하기 위해서 다음과 같이 `docker-compose.yml` 파일을 작성합니다.

```yaml
version: '3.8'

services:
  ranger-zk:
    image: apache/ranger-zk:2.6.0
    container_name: ranger-zk
    hostname: localhost
    ports:
      - "2181:2181"
    networks:
      - rangernw

  ranger-solr:
    image: apache/ranger-solr:2.6.0
    container_name: ranger-solr
    hostname: localhost
    command: solr-precreate ranger_audits /opt/solr/server/solr/configsets/ranger_audits/
    ports:
      - "8983:8983"
    networks:
      - rangernw

  ranger-db:
    image: apache/ranger-db:2.6.0
    container_name: ranger-db
    hostname: localhost
    ports:
      - "5432:5432"
    networks:
      - rangernw
    healthcheck:
      test: su -c "pg_isready -q" postgres
      interval: 10s
      timeout: 2s
      retries: 30

  ranger:
    image: apache/ranger:2.6.0
    container_name: ranger
    hostname: localhost
    ports:
      - "6080:6080"
    environment:
      RANGER_VERSION: "2.6.0"
      RANGER_DB_TYPE: "postgres"
    networks:
      - rangernw
    command: /home/ranger/scripts/ranger.sh
    depends_on:
      ranger-db:
        condition: service_healthy
      ranger-zk:
        condition: service_started
      ranger-solr:
        condition: service_started

networks:
  rangernw:
    name: rangernw
```

`docker-compose`로 다음과 같이 실행합니다. macOS에서 `docker-compose`가 없다면 `brew install docker-compose`를 실행하여 설치합니다.

```
docker-compose up -d
```

이제 웹 브라우저로 http://localhost:6080/에 접속합니다. 초기 접속에 필요한 username, password는 다음과 같습니다.

* Username: admin
* Password: rangerR0cks!

## DeltaLake

|CPU|Run Command|
|--|--|
|Intel|`docker run --name delta_quickstart --rm -it -p 8888:8888 --entrypoint bash deltaio/delta-docker:latest`|
|mac M1|`docker run --name delta_quickstart --rm -it -p 8888:8888 --entrypoint bash deltaio/delta-docker:latest_arm64`|

Shell로 진입시 `startup.sh`를 실행하면 다음과 같이 로그가 출력되고 8888 포트로 접속하도록 함

```
[I 2025-06-17 23:47:21.252 ServerApp] Package jupyterlab took 0.0000s to import
[I 2025-06-17 23:47:21.253 ServerApp] Package jupyter_server_fileid took 0.0008s to import
[I 2025-06-17 23:47:21.255 ServerApp] Package jupyter_server_terminals took 0.0016s to import
[I 2025-06-17 23:47:21.261 ServerApp] Package jupyter_server_ydoc took 0.0060s to import
[I 2025-06-17 23:47:21.261 ServerApp] Package nbclassic took 0.0000s to import
[W 2025-06-17 23:47:21.262 ServerApp] A `_jupyter_server_extension_points` function was not found in nbclassic. Instead, a `_jupyter_server_extension_paths` function was found and will be used for now. This function name will be deprecated in future releases of Jupyter Server.
[I 2025-06-17 23:47:21.262 ServerApp] Package notebook_shim took 0.0000s to import
[W 2025-06-17 23:47:21.262 ServerApp] A `_jupyter_server_extension_points` function was not found in notebook_shim. Instead, a `_jupyter_server_extension_paths` function was found and will be used for now. This function name will be deprecated in future releases of Jupyter Server.
[I 2025-06-17 23:47:21.264 ServerApp] jupyter_server_fileid | extension was successfully linked.
[I 2025-06-17 23:47:21.265 ServerApp] jupyter_server_terminals | extension was successfully linked.
[I 2025-06-17 23:47:21.266 ServerApp] jupyter_server_ydoc | extension was successfully linked.
[I 2025-06-17 23:47:21.267 ServerApp] jupyterlab | extension was successfully linked.
[I 2025-06-17 23:47:21.268 ServerApp] nbclassic | extension was successfully linked.
[I 2025-06-17 23:47:21.270 ServerApp] Writing Jupyter server cookie secret to /home/NBuser/.local/share/jupyter/runtime/jupyter_cookie_secret
[I 2025-06-17 23:47:21.338 ServerApp] notebook_shim | extension was successfully linked.
[I 2025-06-17 23:47:21.373 ServerApp] notebook_shim | extension was successfully loaded.
[I 2025-06-17 23:47:21.373 FileIdExtension] Configured File ID manager: ArbitraryFileIdManager
[I 2025-06-17 23:47:21.373 FileIdExtension] ArbitraryFileIdManager : Configured root dir: /opt/spark/work-dir
[I 2025-06-17 23:47:21.373 FileIdExtension] ArbitraryFileIdManager : Configured database path: /home/NBuser/.local/share/jupyter/file_id_manager.db
[I 2025-06-17 23:47:21.373 FileIdExtension] ArbitraryFileIdManager : Successfully connected to database file.
[I 2025-06-17 23:47:21.373 FileIdExtension] ArbitraryFileIdManager : Creating File ID tables and indices with journal_mode = DELETE
[I 2025-06-17 23:47:21.377 FileIdExtension] Attached event listeners.
[I 2025-06-17 23:47:21.377 ServerApp] jupyter_server_fileid | extension was successfully loaded.
[I 2025-06-17 23:47:21.377 ServerApp] jupyter_server_terminals | extension was successfully loaded.
[I 2025-06-17 23:47:21.377 ServerApp] jupyter_server_ydoc | extension was successfully loaded.
[I 2025-06-17 23:47:21.378 LabApp] JupyterLab extension loaded from /usr/local/lib/python3.9/dist-packages/jupyterlab
[I 2025-06-17 23:47:21.378 LabApp] JupyterLab application directory is /usr/local/share/jupyter/lab
[I 2025-06-17 23:47:21.379 ServerApp] jupyterlab | extension was successfully loaded.

  _   _          _      _
 | | | |_ __  __| |__ _| |_ ___
 | |_| | '_ \/ _` / _` |  _/ -_)
  \___/| .__/\__,_\__,_|\__\___|
       |_|
                                                                           
Read the migration plan to Notebook 7 to learn about the new features and the actions to take if you are using extensions.

https://jupyter-notebook.readthedocs.io/en/latest/migrate_to_notebook7.html

Please note that updating to Notebook 7 might break some of your extensions.

[I 2025-06-17 23:47:21.380 ServerApp] nbclassic | extension was successfully loaded.
[I 2025-06-17 23:47:21.380 ServerApp] Serving notebooks from local directory: /opt/spark/work-dir
[I 2025-06-17 23:47:21.380 ServerApp] Jupyter Server 2.5.0 is running at:
[I 2025-06-17 23:47:21.380 ServerApp] http://5bd5f5dd37c5:8888/lab?token=d4ed14e9afb1860c3f145c8e8d23661f630aeb70fda15845
[I 2025-06-17 23:47:21.380 ServerApp]     http://127.0.0.1:8888/lab?token=d4ed14e9afb1860c3f145c8e8d23661f630aeb70fda15845
[I 2025-06-17 23:47:21.380 ServerApp] Use Control-C to stop this server and shut down all kernels (twice to skip confirmation).
[W 2025-06-17 23:47:21.382 ServerApp] No web browser found: Error('could not locate runnable browser').
[C 2025-06-17 23:47:21.382 ServerApp] 
    
    To access the server, open this file in a browser:
        file:///home/NBuser/.local/share/jupyter/runtime/jpserver-9-open.html
    Or copy and paste one of these URLs:
        http://5bd5f5dd37c5:8888/lab?token=d4ed14e9afb1860c3f145c8e8d23661f630aeb70fda15845
        http://127.0.0.1:8888/lab?token=d4ed14e9afb1860c3f145c8e8d23661f630aeb70fda15845
```
