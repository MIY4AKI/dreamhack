# Dreamhack Linux 201 — nft 방화벽

원본 강의: [nft](https://learn.dreamhack.io/918) · 확인일: 2026-09-30  
읽은 범위: 들어가며, 리눅스 방화벽 알아보기, nft 기본, nft 규칙, nft 마무리 — 총 5페이지. 방화벽 Unit의 원문 기준 환경은 **Ubuntu 26.04**다.

> 개념·명령·규칙 파일·출력 해석·주의점을 재구성했다. 예제 명령은 설명용이며 실행하지 않았다. 방화벽 정책 변경은 원격 접속을 끊거나 서비스를 공개할 수 있으므로, 실습 VM에서 기존 규칙과 복구 방법을 확인하고 연습한다.

## 목차

- [1. 방화벽과 공격 표면](#1-방화벽과-공격-표면)
- [2. netfilter와 세 도구](#2-netfilter와-세-도구)
- [3. nft의 계층 구조](#3-nft의-계층-구조)
- [4. 규칙 작성과 해석](#4-규칙-작성과-해석)
- [5. 파일 적용과 재부팅 후 유지](#5-파일-적용과-재부팅-후-유지)
- [6. 명령 모음과 복습](#6-명령-모음과-복습)

## 1. 방화벽과 공격 표면

인터넷에 연결된 시스템에는 정상 요청뿐 아니라 자동화된 스캔과 악의적인 접속 시도도 들어온다. 방화벽(firewall)은 미리 정한 **규칙(rule)**에 따라 네트워크 통신을 허용하거나 차단하는 1차 방어선이다.

- 웹 서비스는 보통 HTTP 80번, HTTPS 443번 포트로 요청을 받는다.
- 서비스가 늘면 외부에 노출되는 포트와 취약점 후보, 즉 **공격 표면(attack surface)**도 늘어난다.
- 꼭 필요한 통신만 허용하고 나머지는 차단하면 불필요한 노출을 줄일 수 있다.

방화벽은 필요한 방어 수단이지만, 허용한 서비스 자체의 취약점까지 없애는 것은 아니다.

## 2. netfilter와 세 도구

리눅스 커널의 **netfilter** 프레임워크가 네트워크 처리 지점에서 패킷을 검사한다. 규칙을 저장·관리하는 서브시스템으로 `x_tables`, `nf_tables` 등이 있다. 사용자는 방화벽 관리 도구로 검사 규칙을 설정한다.

- **nft**: `nf_tables`를 사용하는 현대적 도구. IPv4·IPv6·ARP 등의 규칙을 하나의 명령 체계로 관리한다.
- **iptables**: 원래 `x_tables` 기반의 전통적인 도구. 강의 환경의 Ubuntu에서는 `iptables-nft` 호환 인터페이스를 통해 `nf_tables`에 규칙을 반영한다.
- **ufw**: Uncomplicated Firewall. iptables 규칙을 간단한 명령으로 관리하도록 돕는다.

이 Unit은 nft → iptables → ufw 순서로 학습한다. 실제 시스템의 백엔드는 배포판·버전·설정에 따라 확인해야 한다.

## 3. nft의 계층 구조

### 설치할 이름과 실행할 이름

```bash
sudo apt install nftables
```

- `nf_tables`: 커널의 netfilter 서브시스템.
- `nftables`: 도구와 관련 파일이 들어 있는 **패키지**.
- `nft`: 실제로 실행하는 **명령**.
- `nftables.service`: 패키지가 제공하는 systemd 서비스.

강의 환경에는 이미 설치되어 있을 수도 있다. 도구를 설치한 것만으로 필요한 방화벽 정책이 자동으로 완성되는 것은 아니다.

### 테이블 → 체인 → 규칙

1. **테이블(table)**: 체인을 담는 상위 묶음. 생성할 때 패밀리를 지정한다.
2. **체인(chain)**: 규칙의 묶음. 강의의 베이스 체인은 패킷 처리 지점인 hook을 지정한다.
3. **규칙(rule)**: 조건에 맞는 패킷을 어떻게 처리할지 정한다.

대표 패밀리는 다음과 같다.

- `ip`: IPv4.
- `ip6`: IPv6.
- `inet`: IPv4와 IPv6를 함께 다룸.
- `arp`: ARP.
- `bridge`: 브리지 트래픽.

강의에서 쓰는 hook은 `input`(이 호스트로 들어오는 패킷), `output`(이 호스트에서 나가는 패킷), `forward`(호스트를 거쳐 다른 곳으로 가는 패킷)이다.

규칙의 대표 처리 방식:

- `accept`: 허용.
- `drop`: 조용히 버림. 송신자는 거부 응답을 받지 못한다.
- `reject`: 거부 응답을 돌려보내며 차단.

테이블·체인의 이름은 사용자가 정할 수 있다. 모든 체인이 자동으로 패킷을 검사하는 것은 아니며, 이 강의의 예제처럼 hook에 연결한 베이스 체인을 통해 검사한다.

## 4. 규칙 작성과 해석

### 현재 전체 규칙 확인

```bash
sudo nft list ruleset
```

`nf_tables`에 등록된 모든 테이블·체인·규칙을 보여 준다. 규칙이 없으면 출력이 비어 있을 수 있지만, Docker 등 다른 프로그램이 규칙을 만들어 두었다면 처음부터 출력이 나올 수도 있다.

### 강의 예제: 기본 차단, 필요한 통신만 허용

아래는 독립된 실습 환경에서 IPv4·IPv6를 함께 다루는 `inet` 테이블을 만들고, 루프백·기존 연결·SSH·TCP 8080을 허용하는 흐름이다.

```bash
# 테이블 생성
sudo nft add table inet my_table

# 처음에는 기본 허용 정책으로 체인 생성
sudo nft 'add chain inet my_table my_chain { type filter hook input priority 0; policy accept; }'

# 필요한 통신 허용
sudo nft add rule inet my_table my_chain iif lo accept
sudo nft add rule inet my_table my_chain ct state established,related accept
sudo nft add rule inet my_table my_chain tcp dport 22 accept
sudo nft add rule inet my_table my_chain tcp dport 8080 accept

# 허용 규칙이 준비된 뒤 기본 정책을 차단으로 변경
sudo nft 'add chain inet my_table my_chain { type filter hook input priority 0; policy drop; }'
```

이 방식은 **default-deny**: 기본적으로 거부하고 필요한 예외만 허용하는 정책이다.

> **보충 — 완전한 IPv6 정책은 아니다**  
> `inet`은 IPv4·IPv6를 함께 다루지만, 위 코드는 강의의 최소 실습 규칙이다. 루프백·기존 연결·TCP 포트만 허용하고 나머지를 모두 버리면 IPv6 Neighbor Discovery 등 필수 ICMPv6 제어 메시지가 차단되어 IPv6 통신에 문제가 생길 수 있다. 실제 환경에서는 Neighbor Solicitation/Advertisement, 필요한 Router Advertisement 및 오류 메시지 등을 별도로 검토해야 한다. [RFC 4890의 ICMPv6 필터링 권고](https://www.rfc-editor.org/rfc/rfc4890.html)

### 체인 생성 구문의 각 부분

- `add chain`: 체인 추가.
- `inet`: 패밀리.
- `my_table`: 체인이 속하는 테이블.
- `my_chain`: 체인 이름.
- `type filter`: 허용·차단을 위한 필터 체인.
- `hook input`: 이 호스트로 들어오는 패킷 처리 지점에 연결.
- `priority 0`: 같은 hook의 체인 검사 우선순위.
- `policy accept`/`drop`: 체인의 규칙으로 결정되지 않은 패킷의 기본 처리.

세미콜론과 중괄호를 셸이 별도 구문으로 해석하지 않도록 따옴표로 묶는다.

### 허용 조건의 뜻

- `iif lo`: 들어온 인터페이스(input interface)가 루프백 `lo`인지 검사한다.
- `tcp dport 22`: TCP 목적지 포트(destination port)가 22인지 검사한다.
- `tcp dport 8080`: TCP 목적지 포트가 8080인지 검사한다.
- `ct state established,related`: 기존 연결과 관련된 패킷을 연결 추적(conntrack) 상태로 구분한다.

**루프백**은 자기 컴퓨터 안에서 통신하는 가상 인터페이스다. 보통 `lo`, IPv4 `127.0.0.1`을 사용한다. 많은 프로그램이 내부 통신에 이용하므로, 이를 모두 차단하면 정상 기능이 깨질 수 있다.

**기존 연결 허용**은 중요하다. INPUT의 기본 정책을 drop으로 바꾸면 서버가 시작한 패키지 다운로드·DNS 조회 등의 정상 응답도 들어오는 패킷이므로 차단될 수 있다. established/related 규칙은 이런 응답을 통과시킨다.

현재 SSH 세션도 established에 해당해 유지될 수 있다. 하지만 이것만으로 **새로운** SSH 접속까지 허용되는 것은 아니다. 종료 후 재접속하려면 실제 SSH 포트의 허용 규칙이 필요하다.

### 순서와 원격 접속 주의

한 체인에서는 규칙을 위에서 아래로 검사한다. 먼저 추가한 규칙이 먼저 검사되므로, 앞의 광범위한 허용·차단 규칙이 뒤의 규칙을 무의미하게 만들 수 있다.

허용 규칙을 만들기 전에 정책을 drop으로 바꾸면 SSH를 포함한 통신을 잃을 수 있다. SSH가 반드시 22번인 것은 아니므로 실제 포트와 관리자 접속 경로를 먼저 확인한다.

### 결과 확인

```bash
sudo nft list ruleset
```

출력에는 `table inet my_table`, 그 안의 `chain my_chain`, `policy drop`, 루프백·연결 상태·22·8080 허용 규칙이 나타난다. 입력한 `priority 0`이 출력에서는 `priority filter`로 표시될 수 있다. nft가 알려진 우선순위 값을 대응하는 이름으로 출력하기 때문이다.

## 5. 파일 적용과 재부팅 후 유지

명령으로 커널에 설정한 규칙만으로는 일반적으로 재부팅 후 유지되지 않는다. 강의는 규칙을 파일에 작성해 전체를 불러오는 방식을 소개한다.

### 강의 예제를 파일로 표현

```nft
#!/usr/sbin/nft -f

# 독립 실습 환경의 기존 규칙을 제거하는 예제
flush ruleset

table inet my_table {
    chain my_chain {
        type filter hook input priority 0; policy drop;
        iif lo accept
        ct state established,related accept
        tcp dport 22 accept
        tcp dport 8080 accept
    }
}
```

`my_ruleset.nft`로 저장한 뒤 적용하는 명령:

```bash
sudo nft -f my_ruleset.nft
```

파일의 규칙을 한 번에 반영하므로, 한 줄씩 설정할 때처럼 “차단 정책만 먼저 적용되고 허용 규칙은 아직 없는” 중간 상태를 피할 수 있다. 다만 파일 자체에서 현재 접속과 재접속을 허용하지 않으면 여전히 접속을 잃는다.

> **중요**: `flush ruleset`은 모든 nft 규칙을 제거한다. Docker·ufw·다른 관리자 도구의 규칙까지 영향을 받을 수 있다. 운영 시스템에 그대로 적용할 초기화 템플릿이 아니다. 정책 파일과 연결 상태를 검토하고 별도의 콘솔·복구 경로를 준비한다.

작성한 설정을 `/etc/nftables.conf`에 저장하고 `nftables` 서비스를 활성화해 두면, 부팅 시 설정을 다시 적용하는 방식으로 유지할 수 있다. **파일 저장, 현재 커널에 적용, 부팅 서비스 활성화**는 서로 다른 단계다.

## 6. 명령 모음과 복습

아래 관리 명령은 root 권한이 필요하므로 `sudo`를 붙인다.

```bash
# 조회
sudo nft list ruleset
sudo nft list tables

# 생성 및 적용
sudo nft add table inet <table>
sudo nft 'add chain inet <table> <chain> { type filter hook input priority 0; policy accept; }'
sudo nft add rule inet <table> <chain> tcp dport <port> accept
sudo nft -f <filename>

# 삭제·초기화 — 영향 범위를 확인한 실습 환경에서만
sudo nft delete table inet <table>
sudo nft flush ruleset
```

테이블 삭제는 그 안의 체인과 규칙도 제거한다. 전체 flush는 모든 테이블·체인·규칙을 비운다.

- [ ] netfilter, nf_tables, nftables 패키지, nft 명령을 구분한다.
- [ ] table → chain → rule 관계와 `inet` 패밀리를 이해한다.
- [ ] hook, priority, policy의 역할을 설명한다.
- [ ] accept/drop/reject의 차이를 안다.
- [ ] 루프백과 established/related 규칙이 필요한 이유를 안다.
- [ ] 현재 SSH 세션 유지와 신규 SSH 접속 허용을 구분한다.
- [ ] 허용 규칙을 먼저 준비하고 차단 정책을 적용한다.
- [ ] 파일 적용과 재부팅 후 영속화의 차이를 안다.
- [ ] 초기화 명령이 다른 프로그램의 규칙에도 영향을 준다는 점을 이해한다.

모든 명령을 외우기보다 구조를 이해하고 필요한 명령을 찾아 사용할 수 있는 정도를 목표로 한다.
