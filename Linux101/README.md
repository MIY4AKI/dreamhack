# Dreamhack Linux 101 — 전체 학습 노트

> 원 Path: [Linux 101](https://dreamhack.io/lecture/paths/linux-101) · 확인일: 2026-09-30
> 저장 위치: `C:\dreamhack\Linux101` · 강의 **8개, 108페이지** + Lab **5개, 24과제** + 워게임 안내 **1개**.

리눅스 환경 구축부터 셸, 파일·디렉터리, 프로세스, 환경 변수, 입출력, 사용자·권한까지 순서대로 공부할 수 있는 한국어 학습 노트다. 보안 실습에서 리눅스 조작 때문에 학습이 끊기지 않도록 기본기를 익히는 것이 Path의 목표다.

## 읽는 순서

1. [리눅스 환경 구축하기](01_linux_environment_setup.md) — 22페이지. VM·스냅샷, Apple Silicon의 UTM, Windows VirtualBox, Ubuntu 설치, SSH·Guest Additions·공유, WSL2.
2. [리눅스 사용법 한 눈에 살펴보기](02_linux_usage_overview.md) — 17페이지. 터미널·셸, 기본 명령 17개, 와일드카드, 리다이렉션·파이프, 권한, 주요 디렉터리.
3. [리눅스 셸](03_linux_shell.md) — 8페이지. 터미널과 셸의 차이, sh·bash·ash·dash·zsh.
4. [파일과 디렉터리](04_files_and_directories.md) — 18페이지. 경로·파일시스템, 조회·생성·삭제·이동·복사, 하드 링크·심볼릭 링크, find.
5. [프로세스 생성 및 관리](05_processes.md) — 13페이지. 프로그램과 프로세스, 부모·자식·고아·좀비, ps·작업 제어·시그널·nice·htop, C 프로그램 생성·실행.
6. [환경 변수](06_environment_variables.md) — 6페이지. 대표 변수, PATH 검색, env·export·unset, 셸 변수와 자식 환경.
7. [파일 디스크립터](07_file_descriptors.md) — 7페이지. FD, 표준 입출력, 파이프·리다이렉션·grep.
8. [사용자와 그룹](08_users_and_groups.md) — 17페이지. 계정 파일, 사용자·그룹 관리, chgrp·chown·chmod, su·sudo.
9. [Lab 5개와 baby-linux 안내](09_labs_and_wargame.md) — Directory 6과제, File 9과제, Process 4과제, File Descriptor 3과제, User 2과제와 워게임 문제 안내.

사이트 구성은 8 Unit, 8 강의, 5 Lab, 1 워게임이며 안내 학습 시간은 약 11시간이다. 기본 컴퓨터 사용 능력과 리눅스를 설치·실행할 수 있는 환경이 선수 조건이다. Lab은 사이트의 Path 진도율에는 포함되지 않는다.

## 이 노트의 정리 기준

- 각 강의의 모든 페이지를 읽고 개념·명령·옵션·예제·화면 선택·주의 사항을 재구성했다.
- 반복된 터미널 출력 전체, 장식·저작권 문구·후기·사용자 개인 진도 등은 학습 내용으로 복제하지 않았다.
- 화면 예시의 UID, PID, IP, 파일 크기, 패키지 버전과 날짜는 고정된 정답이 아니다.
- 원문에서 실제 동작과 다른 설명은 공식 문서 근거와 함께 **보충·정정**으로 구별했다.
- 직접 설치하거나 Linux 명령을 실행해 확인한 실습 보고서는 아니다. Lab의 동적 답·일부 공식 해설, 워게임 첨부 코드 및 검증된 풀이까지 확인했다고 주장하지 않는다.

## 환경과 안전

환경 구축 강의는 2025년 3월의 UTM·VirtualBox·Ubuntu 24.04 예시, 사용법 개요 강의는 Ubuntu 22.04 Desktop 예시다. 현재 다운로드 페이지의 최신 버전이나 사용자의 화면과 차이가 날 수 있다.

노트 속 Bash 명령은 Linux VM·WSL·Dreamhack Lab용이다. **Windows PowerShell에 그대로 실행하지 않는다.** 삭제, 권한·계정 변경, 시그널, FD 쓰기, 외부 전송은 직접 만든 허가된 실습 대상에서만 다룬다. 공개 예시 비밀번호는 실제 계정에 재사용하지 않는다.

## 원 강의 바로가기

- [환경 구축 /756](https://learn.dreamhack.io/756)
- [사용법 개요 /441](https://learn.dreamhack.io/441)
- [셸 /724](https://learn.dreamhack.io/724)
- [파일·디렉터리 /725](https://learn.dreamhack.io/725)
- [프로세스 /726](https://learn.dreamhack.io/726)
- [환경 변수 /727](https://learn.dreamhack.io/727)
- [파일 디스크립터 /728](https://learn.dreamhack.io/728)
- [사용자·그룹 /729](https://learn.dreamhack.io/729)

---

다음 과정: [Linux 201 전체 학습 노트](../Linux201/README.md)
