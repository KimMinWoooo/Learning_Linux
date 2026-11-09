# 🛠️ Linux Production Troubleshooting: Shell Command Master Guide

> **실제 리눅스 VM 운영 환경에서 빈번하게 발생하는 '디스크 고갈(Disk Full)'과 'CPU 폭주(CPU 100%)' 장애 상황을 인위적으로 시뮬레이션하고, 표준 엔지니어링 절차(SOP)에 따라 문제를 진단 및 복구하는 트러블슈팅 실습 프로젝트**

---

## 📋 목차
1. [핵심 쉘 커맨드 10선](#-1-실무-핵심-쉘-커맨드-10선)
2. [시나리오 A: 디스크 풀(Full) 긴급 복구 및 대용량 파일 색출](#-시나리오-a-디스크-풀full-긴급-복구-및-대용량-파일-색출)
3. [시나리오 B: CPU 폭주 프로세스 탐지 및 시그널 제어](#-시나리오-b-cpu-폭주-프로세스-탐지-및-시그널-제어)

---

## 📌 1. 핵심 쉘 커맨드 10선

| 번호 | 명령어 | 주요 역할 | 실무 추천 옵션 및 예시 | 실무 활용 상황 |
| :---: | :--- | :--- | :--- | :--- |
| **1** | `ls` | 파일/디렉터리 목록 조회 | `ls -lh`, `ls -lart` | 파일 크기(MB/GB) 확인, 최근 수정된 파일 맨 아래 정렬 |
| **2** | `tail` | 파일 끝부분 확인 / 스트리밍 | `tail -n 50 app.log`, `tail -f app.log` | 실시간 에러 로그 모니터링, 최근 N줄 확인 |
| **3** | `grep` | 텍스트 패턴 검색 | `grep -i "error" server.log`, `grep -rn "PORT" /etc/` | 로그 내 에러 키워드 추출, 특정 단어가 있는 파일 위치 역추적 |
| **4** | `find` | 파일 시스템 검색 및 작업 연계 | `find /var/log -type f -name "*.log"`, `find . -size +100M -exec ls -lh {} +` | 100MB 이상 대용량 파일 색출, 특정 확장자 일괄 작업 |
| **5** | `df` | 전체 디스크 파티션 여유 공간 점검 | `df -h` | 디스크 사용률(100% 임박 여부) 1초 만에 확인 |
| **6** | `du` | 특정 디렉터리/파일 실제 용량 측정 | `du -sh /var/log`, `du -h --max-depth=1 /var \| sort -hr` | 디스크 용량을 갉아먹는 범인 폴더 순위 매기기 |
| **7** | `ps` | 실행 중인 프로세스 목록 확인 | `ps aux`, `ps aux \| grep nginx` | 서비스 실행 여부 점검, 특정 프로세스의 PID 확인 |
| **8** | `kill` | 프로세스에 종료 시그널 전달 | `kill -15 <PID>` (정상 종료), `kill -9 <PID>` (강제 종료) | 먹통이 된 프로세스 안전하게 종료 또는 강제 사살 |
| **9** | `ss` | 네트워크 포트 및 소켓 상태 확인 | `ss -tulpn` | "Port already in use" 에러 해결, 80/8080 포트 점유자 확인 |
| **10** | `curl` | 터미널 네트워크/HTTP 요청 | `curl -I http://localhost:8080` | 웹 브라우저 없이 API 헬스체크 및 HTTP 응답 코드 확인 |

---

## 🚨 시나리오 A: 디스크 풀(Full) 긴급 복구 및 대용량 파일 색출

* **장애 상황**: 루트(`/`) 파티션 사용률 급증으로 서비스 다운 및 DB 쓰기 불가 위기 감지.
* **복구 절차**: `df -h` (파티션 점검) → `du` + `sort` (디렉터리 추적) → `find` (대용량 파일 색출) → `rm` (안전 삭제) → `df -h` (정상화 검증)

실습 노션 : https://wonderful-morocco-85b.notion.site/A-Full-413232206759809aa62de9d7d1ff5bf4?source=copy_link

---

## 🚨 시나리오 B: CPU 폭주 프로세스 탐지 및 시그널 제어
* **장애 상황**: 서버의 응답 속도가 현저히 느려지고, CPU 사용률이 100%에 도달하여 팬 소음 및 다른 서비스 지연 발생.
* **해결 절차(SOP)**:
  1. `uptime`: 시스템 전체 평균 부하(Load Average) 확인
  2. `top`: 실시간으로 CPU를 가장 많이 점유하는 프로세스(PID) 식별
  3. `ps aux`: 해당 PID의 전체 실행 명령어 및 부모/자식 관계 상세 분석
  4. `kill -15` (SIGTERM): 프로세스에 안전한 종료 요청(Graceful Shutdown)
  5. `kill -9` (SIGKILL): 응답이 없을 경우 OS 커널 레벨 강제 사살
  6. `top` & `ps`: CPU 정상화 및 프로세스 완전 소멸 검증

  실습 노션 : https://wonderful-morocco-85b.notion.site/B-CPU-4132322067598049b97ecadf907c088c?source=copy_link