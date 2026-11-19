# 9주차: 쿠버네티스

## kubectle get node 결과
<img width="1040" height="182" alt="스크린샷 2026-10-29 122228" src="https://github.com/user-attachments/assets/4cfe4413-3054-4e54-915e-cfd334dccf6c" />

## Pod를 지운 뒤 새 Pod가 태어난 get pods -w 화면 확인
<img width="1040" height="182" alt="2" src="https://github.com/user-attachments/assets/012bf068-7e40-4c81-948b-25a0b1f63b29" />


## 내 방명록이 쿠버네티스에서 뜬 브라우저 화면
<img width="1011" height="762" alt="image" src="https://github.com/user-attachments/assets/1d752ee4-7f93-45f0-a4c8-6b460a48036a" />


## "선언형"을 본인 언어로 한 줄(일상 속 예시 하나 포함)
- Pod 3개를 유지해라 처럼 원하는 최종 상태를 선언하면 쿠버네티스가 끊임없이 현재 상태를 감시하며 목표 상태로 자동으로 치유하는 방식

## 오늘 막혔던 것과 해결 방법 

- 발생한 문제: kubectl port-forward 중 address alreat in use 발생

- 해결 방법: 8080 포트를 점유 중인 프로세스를 종료하여 해결 


