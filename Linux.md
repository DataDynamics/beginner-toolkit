# Linux

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
