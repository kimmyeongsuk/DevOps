# DevOps 13주차 전체 정리: GitOps, 배포 전략, 모니터링

---

## 1. CD 그리고 GitOps

### 1.1 핵심 개념 및 용어
* **GitOps (Git + Operations):** "Git이 곧 진실(Git as the Single Source of Truth)"이라는 원칙 아래, 클러스터에 있어야 할 **원하는 상태(Desired State)**를 Git에 선언해 두고 클러스터의 **현재 상태(Actual State)**를 항상 Git과 일치하도록 운영하는 방식
* **Sync (동기화):** Git의 선언 상태와 클러스터의 실제 상태를 같게 맞추는 과정
* **Drift (표류):** 급한 수정(`kubectl edit`, `kubectl scale`)이나 수동 변경으로 인해 클러스터의 실제 상태가 Git의 선언에서 벗어난 상태 (`OutOfSync` 발생 — 장애의 흔한 원인)
* **Self-heal (자가 치유):** Drift가 발생했을 때 Argo CD가 이를 감지하고 Git에 선언된 원래 상태로 자동 복구하는 기능
* **Prune (가지치기):** 관리하던 리소스가 Git 저장소에서 삭제되면 클러스터에서도 해당 리소스를 자동으로 제거하는 기능
* **Argo:** 그리스 신화에 나오는 배 이름에서 유래[cite: 32, 35, 37, 38]

### 1.2 Push 방식 vs Pull 방식 (GitOps)
| 구분 | Push 방식 (CI 직접 배포) | Pull 방식 (GitOps / Argo CD) |
| :--- | :--- | :--- |
| **클러스터 수정 주체** | 바깥의 CI 서버 (GitHub Actions 등) | 클러스터 안의 Argo CD 에이전트 |
| **필요한 권한** | CI 서버에 클러스터 접속 권한을 줘야 함 | 바깥에 줄 클러스터 권한 없음 (최소 권한) |
| **방화벽 뒤 클러스터** | 외부에서 직접 접속할 수 없음 | 안에서 밖(GitHub)으로 나가므로 접근 가능 |
| **배포 기록** | 파이프라인 로그를 따로 관리해야 함 | Git 히스토리(커밋·PR)가 곧 배포 기록 |
| **누가 손으로 수정 시** | 별도의 변경 감지 및 복구 체계 필요 | 자동 감지 후 원래 상태로 되돌림 (`selfHeal`) |
| **되돌리기 (Rollback)** | 이전 파이프라인 재실행 또는 `rollout undo` | `git revert`로 간단하게 복구[cite: 34, 35, 36] |

### 1.3 Argo CD 설치 및 접속
```bash
# [터미널 1] 클러스터 및 기본 앱 실행
minikube start --driver=docker
kubectl apply -f guestbook/k8s
minikube service guestbook

# [터미널 2] Argo CD 네임스페이스 생성 및 설치 (stable 버전)
kubectl create namespace argocd
kubectl apply -n argocd --server-side --force-conflicts -f \
  [https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml](https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml)
kubectl get pods -n argocd -w   # 모든 Pod가 Running(1~3분 소요) 될 때까지 대기

# [터미널 3] 포트 포워딩 (HTTPS 443 포트로 연결)
kubectl port-forward svc/argocd-server -n argocd 8443:443

# [터미널 4] 초기 admin 비밀번호 확인
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
