1. docker compose ps 결과 (healthy 포함)

-----------------------------------------

docker compose ps 명령어를 통해 실행 중인 컨테이너들과 Healthcheck 상태(healthy)를 확인한 결과입니다.



$ docker compose ps

NAME                IMAGE           COMMAND                  SERVICE   CREATED        STATUS

guestbook-redis-1   redis:7-alpine  "docker-entrypoint.s..." redis     27 hours ago   Up 27 hours (healthy)

guestbook-web-1     guestbook-web   "python app.py"          web       10 seconds ago Up 10 seconds (healthy)

guestbook-web-2     guestbook-web   "python app.py"          web       10 seconds ago Up 10 seconds (healthy)

guestbook-web-3     guestbook-web   "python app.py"          web       10 seconds ago Up 10 seconds (healthy)





3. down 후 up과 down -v 후 up 차이 비교

-----------------------------------------

[1] docker compose down 후 up

- 볼륨 삭제 여부: 볼륨(redis-data)을 삭제하지 않고 유지함

- 데이터 보존: 기존 방명록 데이터가 그대로 보존됨

- 주요 용도: 컨테이너 설정 변경이나 코드 업데이트 시 데이터 유지



[2] docker compose down -v 후 up

- 볼륨 삭제 여부: -v 옵션에 의해 볼륨(redis-data)도 함께 삭제됨

- 데이터 보존: 기존 방명록 데이터가 모두 초기화(삭제)됨

- 주요 용도: 전체 환경을 완전히 초기화하고 다시 시작할 때 사용





4. .env를 Git에 올리지 않는 이유

-----------------------------------------

1) 보안 민감 정보 유출 방지 (Security)

   - .env 파일에는 REDIS_PASSWORD와 같은 데이터베이스 비밀번호, API 키, 시크릿 키 등 주요 보안 데이터가 저장됩니다.

   - 이를 Git 레포지토리에 올릴 경우 외부로 민감 정보가 유출되어 보안 사고로 이어질 수 있습니다.



2) 환경별 설정 분리 (Environment Isolation)

   - 개발(Development), 테스트(Testing), 운영(Production) 환경마다 사용하는 비밀번호, 포트, DB 주소 등이 서로 다릅니다.

   - 소스코드는 동일하게 유지하되, 각 실행 환경(서버)에 맞는 .env 파일을 개별 작성하여 관리하는 것이 표준 관례입니다.
