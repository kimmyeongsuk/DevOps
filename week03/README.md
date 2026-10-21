# 03. 자동화와 협업 (4주차)
#  1. Linux 검색 및 자동화 기초

## 주요 명령어
- 파일 검색 (`find`):
  find -type f -name "*.sh"       # .sh 확장자 파일 검색
  
- 문자열 검색(grep):
  grep -R -n "exit 1" week04/    # 하위 디렉터리 내 키워드 검색 및 줄 번호 표시

- 파이프(|) 활용:
  grep "ERROR" server.log | wc -l # 에러 발생 건수 확인

# 2.프로세스 관리 및 자동 백업 스크립트

## 프로세스 명령어

-ps aux | grep <process_name>:
  실행 중인 프로세스 확인

- kill <PID> (SIGTERM/15) / kill -9 <PID> (SIGKILL/9):
   프로세스 정상/강제 종료
