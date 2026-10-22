# Dreamhack Linux 201 — ufw 방화벽

원본 강의: [ufw](https://learn.dreamhack.io/920) · 확인일: 2026-09-30  
읽은 범위: ufw 기본, 규칙, 활성화 및 상태 확인, 마무리, 방화벽 구성해보기 Lab, Unit 마치며 — 총 6페이지. 방화벽 Unit의 원문 기준 환경은 Ubuntu 26.04다.

> 강의의 명령·내부 처리 과정·영속화·실습 안내를 재구성했다. 설치·활성화·초기화는 실행하지 않았다. 방화벽 설정 변경은 실제 접속에 영향을 주므로 독립 실습 환경과 복구 수단을 마련한다.

## 목차

- [1. ufw와 기본 정책](#1-ufw와-기본-정책)
- [2. 포트 규칙과 애플리케이션 프로파일](#2-포트-규칙과-애플리케이션-프로파일)
- [3. 활성화·조회·삭제](#3-활성화조회삭제)
- [4. 내부 처리와 재부팅](#4-내부-처리와-재부팅)
- [5. 명령 모음](#5-명령-모음)
- [6. Lab·Unit 정리·복습](#6-labunit-정리복습)

## 1. ufw와 기본 정책

**ufw(Uncomplicated Firewall)**는 복잡한 iptables 규칙을 짧은 명령으로 다루도록 만든 도구다. 강의의 Ubuntu 환경에는 설치되어 있지만 처음에는 **비활성화** 상태다. 설치와 활성화는 다른 단계다.

```bash
# 설치되지 않았을 때
sudo apt install ufw
```

기본 정책은 어떤 규칙에도 걸리지 않은 통신의 처리다. 강의 환경에서 ufw는 다음 기본 정책을 사용한다.

- 들어오는 연결: `deny incoming`.
- 나가는 연결: `allow outgoing`.
- 상태 출력의 routed 정책: `deny routed`.

따라서 활성화되면 들어오는 연결은 기본적으로 차단되고 별도로 허용한 통신만 접근할 수 있다. iptables 초기 ACCEPT 정책과 비교해 이해한다.

### 이전 실습 초기화에 관한 주의

강의는 이전 iptables 규칙을 비우는 명령을 제시한다.

```bash
sudo iptables -F
```

> **보충/주의**: `-F`는 기존 규칙을 비우는 것이지 기본 정책을 초기화하는 것이 아니다. 직전 iptables 실습에서 INPUT을 DROP으로 설정했다면, 이 명령만 실행할 때 SSH 허용 규칙까지 사라져 접속을 잃을 수 있다. 강의의 실습 전제와 현재 정책을 대조해야 한다. 다른 프로그램이 관리하는 체인이 있는 실제 시스템에서도 무조건 초기화하지 않는다.

## 2. 포트 규칙과 애플리케이션 프로파일

### 포트 허용·차단

```bash
# TCP 8080으로 들어오는 연결 허용
sudo ufw allow 8080/tcp

# TCP 8080으로 들어오는 연결 차단
sudo ufw deny 8080/tcp

# 추가했던 허용 규칙 제거
sudo ufw delete allow 8080/tcp

# 사용자가 추가한 규칙 확인
sudo ufw show added
```

허용·차단 명령을 모두 나열한 것은 각 문법을 설명하기 위해서다. 같은 포트에 상충되는 규칙을 무작정 차례로 넣기보다, 의도한 정책과 기존 규칙을 먼저 확인한다.

### 애플리케이션 프로파일

일부 프로그램은 자신이 사용하는 포트를 ufw의 **애플리케이션 프로파일**에 등록한다. 이름으로 규칙을 추가하면 포트 번호를 일일이 외울 부담을 줄일 수 있다.

```bash
sudo ufw app list
sudo ufw app info OpenSSH
sudo ufw allow OpenSSH
```

- `app list`: 설치되어 등록된 프로파일 목록.
- `app info <profile>`: 해당 프로파일의 설명과 포트 정보.
- `allow <profile>`: 프로파일이 지정한 통신 허용.

강의의 OpenSSH 프로파일 출력에는 Profile, Title, Description과 함께 **`22/tcp`**가 나타난다. SSH 서버가 설치되어 있어야 해당 프로파일이 나타날 수 있다.

> 프로파일이 실제 서비스 설정을 자동으로 검증해 주는 것은 아니다. SSH 포트를 변경했다면 현재 포트와 프로파일의 22/tcp가 일치하는지 확인한다.

## 3. 활성화·조회·삭제

### 활성화와 비활성화

```bash
sudo ufw enable
sudo ufw disable
```

활성화하기 전에 관리자 재접속에 필요한 포트를 허용한다. `disable`은 방화벽 보호를 중단할 수 있으므로 단순히 오류를 해결하는 습관처럼 사용하면 안 된다.

### 상태 조회

```bash
sudo ufw status
sudo ufw status verbose
sudo ufw status numbered
```

- `status`: 활성화 여부와 사용자 규칙.
- `verbose`: 기본 정책·로깅 등 자세한 정보.
- `numbered`: 규칙 번호. 번호로 삭제할 때 사용한다.

강의는 `enable` 후 다음 상태 정보를 확인한다.

- 활성화 및 부팅 시 활성화 안내.
- `Status: active`.
- `Logging: on (low)`.
- `Default: deny (incoming), allow (outgoing), deny (routed)`.
- `New profiles: skip`.
- OpenSSH의 `22/tcp ALLOW IN` 규칙이 IPv4와 IPv6 양쪽에 표시됨.

### 규칙 삭제

규칙을 추가했던 문법으로 지정하거나 조회한 번호를 이용한다.

```bash
sudo ufw delete allow 8080/tcp
sudo ufw status numbered
sudo ufw delete <number>
```

삭제 후 번호가 바뀔 수 있으므로 다음 규칙을 지우기 전에 다시 조회한다.

### 기존 연결과 새 연결

nft·iptables 예제는 established/related 허용을 직접 추가했다. ufw는 활성화 시 기본 규칙에 연결 추적 상태의 정상 응답 허용을 포함한다.

따라서 기존 SSH 세션이 유지될 수 있지만, 현재 연결이 유지된다는 사실만으로 **재접속 가능**하다고 판단하면 안 된다. 세션이 끝난 뒤 새로 연결하려면 실제 SSH 포트 허용 규칙이 필요하다. 활성화 전 허용하고 별도의 연결로 재접속 여부를 확인하는 흐름을 이해한다.

## 4. 내부 처리와 재부팅

강의 환경의 규칙 반영 과정:

```text
ufw 명령
   → iptables 규칙(iptables-nft)
   → nf_tables
   → netfilter가 패킷 처리
```

ufw는 `ufw-...` 이름의 전용 iptables 체인을 만들고 그 안에서 규칙을 관리한다. 다음 명령으로 같은 설정이 하위 도구에서 어떻게 보이는지 확인할 수 있다.

```bash
sudo ufw allow 8080/tcp
sudo iptables -S
sudo nft list ruleset
```

- `iptables -S`: ufw 전용 체인과 규칙을 확인.
- `nft list ruleset`: nft 백엔드에 반영된 규칙을 확인.

반대로 `ufw status`는 다른 프로그램이 임의로 만든 모든 방화벽 규칙을 보여 주는 명령이 아니다. Docker가 공개한 포트를 이해할 때 특히 중요하며, 다음 강의에서 다룬다.

ufw는 설정과 활성화 상태를 파일로 저장하고 부팅 시 다시 적용한다. 따라서 ufw로 관리한 설정은 별도의 `iptables-save`나 `nft -f` 절차 없이 재부팅 후 유지하는 흐름이다. 단, 서비스·운영체제 설정은 실제 환경에서 확인해야 한다.

## 5. 명령 모음

일반적인 대상 문법:

```bash
sudo ufw allow <port>/tcp
sudo ufw deny <port>/tcp
sudo ufw delete allow <port>/tcp
sudo ufw allow <profile>
sudo ufw app info <profile>
```

추가 관리 명령:

```bash
# 사용자 규칙과 프로파일
sudo ufw show added
sudo ufw app list

# 기본 수신 정책 변경
sudo ufw default deny incoming

# 설정 다시 읽어 적용
sudo ufw reload

# 모든 규칙과 설정 초기화 — 기존 정책을 잃을 수 있음
sudo ufw reset
```

`reset`은 단순 조회나 새로고침이 아니다. 이전 규칙을 제거하는 초기화 작업이므로 필요한 설정을 먼저 보관하고 영향 범위를 확인한다.

## 6. Lab·Unit 정리·복습

강의의 **방화벽 구성해보기 Lab** 페이지는 앞에서 배운 내용을 실습 환경에서 확인하도록 배너를 안내한다. 강의 본문에서는 구체적인 풀이를 제공하지 않는다. 접근 가능 여부와 실습 문제 안내는 이 폴더의 별도 실습 정리를 따른다.

Unit의 결론:

- 실제 패킷 처리는 커널의 netfilter가 담당한다.
- nft는 현대적 관리 도구, iptables는 기존 자료를 읽는 데 중요한 도구, ufw는 간편한 관리 도구다.
- **기본 차단 후 꼭 필요한 항목만 허용**하는 원칙을 공통으로 적용한다.
- 원격 관리에서는 기존 세션 유지와 재접속을 모두 고려한다.
- 새로 학습한다면 nft를 먼저 익히되 과거 iptables 예제도 읽을 수 있도록 한다.
- 복잡한 설정과 Docker 공개 포트에는 단순 ufw 규칙만으로 충분하지 않을 수 있다.

- [ ] 설치와 활성화가 다르다는 점을 설명한다.
- [ ] incoming/outgoing/routed 정책을 구분한다.
- [ ] 포트·프로파일 이름으로 규칙을 만들 수 있다.
- [ ] `status`, `verbose`, `numbered`, `show added`의 차이를 안다.
- [ ] 규칙 문법 또는 번호로 삭제할 수 있다.
- [ ] ufw → iptables-nft → nf_tables 관계를 설명한다.
- [ ] ufw 상태 출력이 전체 시스템의 모든 규칙을 보여 주지 않음을 안다.
- [ ] 재부팅 시 유지되는 이유와 reset의 영향을 안다.
- [ ] SSH 허용을 확인한 뒤 활성화하는 순서를 이해한다.
