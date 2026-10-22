# Dreamhack Linux 101 — 리눅스 환경 구축하기

- 원문: [리눅스 환경 구축하기](https://learn.dreamhack.io/756)
- 확인일: 2026-09-30
- 읽은 범위: 원 강의의 4개 대주제, 전체 22페이지(`#1`~`#22`)
- 정리 방식: 개념, 설치 순서, 화면에서 선택한 값, 명령어를 학습용으로 재구성했다. 설치 프로그램과 운영체제를 실제로 설치하거나 실행해 검증한 기록은 아니다.

> 강의의 설치 예시는 2025년 3월을 기준으로 작성되었다. UTM 4.6.4, VirtualBox 7.1.4, Ubuntu 24.04.2의 화면을 설명하므로 현재 프로그램의 화면·기능·다운로드 버전과 다를 수 있다. Apple Silicon에서 x86 GUI 환경을 구축하기 어렵다는 설명도 당시의 상황에 관한 것이다.

## 목차

1. [가상 머신의 개념과 필요성 — 1~2페이지](#1-가상-머신의-개념과-필요성--12페이지)
2. [Apple Silicon Mac: UTM과 Ubuntu Server — 3~9페이지](#2-apple-silicon-mac-utm과-ubuntu-server--39페이지)
3. [Windows: VirtualBox와 Ubuntu Desktop — 10~19페이지](#3-windows-virtualbox와-ubuntu-desktop--1019페이지)
4. [Windows: WSL2와 Ubuntu — 20~22페이지](#4-windows-wsl2와-ubuntu--2022페이지)
5. [명령어 모음과 완료 확인](#5-명령어-모음과-완료-확인)
6. [복습 체크리스트](#6-복습-체크리스트)

## 1. 가상 머신의 개념과 필요성 — 1~2페이지

### 1페이지: 가상 머신

가상 머신(Virtual Machine, VM)은 실제 컴퓨터의 자원을 이용해 별도의 컴퓨터 환경을 구성하는 방식이다. 한 물리 컴퓨터에서 여러 운영체제를 사용하거나, 같은 운영체제의 서로 다른 버전으로 실습 환경을 만들 수 있다.

- **호스트 머신(Host)**: VM 프로그램을 실행하는 실제 컴퓨터. 이 강의에서는 Mac 또는 Windows 컴퓨터다.
- **게스트 머신(Guest)**: 호스트 위에서 실행되는 가상 컴퓨터. 그 안에 Ubuntu를 설치한다.
- 하나의 호스트에서 여러 VM을 실행할 수 있다. 운영체제마다 물리 컴퓨터를 마련하는 비용과 관리 부담을 줄일 수 있다.

해킹 실습과 워게임은 Windows, Linux, 특정 운영체제 버전 등 서로 다른 실행 조건을 요구한다. 따라서 실습 대상에 맞는 환경을 만드는 능력이 필요하다. 강의는 Apple Silicon Mac의 UTM, Windows의 VirtualBox, Windows의 WSL2를 각각 소개한다.

### 2페이지: VM을 사용하는 이유

1. **환경 재현**: 실습 프로그램에 맞는 운영체제와 버전을 별도의 게스트에 구성한다.
2. **실험 환경 분리**: 호스트와 구분된 환경에서 프로그램을 분석하거나 실험한다.
3. **복구**: VirtualBox의 스냅샷으로 특정 시점의 VM 상태를 저장하고, 오류가 생기면 저장한 상태로 돌아갈 수 있다.

강의는 독립된 VM을 악성 코드 분석의 장점으로 설명한다. 다만 이를 “VM에서 실행하면 호스트가 절대 영향을 받지 않는다”는 보장으로 이해하면 안 된다. 강의 후반의 네트워크 연결, 공유 폴더, 클립보드, 드래그 앤 드롭은 호스트와 게스트 사이의 연결 기능이다. 환경 분리의 정도는 이런 연결과 VM 설정에 따라 달라진다.

**Apple Silicon과 x86 실습의 관계**: 강의와 많은 CTF·워게임은 x86 계열 실행 환경을 전제로 한다. Apple Silicon은 ARM 계열이므로 x86 게스트를 실행할 때 아키텍처 차이를 해결해야 한다. 강의가 설명한 x86 Linux/Windows GUI 환경의 성능상 어려움은 **2025년 3월 기준**이며, 최신 상태에 대한 판단으로 확대하지 않는다.

## 2. Apple Silicon Mac: UTM과 Ubuntu Server — 3~9페이지

### 3페이지: 사용할 환경

UTM은 macOS에서 여러 운영체제 환경을 실행하는 오픈소스 VM 소프트웨어다. 강의는 Apple Silicon Mac에 **Ubuntu 24.04 계열 64-bit PC(AMD64) Server**를 설치한다. 여기서 AMD64는 x86-64 아키텍처를 뜻하며, ARM용 Ubuntu를 설치하는 예제가 아니다.

### 4페이지: UTM 다운로드

- 강의의 UTM 버전: **4.6.4** — 2025년 3월 당시의 예시.
- [UTM macOS 설치 안내](https://docs.getutm.app/installation/macos/)에서 **Download from Github**를 선택해 `UTM.dmg`를 받는다.

### 5페이지: UTM 설치

1. `UTM.dmg`를 연다.
2. 디스크 이미지 창에서 `UTM.app`을 **Applications**로 드래그한다.
3. 응용 프로그램 목록에 추가된 `UTM.app`을 실행한다.
4. UTM의 초기 화면이 열리는지 확인한다.

### 6페이지: Ubuntu Server ISO 다운로드

[Ubuntu Server 다운로드](https://ubuntu.com/download/server/)에서 설치 ISO를 받는다. 강의 화면의 실제 파일은 다음과 같다.

```text
ubuntu-24.04.2-live-server-amd64.iso
```

본문의 `24.04.02` 표기는 화면에 표시된 `24.04.2`와 다르므로, 이 노트에서는 실제 ISO 파일명에 맞춰 **24.04.2 LTS**로 적었다. Finder에서 다운로드를 확인할 때 **Server인지 Desktop인지**, **amd64 파일인지** 구분한다. 이후 VirtualBox 예제는 Desktop ISO를 쓰므로 두 파일을 혼동하지 않는다.

### 7페이지: UTM 가상 머신 생성

1. Mac에서 `Command ⌘ + Space`로 Spotlight를 열어 UTM을 실행한다.
2. **Create a New Virtual Machine**을 선택한다.
3. **Emulate**를 선택한다. 호스트는 ARM인 Apple Silicon이고 설치할 게스트는 AMD64이므로 아키텍처가 다르기 때문이다. **Virtualize**와 **Emulate**를 같은 방식으로 생각하지 않는다.
4. 운영체제 종류로 **Linux**를 선택한다.
5. **Browse…**에서 앞서 받은 Server ISO를 고르고 **Continue**로 진행한다.
6. **Hardware**에서 강의의 다음 값을 설정한다.

```text
Architecture: X86_64
System: Standard PC (Q35 + ICH9, 2009)
        (alias of pc-q35-9.1) (q35)
Memory: 4096 MiB
CPU: 2
Enable Hardware OpenGL Acceleration: Off
```

7. **Storage**의 디스크 용량은 **45 GiB**로 지정한다.
8. **Shared Directory**는 설정하지 않고 진행한다. 공유 폴더는 필수가 아니며 설치 후에도 구성할 수 있다.
9. **Summary**에서 VM 이름을 **Ubuntu 24.04 Server**로 정하고 설정을 확인한 뒤 **Save**한다.

CPU와 RAM은 호스트의 자원을 게스트에 할당하는 것이다. 위 값은 강의 예시이며 모든 컴퓨터에 대한 고정 권장값은 아니다. 호스트에서 사용할 자원을 남겨 두고 조정해야 한다.

### 8페이지: Ubuntu Server 설치

Server 설치 화면은 방향키로 항목을 선택하고 `Enter`로 확정한다. 버튼 이동에는 `Tab`을 사용할 수 있다.

1. VM의 재생 버튼으로 부팅하고 **Try or Install Ubuntu Server**를 선택한다.
2. 언어는 **English**로 선택한다.
3. 키보드 레이아웃은 기본 영어 설정을 유지하고 **Done**으로 진행한다.
4. 설치 종류는 **Ubuntu Server**를 선택한다. **Ubuntu Server (minimized)**도 선택지로 제시되지만 강의는 일반 Server 구성을 사용한다.
5. 네트워크 설정은 변경하지 않고 **Done**을 선택한다.
6. 프록시는 입력하지 않고 진행한다.
7. 패키지 저장소의 **Mirror address**는 기본 URL을 유지한다.
8. 저장소와 파티션은 별도 사용자 설정 없이 기본값으로 진행한다. 화면 예시는 가상 디스크 전체를 사용하고 **LVM group**을 구성하며 **LUKS 암호화는 선택하지 않은** 상태다.
9. 파티션 요약을 확인하고 **Done → Continue**로 진행한다. 강의 화면에는 약 45G의 가상 디스크, `/`의 LVM 논리 볼륨 약 20.972G, `/boot` 2G, `/boot/efi` 약 1.049G가 표시된다. 이는 해당 설치 예시의 배치이며 다른 환경에서 정확히 같은 크기가 나와야 하는 조건은 아니다.
10. 로그인 계정 정보를 입력한다. 강의 예시의 이름·사용자명은 **user**, 서버명은 **user-utm**이다. 비밀번호와 확인란에는 **자신이 정한 비밀번호**를 입력한다.
11. Ubuntu Pro는 **Skip for now → Continue**로 건너뛴다.
12. OpenSSH 서버 설치는 이 단계에서 선택하지 않고 **Done**으로 진행한다. SSH가 필요해지는 다음 페이지에서 설치한다.
13. 추가 Snap 패키지는 선택하지 않고 **Done**으로 진행한다. Snap은 `apt`와 별개로 프로그램을 설치하는 패키지 방식이며, 기본 Server 환경을 만드는 이 단계에서는 추가 선택이 필수가 아니다.
14. 다운로드와 설치를 기다린다. 강의는 **10분 이상** 걸릴 수 있다고 설명한다. 완료되면 **Reboot now**를 선택한다.
15. 설치 미디어를 제거하라는 메시지가 나오면, 부팅용 ISO를 가상 CD/DVD에서 분리한다. 강의의 절차는 UTM에서 해당 VM을 우클릭해 **Stop**, **[CD/DVD] → Clear**로 ISO를 제거하고 `empty` 표시를 확인한 뒤 다시 재생하는 것이다.
16. ISO가 남아 있어 설치 초기 화면으로 다시 부팅된다면 설치를 반복하지 말고 미디어 연결을 확인한다.
17. 처음 만든 사용자명과 비밀번호로 로그인한다. 강의 예시의 셸 프롬프트는 다음과 같다.

```text
user@user-utm:~$
```

> 원문에 표시된 비밀번호는 공개된 실습 예시다. 실제 환경의 비밀번호로 그대로 재사용하지 않고 직접 정한다.

### 9페이지: SSH로 접속해 복사·붙여넣기 사용

UTM의 게스트 콘솔에서 호스트와 직접 복사·붙여넣기가 되지 않을 때, Mac 터미널에서 SSH로 접속하면 호스트 터미널의 복사·붙여넣기 기능을 사용할 수 있다.

**Ubuntu 게스트에서 실행**:

```bash
# VM의 네트워크 인터페이스와 IP 주소 확인
ip addr

# SSH 서버가 없다면 설치
sudo apt update
sudo apt install -y openssh-server
```

강의 예시는 `enp0s1` 인터페이스에서 **192.168.64.3**을 확인한다. 인터페이스 이름과 IP 주소는 자신의 VM에서 확인한 값을 사용한다.

**Mac 호스트 터미널에서 실행**:

```bash
ssh <USERNAME>@<UBUNTU_IP>

# 강의의 공개 예시 주소·사용자명
ssh user@192.168.64.3
```

Ubuntu 사용자 계정의 비밀번호를 입력한다. 입력 중 화면에 글자나 별표가 나타나지 않아도 입력은 진행될 수 있다. 접속 후에는 Mac 터미널에서 Ubuntu 셸을 사용하게 된다.

## 3. Windows: VirtualBox와 Ubuntu Desktop — 10~19페이지

### 10페이지: 사용할 환경

VirtualBox는 게스트 운영체제의 실행 환경을 제공하는 VM 소프트웨어로, 강의는 x86 아키텍처 가상화와 하이퍼바이저 역할을 소개한다.

Linux는 공개된 소스와 다양한 도구·명령을 갖춘 환경이고, 서버와 보안 실습에서 널리 쓰인다. Dreamhack의 많은 시스템 해킹·리버싱·웹 해킹 실습도 Linux 서버 환경을 전제로 하므로 Linux를 사용할 수 있는 환경을 마련한다.

- VM 프로그램: **VirtualBox 7.1.4**.
- 게스트: **Ubuntu 24.04 계열 64-bit PC Desktop**.
- 강의의 호스트 기준: **x86-64 Windows**.
- 강의는 macOS·Linux에서도 유사한 흐름으로 구성할 수 있다고 설명하지만, Apple Silicon에서의 아키텍처 차이는 앞의 UTM 설명과 구분한다.

### 11페이지: VirtualBox 다운로드

[VirtualBox 7.1 이전 빌드 목록](https://www.virtualbox.org/wiki/Download_Old_Builds_7_1)에서 호스트 운영체제에 맞는 설치 파일을 받는다. 강의는 **7.1.4의 Windows hosts**를 선택하고 Downloads 폴더에서 EXE 파일을 확인한다.

### 12페이지: VirtualBox 설치

1. EXE를 실행하고 **Next**로 진행한다.
2. 라이선스 내용을 확인하는 화면을 거친다.
3. **Custom Setup**의 기능 구성은 기본값을 유지한다.
4. 네트워크 기능 설치 안내에서 강의는 **Yes**로 진행한다.
5. **Python Core / win32api** 의존성 안내에서도 강의는 **Yes**를 선택한다.
6. 바로가기·파일 연결 옵션은 다음과 같이 선택한다.

```text
Create start menu entries: 선택
Create a shortcut on the desktop: 선택하지 않음
Create a shortcut in the Quick Launch Bar: 선택
Register file associations: 선택
```

7. **Next → Install**로 설치하고 완료 후 **Finish**한다.

**Python 안내의 정확한 의미**: 본문은 Python Core와 win32api를 설치하는 단계처럼 설명하지만, 실제 강의 이미지에는 **Python bindings에 필요한 의존성이 없고, 계속 진행하면 나중에 수동으로 설정해야 한다**는 안내가 나온다. 해당 화면의 **Yes**를 눌렀다는 사실만으로 Python과 win32api까지 자동 설치되었다고 이해하지 않는다.

### 13페이지: Ubuntu Desktop ISO 다운로드

[Ubuntu Desktop 다운로드](https://ubuntu.com/download/desktop/)에서 강의 예시의 **24.04.2 LTS** ISO를 받는다. Server 설치 이미지와 구별한다.

```text
ubuntu-24.04.2-desktop-amd64.iso
```

### 14페이지: VirtualBox 가상 머신 생성

**New**로 VM을 만들고 다음 값을 설정한다.

```text
Name: Ubuntu_2404
Type: Linux
Version: Ubuntu (64-bit)
Base Memory: 4096 MB
Processors: 2
Virtual Hard disk: 45 GB
```

강의는 CPU 2코어·RAM 8GB인 호스트 노트북을 예로 든다. RAM과 CPU를 과도하게 할당하면 호스트 성능이 저하될 수 있으므로 실제 장비의 자원에 맞게 조정한다. **Summary**에서 값을 다시 확인하고 **Finish**한 뒤 VM이 목록에 생겼는지 확인한다. UTM의 디스크 예시는 **45 GiB**, 이 VirtualBox 본문 예시는 **45 GB**로 단위 표기가 다르다.

### 15페이지: VM의 가상 광학 드라이브에 ISO 삽입

아직 OS가 없는 VM은 실제 컴퓨터처럼 설치 미디어로 부팅해야 한다. 여기서는 가상 CD/DVD 장치에 ISO를 연결한다.

1. VM을 선택해 **Settings**를 연다.
2. **Storage**에서 **Controller: IDE**의 광학 드라이브/CD 아이콘을 선택한다.
3. 디스크 이미지 선택 창에서 **Add**를 누른다.
4. 다운로드한 Desktop ISO를 찾아 **Open**한다.
5. 추가된 ISO를 선택해 **Choose**, 설정 창에서 **OK**로 확정한다.

### 16페이지: Ubuntu Desktop 설치

1. 해당 VM의 **Start**를 눌러 부팅한다.
2. **Try or Install Ubuntu**를 선택한다.
3. 설치 언어는 강의 예시에서 **English**다.
4. **Accessibility**는 추가 설정 없이 진행한다. 해당 화면에는 시각·청각·입력·포인팅·확대 설정이 있으며, 필요할 때 조정할 수 있다.
5. 키보드 레이아웃을 선택한다.
6. 인터넷 화면은 강의 이미지의 **Use wired connection** 상태로 진행한다.
7. 업데이트 안내는 **SKIP**으로 건너뛴다. 강의가 설명하는 24.04 설치 흐름을 따른 선택이다.
8. **Install Ubuntu**를 선택한다.
9. 앱 구성은 **Default selection**을 사용한다. **Extended selection**은 선택하지 않는다.
10. 추가 그래픽·Wi-Fi용 제삼자 소프트웨어와 추가 미디어 형식 지원은 선택하지 않는다.
11. 디스크 설치 방식은 화면의 **Erase disk and install Ubuntu**를 유지한다. 고급 기능은 선택하지 않고, 수동 설치도 사용하지 않는다. 여기서 대상은 이 실습을 위해 만든 **VM의 가상 디스크**다.
12. 사용자 계정을 만든다. 강의 예시의 이름·사용자명은 **user**, 컴퓨터 이름은 **user-VirtualBox**다. 비밀번호·확인란에는 자신이 정한 값을 입력한다.
13. 원하는 시간대를 선택한다.
14. 최종 설정을 검토하고 **Install**한다.
15. 설치 후 **Restart Now**를 선택하고, 재부팅 과정의 설치 미디어 관련 안내에 따라 `Enter`로 마친다.

원문의 비밀번호 문자열은 공개 실습 예시이며 이 노트에서는 재사용할 값으로 제시하지 않는다.

### 17페이지: 로그인과 첫 실행 설정

1. 로그인 화면에서 만든 사용자 계정을 선택한다.
2. 자신이 정한 비밀번호로 로그인한다.
3. 첫 실행 안내에서 **Next**로 진행한다.
4. Ubuntu Pro는 **SKIP**으로 건너뛴다.
5. 시스템 데이터 공유는 **No, don't share system data**를 선택한다.
6. **Finish**를 눌러 Desktop 초기 설정을 마친다.

### 18페이지: Guest Additions 설치

Guest Additions는 게스트 OS에 설치하는 드라이버와 프로그램의 모음이다. 게스트 성능과 호스트 연동을 돕고 다음 기능을 제공한다.

- 호스트와 게스트의 공유 폴더.
- 공유 클립보드.
- 드래그 앤 드롭을 통한 파일 전달.
- VM 창 크기에 맞춘 게스트 화면 해상도 변경.

**설치 순서**:

1. VM 창의 **Devices → Insert Guest Additions CD image…**를 선택한다.
2. Ubuntu의 파일 탐색기에서 추가된 CD 아이콘을 연다. 아이콘이 바로 안 보이면 목록을 아래로 확인한다.
3. 강의는 `Shift`를 누른 상태로 해당 위치에서 우클릭해 **Open in Terminal**을 선택한다.
4. 열린 터미널에서 설치 스크립트를 실행하고 계정 비밀번호를 입력한다.

```bash
./autorun.sh
```

`bzip2`가 없다는 오류가 발생하면 강의는 다음 명령으로 설치한 뒤 스크립트를 다시 실행한다.

```bash
sudo apt update
sudo apt install bzip2
./autorun.sh
```

설치가 끝나면 터미널의 안내에 따라 `Enter`를 누르고 재부팅한다.

```bash
sudo reboot
```

재부팅 후 VM 창의 크기를 바꿨을 때 Ubuntu 해상도도 함께 바뀌는지 확인한다.

### 19페이지: 양방향 클립보드와 드래그 앤 드롭

1. VirtualBox에서 해당 VM의 **Settings → General → Advanced**를 연다.
2. **Shared Clipboard**를 **Bidirectional**로 설정한다.
3. **Drag'n Drop**도 **Bidirectional**로 설정한다.
4. **OK**로 저장한다.
5. 강의의 확인 방법은 Windows 메모장의 `hello ubuntu!!!` 문자열을 복사하고 Ubuntu 터미널에서 `Ctrl + Shift + V`로 붙여넣는 것이다.

Guest Additions 설치와 두 기능의 설정은 별도의 단계다. 또한 **Bidirectional**은 호스트와 게스트 양쪽 방향의 전달을 허용한다는 의미다.

## 4. Windows: WSL2와 Ubuntu — 20~22페이지

### 20페이지: WSL2 환경과 Windows 버전 확인

강의는 Windows에서 Ubuntu 24.04를 사용하는 또 다른 방법으로 WSL2를 소개한다. 강의 기준으로 **Windows 10 Version 2004 이상**이면 자원 사용 부담이 적은 WSL2를 제안하고, WSL 사용 중 오류가 반복되면 VirtualBox를 대안으로 설명한다. 이는 강의의 선택 안내이며 모든 환경에서 같은 결과를 보장하는 비교는 아니다.

Windows 버전 확인:

1. `Windows + R`로 실행 창을 연다.
2. `winver`를 입력한다.
3. 표시된 버전을 확인한다. 강의의 실제 설치 예시는 **Windows 11**이다.

### 21페이지: WSL2 설치와 첫 실행

1. `Windows + R`을 누르고 `PowerShell`을 입력한다.
2. 실행 창에서 `Ctrl + Shift + Enter`로 관리자 권한 PowerShell을 연다.
3. 다음 명령으로 WSL에 필요한 기능과 Ubuntu 24.04 배포판을 설치한다.

```powershell
wsl.exe --install -d Ubuntu-24.04
```

4. 변경 사항 적용에 재부팅이 필요하다는 안내가 나오면 **Windows 컴퓨터를 재부팅**한다.
5. 재부팅 후 PowerShell에서 배포판을 실행한다.

```powershell
wsl.exe -d Ubuntu-24.04
```

6. 첫 실행에 나타나는 사용자 계정과 새 비밀번호 입력 단계에서 Linux 사용자 정보를 만든다.
7. Ubuntu 셸에서 배포판 버전을 확인한다.

```bash
lsb_release -a
```

강의의 완료 기준은 Ubuntu의 릴리스가 **24.04**로 나오는 것이다. 이 명령은 Ubuntu 배포판 버전을 확인하는 것이며, WSL 엔진의 1/2 버전을 직접 확인하는 명령으로 해석하지 않는다. 원문은 별도의 WSL 엔진 버전 조회 명령을 다루지 않는다.

### 22페이지: 이후 Ubuntu를 여는 두 방법

- Windows 검색 창에서 **Ubuntu**를 찾아 앱을 실행한다.
- PowerShell에서 다음 명령으로 기본 WSL 배포판의 홈 디렉터리를 연다.

```powershell
wsl ~
```

강의에서는 앞서 설치한 Ubuntu를 사용하는 상황이다. 배포판을 지정해 실행하는 명령 `wsl.exe -d Ubuntu-24.04`와, 기본 배포판을 여는 `wsl ~`의 용도를 구분해 기억한다.

## 5. 명령어 모음과 완료 확인

### Mac + UTM

Ubuntu 게스트에서 IP를 확인하고 SSH 서버를 설치한다.

```bash
ip addr
sudo apt update
sudo apt install -y openssh-server
```

Mac 호스트에서 실제 VM 주소로 연결한다.

```bash
ssh <USERNAME>@<UBUNTU_IP>
```

**완료 확인**: Server ISO가 분리되어 있고, 만든 계정으로 로그인해 셸 프롬프트를 사용할 수 있으며, Mac 터미널의 SSH 세션에서 복사·붙여넣기를 사용할 수 있다.

### Windows + VirtualBox

Guest Additions CD 디렉터리에서 실행한다.

```bash
./autorun.sh
```

`bzip2` 누락 시 설치 후 재시도하고 재부팅한다.

```bash
sudo apt update
sudo apt install bzip2
./autorun.sh
sudo reboot
```

**완료 확인**: Ubuntu Desktop에 로그인할 수 있고, 창 크기 변경에 해상도가 반응하며, Windows에서 복사한 문자열을 Ubuntu 터미널에 붙여넣을 수 있다. 드래그 앤 드롭은 설정이 양방향으로 지정되었는지도 확인한다.

### Windows + WSL2

관리자 PowerShell에서 설치하고, 재부팅 뒤 실행한다.

```powershell
wsl.exe --install -d Ubuntu-24.04
wsl.exe -d Ubuntu-24.04
wsl ~
```

Ubuntu 셸에서 확인한다.

```bash
lsb_release -a
```

**완료 확인**: Linux 계정 생성과 첫 로그인이 완료되고, Ubuntu 24.04가 표시되며, Windows 검색 또는 PowerShell로 Ubuntu를 다시 열 수 있다.

## 6. 복습 체크리스트

- [ ] 호스트 머신과 게스트 머신을 구분할 수 있다.
- [ ] VM이 실습 환경 재현과 비용 절감에 도움이 되는 이유를 설명할 수 있다.
- [ ] 스냅샷의 저장·복구 목적을 이해한다.
- [ ] VM의 환경 분리를 절대적인 보안 보장으로 해석하지 않는다.
- [ ] Apple Silicon의 ARM과 강의 게스트의 AMD64 차이를 설명할 수 있다.
- [ ] 이 UTM 실습에서 Emulate를 선택한 이유를 안다.
- [ ] Ubuntu Server와 Desktop ISO를 구별할 수 있다.
- [ ] UTM과 VirtualBox 예시의 CPU·RAM·디스크 값을 알고 실제 장비에 맞게 조정할 수 있다.
- [ ] 설치 완료 후 ISO를 분리해야 하는 이유를 안다.
- [ ] `ip addr`의 VM 주소로 호스트에서 SSH에 접속할 수 있다.
- [ ] 공개 예시 비밀번호 대신 자신의 비밀번호를 사용한다.
- [ ] Guest Additions와 양방향 공유 설정을 각각 설명할 수 있다.
- [ ] `bzip2` 누락 시 강의가 제시한 복구 절차를 안다.
- [ ] `winver`로 Windows 버전을 확인할 수 있다.
- [ ] WSL 설치 명령과 Ubuntu 배포판 실행 명령을 구분한다.
- [ ] `lsb_release -a`가 Ubuntu 배포판 버전 확인 명령임을 안다.
- [ ] `wsl ~`의 홈 디렉터리 실행 방식을 이해한다.

## 원문 페이지 확인 범위

강의 제목과 함께 다음 22페이지를 모두 읽어 위 절에 대응시켰다.

1. 가상 머신
2. 가상 머신이 필요한 이유
3. 리눅스 VM 구축 - UTM
4. UTM 다운로드
5. UTM 설치
6. Ubuntu ISO 파일 다운로드 — Server
7. 가상 머신 생성 — UTM
8. Ubuntu Linux 24.04 Server 설치
9. SSH로 복사/붙여넣기 이용하기
10. 리눅스 VM 구축 - VirtualBox
11. VirtualBox 다운로드
12. VirtualBox 설치
13. Ubuntu ISO 파일 다운로드 — Desktop
14. 가상 머신 생성 — VirtualBox
15. ISO 파일 삽입
16. Ubuntu 24.04 Desktop 설치
17. 계정 로그인
18. VirtualBox Guest Additions 설치
19. 복사/붙여넣기, 드래그&드랍 활성화
20. 리눅스 VM 구축 - WSL2
21. WSL2 설치
22. 우분투 실행 방법

원 강의의 화면·설치 미디어·정확한 문맥은 [Dreamhack 원문](https://learn.dreamhack.io/756)에서 확인한다. 이 문서는 문장과 이미지를 그대로 옮긴 강의 복제본이 아니라 개인 학습을 위한 요약이다.
