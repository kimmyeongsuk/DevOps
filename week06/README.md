# 6주차 과제: Dockerfile & ghcr.io 배포


## 1. 내 이미지 주소
- `ghcr.io/kimmyeongsuk/guestbook:v2`

---

## 2. 친구 이미지 실행 결과

---<img width="882" height="593" alt="image" src="https://github.com/user-attachments/assets/5fb2fef8-6687-4dd9-b8a2-30be1d54a8b8" />


## 3. Dockerfile 명령어 설명

```dockerfile
FROM ghcr.io/kimmyeongsuk/guestbook:v1
ENV APP_TITLE="방 명 록" \
    THEME_COLOR="#7C9A82"

FROM: 컨테이너 빌드의 베이스가 될 기존 이미지를 지정
ENV : 컨테이너 환경에서 사용할 기본 환경 변수를 설정
THEME_COLOR로 테마 색상을 구워


## 4. 빌드 캐시 동작 로그(CACHED)

[+] Building 0.8s (5/5) FINISHED
 => [internal] load build definition from Dockerfile.custom                   0.0s
 => => transferring dockerfile: 142B                                         0.0s
 => [internal] load .dockerignore                                            0.0s
 => => transferring context: 2B                                              0.0s
 => [1/2] CACHED FROM ghcr.io/kimmyeongsuk/guestbook:v1                       0.0s
 => CACHED exporting to image                                                0.0s

## 5. 설정을 넣는 세가지 방법의 차이 요약


1.코드 (app.py)

  애플리케이션 소스코드 내부 변수에 값이 직접 하드코딩된 방식입니다.

  설정을 변경할 때마다 코드 수정 후 Docker 이미지를 매번 새로 빌드해야 하며, 우선순위가 가장 낮습니다.

2. ENV (Dockerfile)

  Dockerfile에 ENV 구문으로 기본 환경 변수 값을 정의하는 방식입니다.

  이미지를 생성할 때 기본값으로 구워지며, 소스코드를 변경하지 않고도 애플리케이션에 설정값을 전달할 수 있습니다.

3. -e (docker run 옵션)

  컨테이너를 실행할 때 docker run -e THEME_COLOR="#C6A15B"와 같이 동적으로 환경 변수를 주입하는 방식입니다. 

  코드나 Dockerfile의 ENV 설정보다 우선순위가 가장 높아 기존에 설정된 값을 덮어씁니
