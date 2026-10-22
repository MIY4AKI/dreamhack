# Dreamhack Linux 201 — 전체 학습 노트

> 원 Path: [Linux 201](https://dreamhack.io/lecture/paths/linux-201) · 확인일: 2026-09-30
> 저장 위치: `C:\dreamhack\Linux201` · 강의 **7개, 42페이지** + Network Lab **2과제** + 방화벽 기초 퀴즈 **7문항**.

Linux 101의 기본 명령을 바탕으로 패키지 관리, 네트워크, Vim, 방화벽과 Docker까지 공부하는 한국어 학습 노트다. 시스템·웹 해킹, 포렌식, 클라우드 보안 실습에서 반복해서 사용하는 환경 관리 능력을 익히는 과정이다.

## 읽는 순서

1. [APT 패키지 관리](01_apt_package_management.md) — 5페이지. 패키지·의존성, apt와 apt-get, 설치·업데이트·삭제·조회·저장소.
2. [네트워크](02_network.md) — 9페이지. OSI·TCP/IP, 인터페이스, ifconfig, nc·ssh·wget·curl과 옵션.
3. [편집기 — Vim](03_vim_editor.md) — 9페이지. vi와 Vim, 모드 전환, 편집·이동·복사·검색·저장·종료.
4. [nft 방화벽](04_nft_firewall.md) — 5페이지. netfilter와 도구 계층, table·chain·rule, 허용·차단·기본 정책·영속화.
5. [iptables 방화벽](05_iptables_firewall.md) — 4페이지. nft 호환 backend, 규칙 조회·추가·삭제, 기본 차단 정책과 필수 허용 규칙.
6. [ufw 방화벽](06_ufw_firewall.md) — 6페이지. 간편 규칙·기본 정책, 활성화·상태 확인·삭제·로그, Lab 안내.
7. [방화벽 고급 설정과 Docker](07_advanced_firewall_and_docker.md) — 4페이지. 출발지 IP·속도 제한·인터페이스·NAT, Docker 공개 포트와 ufw/FORWARD 경로.
8. [Lab과 방화벽 기초 퀴즈](08_labs_and_quiz.md) — Network Lab 2과제, Pro Lab 접근 상태, 퀴즈 7문항의 학습 포인트와 해설.

사이트 구성은 4 Unit(APT 패키지 관리, 네트워크 관리, Vim, 방화벽), 7 강의, 2 Lab, 1 퀴즈다. 안내 학습 시간은 약 3시간 30분이며 기본 컴퓨터 사용과 리눅스 기본 명령에 대한 이해가 선수 지식이다.

## 확인 범위와 남은 항목

- 7개 일반 강의의 모든 페이지와 제공된 코드·옵션·주요 그림 설명을 확인했다.
- Network Lab의 과제 2개와 방화벽 퀴즈의 문항·선택지 7개를 읽었다. 답을 제출하거나 채점 결과를 확인한 것은 아니다.
- **‘방화벽 구성해보기’ Pro Lab 본문은 현재 접근 불가로 미확인이다.** 과제 조건이나 해설을 추측해 만들지 않았다. 사용자에게 해당 페이지를 열어 달라고 요청한 상태이며, 확인되면 Lab 노트에 추가한다.
- 일반 강의의 원문을 그대로 복제하지 않고 학습용 요약으로 재구성했다. 반복 로그·후기·개인 진도 등은 제외했다.
- 구현마다 다른 nc·ps 옵션, curl 헤더 옵션 등 부정확할 수 있는 부분은 공식 문서와 대조하여 보충·정정했다.

## 실습 시 중요한 주의

노트는 Linux 명령을 설명하는 문서이며 사용자 Windows나 운영 서버를 변경하지 않았다. 실제 실습 시에는 다음을 구별한다.

- 원격 접속 중 기본 정책을 DROP으로 바꾸면 SSH가 끊길 수 있다.
- 규칙을 지우는 것과 기본 정책을 바꾸는 것은 다르다.
- IPv4·IPv6와 backend, nft family의 적용 범위를 확인한다.
- 실행 중 규칙과 재부팅 후 유지 설정은 다르다.
- Docker 공개 포트를 ufw의 일반 INPUT 규칙만으로 보호한다고 가정하지 않는다.
- 연결·다운로드 예시는 허가된 대상에만 실행하고, 민감한 파일·토큰을 외부로 보내지 않는다.

## 원 강의 바로가기

- [APT /730](https://learn.dreamhack.io/730)
- [네트워크 /731](https://learn.dreamhack.io/731)
- [Vim /732](https://learn.dreamhack.io/732)
- [nft /918](https://learn.dreamhack.io/918)
- [iptables /919](https://learn.dreamhack.io/919)
- [ufw /920](https://learn.dreamhack.io/920)
- [고급 설정·Docker /921](https://learn.dreamhack.io/921)

---

이전 과정: [Linux 101 전체 학습 노트](../Linux101/README.md)
