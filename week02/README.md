# 02. 셸 스크립팅과 로컬 AI 서비스 자동화 (3주차)

## 디렉터리 구조
```text
~/devops/
└── week03/
    └── qwen-web/
        ├── chat.py      # Python 웹 서버 스크립트 (8000/8080 포트)
        ├── start.sh     # 메인 실행 셸 스크립트
        └── README.md    # 주차별 학습 내용 및 실습 기록
```[cite: 14, 22]


## 실습 환경 구성 및 준비

### 1. Ollama 및 Qwen 모델 다운로드
- **Ollama 설치**:
  ```bash
  curl -fsSL [https://ollama.com/install.sh](https://ollama.com/install.sh) | sh
  ```[cite: 15]
- **Qwen3 (0.6B) 모델 다운로드 및 확인**:
  ```bash
  ollama pull qwen3:0.6b
  ollama list
  ```[cite: 18]
- **API 동작 검증**:
  ```bash
  curl -fsS [http://127.0.0.1:11434/api/tags](http://127.0.0.1:11434/api/tags)
  ```[cite: 17]

---

## 셸 스크립트 작성 및 핵심 개념 설명

### 1. `start.sh` 스크립트
```bash
#!/bin/bash

### 1. 스크립트가 위치한 디렉터리로 절대 경로 이동
cd "$(dirname "$0")" || exit 1

### 2. Python3 설치 여부 검사 (조건문 및 리다이렉션)
if ! command -v python3 >/dev/null 2>&1; then
    echo "Error: Python3가 설치되어 있지 않습니다." >&2
    exit 1
fi

### 3. 애플리케이션 실행 (프로세스 대체)
exec python3 chat.py
```[cite: 33, 41, 48]

---

## 셸 스크립트 핵심 개념 정리

1. **Shebang (`#!/bin/bash`)**
   - 스크립트 맨 첫 줄에 작성하여 해당 파일을 해석할 인터프리터를 지정합니다[cite: 34].
2. **실행 권한 부여 (`chmod u+x`)**
   - 기본 생성 파일의 `Permission denied` 에러를 방지하기 위해 사용자 실행 권한을 부여합니다[cite: 7, 42].
     ```bash
     chmod u+x start.sh
     ```[cite: 7, 42]
3. **경로 처리 (`cd "$(dirname "$0")"`)**
   - `$0`은 실행된 스크립트 경로[cite: 36, 37], `dirname`은 디렉터리 경로만 추출[cite: 36].
   - 어디서 스크립트를 호출하든 파일이 존재하는 위치 기준 상대 경로 문제를 방지합니다[cite: 33, 36].
4. **논리 연산자 및 종료 코드 (`|| exit 1`)**
   - `||` (OR) 연산자는 앞 명령이 실패했을 때 뒤 명령을 실행합니다[cite: 40].
   - `exit 1`을 통해 오류 상태 코드(`1`)를 남기며 프로세스를 안전하게 종료합니다[cite: 40, 54].
5. **표준 출력 및 리다이렉션 (`>/dev/null 2>&1`)**
   - `1`: 표준 출력(stdout), `2`: 표준 오류(stderr)[cite: 52].
   - `>/dev/null`: 출력을 버림[cite: 51], `2>&1`: 표준 오류를 표준 출력 방향으로 통합하여 화면에 메시지를 숨깁니다[cite: 52].
6. **`exec` 명령어**
   - 새로운 프로세스를 띄우지 않고 현재 Bash 프로세스를 `python3` 프로세스로 대체하여 PID를 유지합니다[cite: 41].

---

## 환경 변수 활용 및 테스트

`export` 명령을 사용하여 원래 터미널의 환경 변수는 건드리지 않고, 실행 시점에 동적으로 포트 및 모델을 변경할 수 있습니다[cite: 43, 46].

### `start_with_export.sh` (Heredoc 작성 예시)
```bash
cat > start_with_export_2.sh << 'EOF'
#!/bin/bash
cd "$(dirname "$0")" || exit 1
export MODEL="qwen3:0.6b"
export WEB_PORT="8080"
exec ./start.sh
EOF

chmod u+x start_with_export_2.sh
./start_with_export_2.sh
```[cite: 43, 44, 46]

---
