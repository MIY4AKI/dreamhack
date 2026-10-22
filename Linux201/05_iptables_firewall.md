# Dreamhack Linux 201 — iptables 방화벽

원본 강의: [iptables](https://learn.dreamhack.io/919) · 확인일: 2026-09-30  
읽은 범위: iptables 기본, 규칙, default-deny 정책, 마무리 — 총 4페이지. 방화벽 Unit의 원문 기준 환경은 Ubuntu 26.04다.

> 강의의 규칙·옵션·조회 결과·저장 절차를 재구성했다. 예제를 실제 시스템에 적용하지 않았다. 삭제·기본 정책 변경은 접속이나 다른 프로그램의 통신에 영향을 줄 수 있으므로 독립된 실습 VM과 복구 경로를 준비한다.

## 목차

- [1. iptables와 nft의 관계](#1-iptables와-nft의-관계)
- [2. 테이블·체인·기본 정책](#2-테이블체인기본-정책)
- [3. 조회와 포트 차단](#3-조회와-포트-차단)
- [4. default-deny 구성](#4-default-deny-구성)
- [5. 저장·복원·부팅 시 적용](#5-저장복원부팅-시-적용)
- [6. 추가 명령과 복습](#6-추가-명령과-복습)

## 1. iptables와 nft의 관계

iptables는 원래 `x_tables` 기반으로 개발된 전통적인 방화벽 도구다. 과거 문서와 인터넷 자료에 iptables 예제가 많이 남아 있어, 구조를 이해하면 해당 자료를 읽는 데 도움이 된다.

강의 환경에서 `iptables` 명령은 `iptables-nft` 호환 인터페이스를 통해 `nf_tables`에 규칙을 반영한다. 같은 `nf_tables`를 사용하므로 nft로도 규칙을 확인할 수 있다. 실제 환경이 legacy 백엔드인지 nft 백엔드인지에 따라 이 관계는 달라질 수 있다.

```bash
# 설치되어 있지 않을 때 강의의 설치 명령
sudo apt install iptables
```

강의는 이전 nft 실습을 초기화하기 위해 다음 명령도 제시한다.

```bash
sudo nft flush ruleset
```

> 위 명령은 이전 강의에서 만든 테이블만 지우는 것이 아니라 **전체 nft 규칙**을 제거한다. 다른 프로그램의 규칙이 있는 시스템에서 무조건 실행하면 안 된다.

## 2. 테이블·체인·기본 정책

iptables도 테이블 → 체인 → 규칙 구조로 동작한다. nft와 달리 기본 테이블·체인이 미리 정의되어 있다. 패킷 허용·차단에는 기본적으로 **filter 테이블**을 사용한다.

filter의 기본 체인:

- `INPUT`: 이 호스트 자체로 들어오는 패킷.
- `OUTPUT`: 이 호스트에서 나가는 패킷.
- `FORWARD`: 이 호스트를 거쳐 다른 곳으로 전달되는 패킷.

강의는 외부의 호스트 접근을 `INPUT` 체인에서 관리한다. 어떤 규칙에도 최종적으로 걸리지 않은 패킷은 체인의 **기본 정책(policy)**을 따른다.

초기화된 강의 환경의 기본 체인 정책은 `ACCEPT`이므로, 차단 규칙이 없다면 이 정책은 패킷을 허용한다. 이것은 서비스가 실제로 실행 중이거나 해당 포트에서 반드시 연결이 된다는 뜻은 아니다. 방화벽의 허용과 프로그램의 listen 상태는 구분한다.

## 3. 조회와 포트 차단

### 현재 규칙 조회

```bash
sudo iptables -L
sudo iptables -L -n -v
sudo iptables -S
```

- `-L`: 체인의 규칙 목록.
- `-n`: 주소와 포트를 숫자로 출력해 이름 해석을 하지 않는다.
- `-v`: 자세한 정보와 규칙에 매칭된 패킷·바이트 통계.
- `-S`: 규칙을 명령 구문 형태로 출력한다.

### TCP 8080 차단

```bash
sudo iptables -A INPUT -p tcp --dport 8080 -j DROP
```

- `-A INPUT`: INPUT 체인의 맨 끝에 규칙 추가(append).
- `-p tcp`: TCP 프로토콜 조건.
- `--dport 8080`: 목적지 포트 조건.
- `-j DROP`: 조건이 맞으면 버리는 처리로 이동(jump).

체인의 규칙은 위에서 아래로 검사한다. 이 예제의 ACCEPT/DROP처럼 최종 판단을 하는 앞의 규칙에 매칭되면 뒤의 규칙은 적용되지 않는다. 따라서 추가 순서가 중요하다.

### nft로 같은 규칙 확인

```bash
sudo nft list ruleset
```

강의 출력에서는 `table ip filter` 아래 `chain INPUT`과 다음 규칙을 확인할 수 있다.

```nft
tcp dport 8080 counter packets 0 bytes 0 drop
```

- `ip`: 이 예제가 IPv4 규칙임을 나타낸다.
- `counter`: 매칭 패킷·바이트 통계.
- 기존 정책은 `accept`이며 8080만 명시적으로 차단한 상태다.

반대로 `iptables -L`/`-S`는 일반적으로 nft가 별도로 만든 임의의 테이블까지 보여 주지 않는다. iptables가 관리하는 테이블을 조회하기 때문이다. 한 시스템에서 nft·iptables·ufw를 섞어 쓰면 규칙의 생성 주체와 실제 처리 경로를 혼동하기 쉽다.

## 4. default-deny 구성

기본 허용에서 위험한 포트를 하나씩 차단하는 방식은 빠뜨린 서비스가 노출될 수 있다. 기본 정책을 `DROP`으로 하고 필요한 것만 허용하는 방식이 더 안전한 출발점이다.

강의는 앞서 만든 8080 DROP 규칙을 지우고 허용 규칙을 만드는 예제를 보여 준다. 기존 DROP 규칙 뒤에 8080 ACCEPT를 추가하면 앞의 DROP에 먼저 걸리므로 허용되지 않는다.

```bash
# 강의의 초기 ACCEPT 정책 상태에서만 사용하는 초기화 예시
sudo iptables -F

# 자기 컴퓨터 내부 통신과 기존 연결의 응답 허용
sudo iptables -A INPUT -i lo -j ACCEPT
sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

# 새 SSH 연결과 TCP 8080 허용
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 8080 -j ACCEPT

# 허용 규칙을 만든 뒤 마지막으로 기본 차단
sudo iptables -P INPUT DROP
```

추가로 등장한 옵션:

- `-i lo`: 입력 인터페이스가 루프백인지 검사한다.
- `-m conntrack`: 연결 추적 매치 모듈 사용.
- `--ctstate ESTABLISHED,RELATED`: 이미 맺어진 연결 및 관련 패킷만 매칭.
- `-P INPUT DROP`: INPUT 기본 정책 변경.

현재 SSH 연결은 established 규칙으로 유지될 수 있지만, 신규 연결은 실제 SSH 포트를 허용해야 한다. 포트가 22가 아니면 예제의 숫자를 그대로 사용해서는 안 된다.

> **중요 — flush와 정책은 다르다**  
> `iptables -F`는 규칙을 비우지만 기본 정책을 ACCEPT로 바꾸지 않는다. 이미 INPUT 정책이 DROP인 상태에서 실행하면 기존 SSH 허용·연결 추적 규칙이 사라져 접속을 잃을 수 있다. 강의 코드의 “SSH 연결이 끊기지 않음” 설명은 **아직 ACCEPT 정책인 실습 상태**를 전제로 한다.

> **IPv6 주의**: 여기의 `iptables` 예제는 IPv4다. IPv6까지 함께 관리하는 nft의 `inet` 예제와 범위가 다르다. IPv6 서비스가 있다면 별도의 IPv6 정책도 확인해야 한다.

## 5. 저장·복원·부팅 시 적용

명령으로 설정한 런타임 규칙은 별도 자동 복원 설정이 없으면 재부팅 후 유지되지 않는다.

```bash
# 현재 IPv4 규칙을 사용자 작업 디렉터리의 파일로 저장
sudo iptables-save > rules.v4

# 저장한 파일로 복원
sudo iptables-restore < rules.v4
```

강의는 nft처럼 설정 파일을 중심으로 작성하기보다 iptables 명령으로 규칙을 만들고 `iptables-save`로 결과를 보관하는 흐름을 권한다. 저장 파일은 현재 규칙의 직렬화 결과이며, 수정하려면 명령으로 규칙을 바꾼 뒤 다시 저장하는 방식이 입문자에게 명확하다.

### 부팅 시 자동 적용

- `iptables-persistent` 패키지를 사용한다.
- 함께 설치되는 `netfilter-persistent`가 부팅 시 저장된 규칙을 적용한다.
- `iptables-persistent`는 netfilter-persistent가 iptables 규칙을 처리하도록 돕는 구성 요소다.
- 강의는 `netfilter-persistent save` 또는 `/etc/iptables/rules.v4`로 저장하는 방식을 소개한다.

```bash
# 관리자 권한이 필요한 저장 예
sudo netfilter-persistent save
```

> **보충 — 셸 리다이렉션 권한**  
> 강의의 `iptables-save > /etc/iptables/rules.v4`는 관리자 권한으로 파일을 쓸 수 있는 환경을 전제로 한다. `sudo iptables-save > /etc/iptables/rules.v4`라고만 쓰면 `>`는 일반 사용자의 셸이 처리하므로 쓰기 권한 오류가 날 수 있다. 저장 경로의 권한과 현재 셸 권한을 구분한다. 복원은 현재 정책을 바꾸므로 파일을 확인한 뒤 수행한다.

## 6. 추가 명령과 복습

```bash
# 규칙 번호까지 확인
sudo iptables -L -n -v --line-numbers

# INPUT의 지정 번호 규칙 제거
sudo iptables -D INPUT <number>

# 사용자 정의 체인 제거
sudo iptables -X
```

- `-D`: 지정한 규칙 삭제. 하나를 삭제하면 뒤 규칙 번호가 바뀔 수 있으므로 다음 삭제 전 다시 조회한다.
- `-X`: 사용자 정의 체인 삭제. 참조되거나 비어 있지 않은 체인은 먼저 관계를 정리해야 할 수 있다.
- `-F`: 규칙 비우기.
- `-P`: 기본 정책 지정.
- `-A`: 맨 끝에 규칙 추가.

일반 포트 허용·차단 형태:

```bash
sudo iptables -A INPUT -p tcp --dport <port> -j ACCEPT
sudo iptables -A INPUT -p tcp --dport <port> -j DROP
```

- [ ] INPUT/OUTPUT/FORWARD의 대상이 다른 것을 이해한다.
- [ ] `-A/-p/--dport/-j`를 해석한다.
- [ ] 규칙 순서와 체인 기본 정책을 구분한다.
- [ ] 기존 연결의 응답 허용과 신규 SSH 접속 허용을 구분한다.
- [ ] `-F`가 기본 정책을 초기화하지 않는다는 사실을 안다.
- [ ] nft에서 보이는 규칙과 iptables에서 조회되는 범위 차이를 이해한다.
- [ ] 저장·복원·부팅 시 자동 적용을 구분한다.
- [ ] IPv4 예제를 IPv6 전체 방화벽으로 착각하지 않는다.
