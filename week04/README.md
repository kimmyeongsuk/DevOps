# 04. DevOps 네트워크 기초 및 문제 해결 (4주차)

# 1. 네트워크 핵심 개념

## IP 주소 및 NAT
- **IP 주소**: 네트워크상에서 장치를 식별하기 위한 주소 (건물 주소 역할)[cite: 108].
  - **공인 IP**: 인터넷상에서 유일한 식별 주소[cite: 108].
  - **사설 IP**: 내부 네트워크(가정/학교 등)에서 사용하는 주소 (`192.168.x.x`, `10.x.x.x`, `172.16~31.x.x`)[cite: 108].
  - **`127.0.0.1` (`localhost`)**: 자기 자신을 가리키는 루프백 주소[cite: 108].
- **NAT (Network Address Translation)**: 사설 IP를 공인 IP와 매핑하여 인터넷과 통신할 수 있도록 변환해주는 기술[cite: 109].

### 포트 (Port) & 주요 번호
- **포트**: 한 컴퓨터 내에서 동작하는 여러 서비스(프로세스)를 구분하는 번호 (호수 비유)[cite: 110].
- **주요 기본 포트**:
  - `22`: SSH (원격 접속)[cite: 111]
  - `80`: HTTP / `443`: HTTPS[cite: 111]
  - `3306`: MySQL / `6379`: Redis[cite: 111]
  - `5000`, `8080`: 개발용 웹 서버[cite: 111]

### DNS (Domain Name System)
- 사람이 읽기 쉬운 도메인 이름(예: `cs.inhatc.ac.kr`)을 컴퓨터가 이해하는 IP 주소(예: `221.154.90.200`)로 변환해주는 서비스[cite: 106, 115].

## 2. 네트워크 상태 조회 및 통신 명령

### 1) IP 및 연결 확인
# 내 사설/공인 IP 확인
hostname -I              # 사설 IP 확인
curl ifconfig.me         # 공인 IP 확인

# ICMP 패킷을 통한 대상 서버 통신 확인
ping -c 4 8.8.8.8
ping -c 4 cs.inhatc.ac.kr
> **주의**: 방화벽에서 ICMP(ping)를 차단하더라도 웹 서비스는 정상 작동 중일 수 있습니다[cite: 117].

### 2) DNS 조회 (`dig`, `nslookup`)
dig cs.inhatc.ac.kr +short   # 도메인의 IP 주소만 간결하게 조회
nslookup cs.inhatc.ac.kr      # DNS 정보 상세 조회

### 3) HTTP 요청 및 JSON 데이터 가공 (`curl`, `jq`)
curl -I [https://google.com](https://google.com)                       # HTTP 응답 헤더만 조회
curl -s [https://api.github.com/users/torvalds](https://api.github.com/users/torvalds) | jq -r '.name'  # JSON 데이터 파싱

## 3. 웹 서버 실행 및 포트 충돌 해결

### 1) Python HTTP 웹 서버 실행
mkdir -p ~/devops/week04/site
cd ~/devops/week04/site
python3 -m http.server 8080 --bind 0.0.0.0

- **`127.0.0.1`**: 로컬(내 컴퓨터) 접속만 허용[cite: 126].
- **`0.0.0.0`**: 모든 네트워크 인터페이스의 접속 요청을 대기 (LISTEN)[cite: 123, 126].

### 2) 대기 중인 TCP 포트 확인 (`ss`)

ss -tlnp                    # LISTEN 상태인 TCP 포트 및 프로세스 PID 확인
ss -tlnp | grep 8080        # 8080 포트 점유 여부 확인

### 3) 포트 충돌(`OSError: [Errno 98] Address already in use`) 해결
동일한 포트를 여러 프로세스가 동시에 사용할 수 없으므로 충돌이 발생합니다[cite: 128, 133].

1. **포트 번호 변경하여 실행**:
   python3 -m http.server 8000

2. **점유 중인 프로세스 종료**:
   # ss 명령어로 PID 확인 후 종료
   kill <PID>              # SIGTERM (정상 종료)
   pkill -f "http.server"   # 패턴 기반 프로세스 일괄 종료


##  4. 외부 접속 불가 시 점검 체크리스트
1. **LISTEN 상태 확인**: 서버 프로세스가 포트를 정상 개방했는지 확인 (`ss -tlnp`)[cite: 123].
2. **방화벽(Firewall) 확인**: 서버 포트가 켜져 있어도 방화벽 규칙에서 해당 포트(예: 8080)가 차단되어 있으면 접근 불가[cite: 132].
