# DevOps 13주차 정리: GitOps, 배포 전략, 모니터링

---

## 1. CD 그리고 GitOps

### 핵심 개념 및 용어
* **GitOps (Git + Operations):** "Git이 곧 진실(Git as the Single Source of Truth)"이라는 원칙 아래, 클러스터에 있어야 할 **원하는 상태(Desired State)**를 Git에 선언해 두고 실제 클러스터 상태(Actual State)를 항상 Git과 일치하도록 운영하는 방식[cite: 32, 35, 37].
* **Sync (동기화):** Git의 선언 상태와 클러스터의 실제 상태를 같게 맞추는 작업[cite: 32].
* **Drift (표류):** 누군가 `kubectl scale`이나 `kubectl edit` 등으로 클러스터를 수동 조작하여 실제 상태가 Git의 선언에서 벗어난 상태 (`OutOfSync` 발생)[cite: 32, 38].
* **Self-heal (자가 치유):** Drift가 발생했을 때 Argo CD가 이를 감지하고 Git에 선언된 원래 상태로 자동 복구하는 기능[cite: 32, 38].
* **Prune (가지치기):** Git 저장소에서 리소스 YAML 파일을 삭제했을 때, 클러스터에 남아 있는 해당 오브젝트도 자동으로 삭제하는 기능[cite: 32, 54].

### Push 방식 vs Pull 방식 (GitOps)
| 구분 | Push 방식 (CI 직접 배포) | Pull 방식 (GitOps / Argo CD) |
| :--- | :--- | :--- |
| **클러스터 수정 주체** | 외부의 CI 서버 (GitHub Actions 등) | 클러스터 내부의 에이전트 (Argo CD) |
| **필요한 권한** | CI 서버에 클러스터 접속 권한 부여 필요 | 외부에 클러스터 권한을 줄 필요 없음 (최소 권한) |
| **방화벽/내부망 환경** | 외부에서 로컬/내부망 클러스터 접근 어려움 | 내부에서 외부(GitHub)를 조회하므로 접근 가능 |
| **배포 기록** | CI 파이프라인 로그를 별도로 관리 | Git 커밋 및 PR 히스토리가 곧 배포 기록 |
| **수동 변경(Drift) 대응** | 별도의 감지 및 복구 체계 구축 필요 | 자동 감지 후 `selfHeal`로 즉시 원복 |
| **롤백(Rollback)** | 이전 파이프라인 재실행 또는 `rollout undo` | `git revert` 후 PR 머지로 간단히 복구 |

### Argo CD 상태 읽는 법
* **Health 상태**
  * `Healthy`: Pod Ready 등 모든 오브젝트가 정상 작동 중[cite: 47].
  * `Progressing`: 롤링 업데이트 등 리소스가 변경 중인 상태[cite: 47].
  * `Degraded`: `ImagePullBackOff`, `CrashLoopBackOff` 등 Pod에 문제가 발생한 상태[cite: 47].
  * `Missing`: Git에는 선언되어 있으나 클러스터에 아직 생성되지 않은 상태[cite: 47].
* **Sync 상태**
  * `Synced`: 클러스터 상태와 Git 선언이 일치함[cite: 47].
  * `OutOfSync`: 클러스터 상태와 Git 선언이 다름 (동기화 대기 또는 자동 동기화 꺼짐)[cite: 47].
* **주의 조합 (`Synced` + `Degraded`):** Git에 적힌 대로 배포는 정확히 수행했으나, Git에 잘못된 내용(예: 오타가 있는 이미지 태그)을 적어 Pod가 뜨지 못하는 상태[cite: 47, 94].

### 주요 실습 명령어 요약
```bash
# 1. Argo CD 설치 및 접속
kubectl create namespace argocd
kubectl apply -n argocd --server-side --force-conflicts -f [https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml](https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml)
kubectl port-forward svc/argocd-server -n argocd 8443:443
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d ; echo

# 2. 애플리케이션 등록 및 확인 (guestbook/k8s 바깥 루트에 저장)
kubectl apply -f argocd-application.yaml
kubectl get application -n argocd

# 3. 롤백 (kubectl rollout undo 대신 git revert 사용)
git revert <커밋해시> --no-edit

# 4. 실습 정리 (애플리케이션 등록 해제)
kubectl delete application guestbook -n argocd
