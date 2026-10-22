# Dreamhack Linux 201 — 방화벽 고급 설정과 Docker

원본 강의: [방화벽 고급 설정과 Docker](https://learn.dreamhack.io/921) · 확인일: 2026-09-30  
읽은 범위: 들어가며, 고급 설정, Docker와 ufw, 마치며 — 총 4페이지. 방화벽 Unit의 원문 기준 환경은 Ubuntu 26.04다.

> 조건·규칙 예제·NAT·Docker 패킷 경로를 학습용으로 재구성했다. 모든 명령은 설명용이며 실행하지 않았다. 예제는 nft/iptables가 관리하는 독립 실습 규칙을 전제로 하며, 기존 Docker·ufw 설정에 그대로 이어 붙이는 실행 스크립트가 아니다.

## 목차

- [1. 세밀한 조건을 사용하는 이유](#1-세밀한-조건을-사용하는-이유)
- [2. 출발지 IP 기반 제어](#2-출발지-ip-기반-제어)
- [3. 패킷 개수 제한](#3-패킷-개수-제한)
- [4. 인터페이스 조건](#4-인터페이스-조건)
- [5. NAT와 hook](#5-nat와-hook)
- [6. Docker 공개 포트와 ufw](#6-docker-공개-포트와-ufw)
- [7. 복습 체크](#7-복습-체크)

## 1. 세밀한 조건을 사용하는 이유

운영 환경에서는 포트 허용·차단만으로 정책을 표현하기 어려울 수 있다. 출발지 IP·대역, 들어오는 패킷 수, 인터페이스, 주소 변환 등을 기준으로 제어해야 한다. 강의는 이런 세부 기능을 nft와 iptables로 직접 다루는 방법을 소개한다.

ufw의 간단한 명령은 일반적인 호스트 방화벽 관리에 편리하지만, 모든 하위 규칙이나 Docker가 별도로 만든 규칙까지 그 명령만으로 관리하는 것은 아니다. 단순화된 “ufw는 아무 고급 조건도 지원하지 않는다”는 의미로 받아들이지는 않는다.

## 2. 출발지 IP 기반 제어

### 특정 IP에서 TCP 8080 허용

```bash
# nft
sudo nft add rule inet my_table my_chain ip saddr 203.0.113.10 tcp dport 8080 accept

# iptables
sudo iptables -A INPUT -p tcp --dport 8080 -s 203.0.113.10 -j ACCEPT
```

- nft의 `ip saddr`: IPv4 출발지 주소(source address).
- iptables의 `-s`: 출발지 주소.
- 포트와 주소 조건을 **함께** 만족하는 패킷을 허용한다.
- `203.0.113.10`은 강의의 설명용 주소로, 실제 사무실 주소가 아니다.

### 내부 대역에서 SSH 허용

```bash
# nft
sudo nft add rule inet my_table my_chain ip saddr 192.168.0.0/24 tcp dport 22 accept

# iptables
sudo iptables -A INPUT -p tcp --dport 22 -s 192.168.0.0/24 -j ACCEPT
```

`/24`는 앞의 24비트를 네트워크 부분으로 보는 CIDR 표기다. 이 예제는 `192.168.0.0/24` 대역에서 들어오는 SSH 연결을 허용한다. 민감한 관리 서비스를 신뢰할 수 있는 대역으로 제한하는 방법이다.

> **적용 전제**: 이것이 “그 IP/대역에서만 허용”하는 정책이 되려면 다른 모든 출발지를 허용하는 앞선 규칙이 없어야 하고, 나머지를 차단하는 규칙 또는 기본 DROP 정책이 있어야 한다. 이전 강의의 광범위한 `tcp dport 22 accept` 뒤에 위 규칙을 추가하는 것만으로 SSH 접근이 제한되지는 않는다.

또한 nft의 `inet` 테이블 안에 있더라도 `ip saddr`는 **IPv4 조건**이다. IPv6 접속 정책을 별도로 검토한다. 현재 관리자가 접속한 주소와 대역을 먼저 확인해야 원격 접속을 잃지 않는다.

## 3. 패킷 개수 제한

짧은 시간에 들어오는 패킷 수를 제한하는 rate limiting은 무차별 대입(brute-force)이나 요청 폭주(flooding)를 완화하는 수단이다.

```bash
# nft: 허용량 안의 패킷을 통과시키고 나머지 SSH 패킷은 버림
sudo nft add rule inet my_table my_chain tcp dport 22 limit rate 10/minute accept
sudo nft add rule inet my_table my_chain tcp dport 22 drop

# iptables
sudo iptables -A INPUT -p tcp --dport 22 -m limit --limit 10/minute -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 22 -j DROP
```

- `limit rate 10/minute` / `--limit 10/minute`: 평균 허용률을 분당 10개로 제한한다.
- 제한을 넘는 패킷은 첫 규칙에 매칭되지 않고 다음 규칙 또는 기본 정책으로 넘어간다.
- 예제의 두 번째 규칙은 남은 SSH 패킷을 명시적으로 버린다.

### iptables 매치 모듈

`-m <모듈>`은 기본 조건 이외의 매치 기능을 불러온다.

- `limit`: 허용률 조건.
- `multiport`: 여러 포트 지정.
- 이전 강의의 `conntrack`: 연결 상태 조건.

nft는 해당 조건을 문법 자체로 표현하므로 같은 용도의 `-m` 옵션을 따로 사용하지 않는다.

> **보충/주의**: 이 예제는 SSH **로그인 시도 횟수**가 아니라 조건에 매칭되는 **패킷**의 허용률을 다룬다. 실제 제한은 burst 등의 설정에도 영향을 받으므로 매 분마다 정확히 10개를 허용하는 고정 카운터로 생각하지 않는다. 앞에 무조건 SSH를 허용하는 규칙이 있으면 제한에 도달하지 않으며, 잘못 구성하면 정상 SSH 통신도 방해할 수 있다. 운영 환경의 안전한 로그인 방어 전체를 이 두 줄로 대체할 수는 없다.

## 4. 인터페이스 조건

들어오는 인터페이스에 따라 규칙을 구분할 수 있다. 강의는 루프백 `lo`를 허용하는 예제로 설명한다.

```bash
# nft
sudo nft add rule inet my_table my_chain iif lo accept

# iptables
sudo iptables -A INPUT -i lo -j ACCEPT
```

- `iif`: input interface.
- `-i`: 입력 인터페이스.
- 루프백으로 들어오는 내부 통신을 허용한다.

출력 인터페이스는 다음 NAT 예제의 `oif` 또는 `-o`로 조건을 지정한다.

## 5. NAT와 hook

**NAT(Network Address Translation)**는 패킷의 주소 정보를 변환하는 기능이다. 내부 네트워크의 주소를 외부 인터페이스 주소로 바꾸는 masquerading, 포트 포워딩 등이 관련된다. 컨테이너·가상화 네트워크에서도 자주 사용한다.

### 출발지 주소를 바꾸는 masquerade

```bash
# nft: nat 테이블과 postrouting hook의 nat 타입 체인을
# 먼저 구성했다는 전제의 규칙 추가 명령
sudo nft add rule inet nat postrouting oif eth0 masquerade

# iptables
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```

여기서 nft 예제의 `nat`은 테이블 이름, `postrouting`은 체인 이름이다. 그 체인은 실제로 `postrouting` hook을 가진 NAT 체인으로 미리 만들어져 있어야 한다. 위 한 줄만으로 필요한 테이블·체인을 자동 생성하지 않는다.

### 구문 해석

nft:

- `oif eth0`: 나가는 인터페이스(output interface)가 `eth0`인 패킷.
- `masquerade`: 출발지 주소를 해당 인터페이스의 주소로 바꿈.

iptables:

- `-t nat`: filter가 아닌 NAT 테이블 선택.
- `-A POSTROUTING`: 라우팅 후의 POSTROUTING 체인에 추가.
- `-o eth0`: 출력 인터페이스 조건.
- `-j MASQUERADE`: 마스커레이딩 적용.

`eth0`는 강의의 예시 이름이다. 실제 인터페이스 이름과 네트워크를 확인한다. 주소 변환 규칙은 그 자체로 모든 라우팅·포워딩·보안 설정을 완성하는 것이 아니다.

### hook이란?

hook은 패킷이 커널의 네트워크 처리 과정에서 규칙 검사를 받는 지점이다.

- `input`: 이 호스트로 전달.
- `output`: 이 호스트에서 생성하여 전송.
- `forward`: 다른 목적지로 중계.
- `postrouting`: 라우팅을 마치고 인터페이스로 나가기 전.

체인은 자신이 연결될 hook을 지정하고 그 지점을 지나는 패킷에 규칙을 적용한다. NAT처럼 특정 단계에서 수행해야 하는 처리는 알맞은 hook과 체인 종류를 선택해야 한다.

## 6. Docker 공개 포트와 ufw

### 호스트 INPUT과 컨테이너 FORWARD의 차이

강의의 공개 포트 예시는 `docker run -p 8080:80`이다. 이미지 등 나머지 인수는 생략한 설명용 구문이며, 호스트 8080번을 컨테이너 80번으로 연결한다.

호스트 서비스로 들어오는 일반 패킷은 INPUT 경로를 사용하지만, Docker가 브리지 네트워크의 컨테이너로 전달하도록 만든 공개 포트 트래픽은 다른 경로를 사용한다.

```text
외부 클라이언트 → 호스트 주소:8080
    → Docker 규칙이 목적지를 컨테이너 주소:80으로 변환
    → 커널이 라우팅 시 목적지가 호스트 자신이 아님을 확인
    → FORWARD 경로를 통해 컨테이너로 전달
```

강의 환경에서 Docker는 iptables에 허용·주소 변환 규칙을 직접 추가한다. 컨테이너 트래픽은 INPUT이 아니라 FORWARD 체인 경로에 해당한다.

### ufw status가 전체 노출을 보여 주지 않는 이유

`sudo ufw status`는 ufw가 관리하는 규칙을 보여 준다. Docker가 직접 추가한 규칙은 ufw를 통해 생성한 것이 아니므로 여기에 모두 나타나지 않는다.

또한 일반적인 다음 명령은 호스트로 들어오는 INPUT 연결을 차단하는 정책이다.

```bash
sudo ufw deny 8080/tcp
```

이 규칙만으로 Docker가 공개한 컨테이너 포트를 차단했다고 판단할 수 없다. 실제 패킷이 FORWARD 경로를 지나면 외부에서 여전히 접근할 수 있다. **ufw 상태 화면만 보고 컨테이너가 보호된다고 결론내리지 않는다.**

### 관리할 때 기억할 점

- 호스트 자신이 목적지인 패킷은 INPUT, 호스트가 다른 곳으로 보내는 패킷은 FORWARD라는 구분을 먼저 한다.
- Docker의 목적지 주소 변환은 라우팅 결정 전에 일어나므로 외부에서 원래 호스트 IP로 보낸 요청도 FORWARD로 분류될 수 있다.
- 강의는 Docker 규칙을 관리하려면 nft나 iptables의 실제 경로를 확인해야 한다고 설명한다.
- 강의 환경에서는 Docker가 iptables를 사용하므로 iptables 기준으로 읽는 것이 편리하다.
- Docker의 네트워크 모드·버전·선택한 방화벽 백엔드가 달라지면 구현과 규칙 구조도 달라질 수 있다. 이 설명을 모든 Docker 환경의 유일한 경로라고 단정하지 않는다.
- Docker가 만든 규칙을 무조건 flush하는 것은 컨테이너 네트워크를 깨뜨릴 수 있다. 이 강의는 구체적인 운영 방화벽 배포 절차나 완성된 Docker 차단 스크립트를 제공하는 강의가 아니다.

## 7. 복습 체크

- [ ] 출발지 주소·포트 조건을 동시에 해석한다.
- [ ] 한 IP와 CIDR 대역 조건의 차이를 안다.
- [ ] 제한적 허용 규칙을 추가하는 것과 광범위한 허용을 제거하는 것이 다른 작업임을 안다.
- [ ] rate limiting의 대상이 패킷임을 설명한다.
- [ ] `-m limit`, `-m multiport`, `-m conntrack`의 역할을 구분한다.
- [ ] `iif/-i`와 `oif/-o`를 구별한다.
- [ ] masquerade의 출발지 주소 변환과 NAT 테이블의 역할을 설명한다.
- [ ] hook과 단순 체인 이름을 구분한다.
- [ ] Docker 공개 포트가 INPUT이 아닌 FORWARD로 갈 수 있는 이유를 설명한다.
- [ ] ufw status·deny만으로 Docker의 외부 접근을 판단하면 안 됨을 이해한다.

모든 옵션을 외우는 것이 목표는 아니다. 필요한 조건과 패킷 경로를 이해한 뒤 `man` 페이지나 공식 문서에서 해당 문법을 찾아 사용할 수 있도록 한다.
