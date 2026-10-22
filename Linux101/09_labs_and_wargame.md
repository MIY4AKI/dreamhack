# Linux 101 — Lab 5개와 baby-linux 학습 안내

> 확인일: 2026-09-30 · 범위: 각 Lab의 제공 과제·설명과 워게임 문제 안내.
> 5개 Lab의 **24개 과제**를 읽고 정리했다. 아래 명령은 과제 설명에서 도출한 학습 예시이며, 실제 정답 제출이나 터미널 실행 결과가 아니다. 동적으로 생성되는 파일 내용·PID·플래그·정답 값은 추측하지 않았다.
> Lab 페이지를 방문하면 사이트가 임시 환경을 자동 준비한다. 이 문서 작성 과정에서는 실습 터미널에 명령을 입력하거나 정답을 제출하지 않았다. 일부 해설은 읽기 전용 확인 과정에서 펼쳐지지 않아 모든 공식 해설을 확인했다고 주장하지 않는다.

## 목차

- [1. Directory Lab — 6과제](#directory)
- [2. File Lab — 9과제](#file)
- [3. Process Lab — 4과제](#process)
- [4. File Descriptor Lab — 3과제](#fd)
- [5. User Lab — 2과제](#user)
- [6. baby-linux 워게임](#wargame)
- [7. 실습 환경 사용 시 주의](#lab-notes)

<a id="directory"></a>
## 1. Directory Lab — 6과제

출처: [Lab: Linux - Directory](https://learn.dreamhack.io/labs/17f2b0ed-ee87-407d-ac56-ad88db265d06)

### 과제와 확인 방법

1. **`pwd`**: 로그인 후 홈 디렉터리의 절대 경로를 확인한다. 답은 사용자별 실제 출력이며 `/home/user`라고 고정하지 않는다.
2. **`ls`**: 홈에서 `flag_*` 형태의 파일을 찾고 **전체 파일명**을 확인한다.
3. **`ls -a`**: 기본 목록에서는 숨기는 점(`.`) 시작 항목을 포함해 `.hidden_flag_*`의 전체 이름을 찾는다.
4. **`ls -al`**: 위 숨김 파일의 크기를 **바이트 수, 단위 없이** 확인한다. `-h`의 사람이 읽기 좋은 단위와 구별한다.
5. **`mkdir`**: 홈 아래에 `hello_dir`를 만든다.
6. **`rmdir`**: 홈의 `remove_dir`를 삭제한다. `rmdir`는 비어 있는 디렉터리에 사용하는 명령이다.

```bash
# Lab 터미널용 예시 — 실제 출력에서 파일명과 크기를 확인
cd ~
pwd
ls
ls -a
ls -al
mkdir hello_dir
rmdir remove_dir
```

### Lab에서 소개하는 `ls` 옵션 전체

- `-a`: 숨김 항목을 포함한 모든 이름.
- `-l`: 상세 목록 형식.
- `-Z`: SELinux 보안 문맥 정보.
- `-s`: 할당된 파일시스템 공간을 블록 단위로 표시.
- `-S`: 파일 크기 기준 정렬.
- `-h`: 크기를 K, M 등 읽기 좋은 단위로 표시. 보통 `-l`이나 `-s`와 결합한다.
- `-i`: inode 번호 표시.
- `-R`: 하위 디렉터리를 재귀적으로 조회.

`-al`처럼 짧은 옵션은 합쳐 쓸 수 있다. `ls -l`의 열은 종류·권한, 링크 수, 소유자, 소유 그룹, 바이트 크기, 시각, 이름 순으로 읽는다.

> 보충·정정: Lab의 `-s` 설명은 간단히 파일 크기라고 되어 있으나, 논리적인 바이트 크기(`-l`의 크기 열)와 할당 블록 수는 다를 수 있다. [GNU ls](https://www.gnu.org/s/coreutils/manual/html_node/ls-invocation.html).

<a id="file"></a>
## 2. File Lab — 9과제

출처: [Lab: Linux - File](https://learn.dreamhack.io/labs/3eb7a87d-f0fb-45ab-8b8a-1ad206218a75)

### 1) `touch`

홈 아래 `touch_file`을 만든다. 직접 열람한 첫 공식 해설도 `cd ~` 후 `touch touch_file`의 순서를 안내한다.

### 2) `cat`

홈의 `cat_file` 내용을 읽어 결과를 확인한다. `cat`은 여러 입력을 연결할 수 있는 명령이지만 이 Lab은 단일 파일 읽기를 연습한다.

### 3) `head`

`head_file`의 **앞쪽 53바이트**를 읽는다. 바이트 단위이므로 줄 수 옵션과 혼동하지 않는다.

### 4) `tail`

`tail_file`의 **뒤쪽 100바이트**를 읽는다.

```bash
cd ~
touch touch_file
cat cat_file
head -c 53 head_file
tail -c 100 tail_file
```

일부 파일에는 ANSI escape sequence처럼 터미널 표시를 제어하는 문자가 들어 있다. 전체를 `cat`으로 출력하면 화면 내용이 바뀌어 원하는 답이 그대로 보이지 않을 수 있다. 파일의 바이트와 터미널에 최종 표시된 화면은 같은 것이 아닐 수 있음을 배우는 과제다.

### 5) `mv`

`mv_file`을 `mv_file2`로 이름 변경한다.

### 6) `cp`

`cp_file`을 복사하여 `cp_file2`를 만든다.

### 7) `cp -r`

`cp_dir`의 모든 내용을 재귀 복사하여 `cp_dir2`를 만든다.

```bash
mv mv_file mv_file2
cp cp_file cp_file2
cp -r cp_dir cp_dir2
```

이 세 과제에서는 원본 파일·디렉터리의 내용을 임의로 편집하지 않는다. 과제 성공 검사가 원래 내용을 기준으로 할 수 있다. `mv`는 원본 경로가 이동하고 `cp`는 원본을 유지한다.

### 8) `rm`

홈의 `rm_file`만 삭제한다.

### 9) `rm -rf`

홈의 `rm_dir`과 하위 파일을 제거하는 과제다. `-r`은 재귀, `-f`는 강제 처리이며 확인 질문을 생략할 수 있다.

```bash
# 이 Lab이 제공하는 삭제 대상인지 pwd와 ls로 먼저 확인
rm rm_file
rm -rf ./rm_dir
```

**일반 PC의 홈·루트·프로젝트 전체에 실행하지 않는다.** 특히 잘못 지정한 경로와 와일드카드는 큰 데이터 손실로 이어질 수 있다.

<a id="process"></a>
## 3. Process Lab — 4과제

출처: [Lab: Linux - Process](https://learn.dreamhack.io/labs/81abd8b5-83ad-4972-a266-77928f3b28d0)

### 과제 1. 프로그램 생성 및 실행

홈에 `Hello World!`를 출력하는 ELF 실행 파일 `hello`를 만든다. C 소스와 컴파일 결과는 서로 다른 파일이다. `gcc -o [실행 파일명] [소스 파일명]`의 인자 순서를 익힌다.

```c
#include <stdio.h>

int main(void) {
    puts("Hello World!");
    return 0;
}
```

위 내용을 `hello.c`로 작성했다고 가정한 명령 예시:

```bash
gcc -o hello hello.c
file hello
./hello
```

### 과제 2. 프로세스 상태

`ps_test`의 **Process State Code**를 `ps` 출력에서 확인한다. `ps -el`로 전체 상세 목록을 확인하고 이름에 해당하는 행을 찾아 상태 열을 읽는다. 상태 값을 추측하거나 `htop`의 표시 형식과 혼동하지 않는다.

### 과제 3. PID

`ps_pid_test`의 PID를 확인한다. PID는 해당 프로세스의 ID, PPID는 부모 프로세스의 ID다. 실행 환경마다 바뀌므로 고정 숫자 답은 없다.

```bash
ps -el
```

### 과제 4. 강제 종료

`kill_test_proc`의 PID를 먼저 찾은 뒤 9번 시그널(SIGKILL)을 보내는 과제다.

```bash
# 숫자는 실제 Lab 대상 프로세스의 PID로 치환
kill -9 LAB_TARGET_PID
```

`kill`은 이름과 달리 시그널을 보내는 명령이다. 평상시에는 정리 기회를 주는 SIGTERM을 먼저 고려하지만 이 과제는 SIGKILL 사용을 요구한다. 실제 서비스나 다른 사용자의 프로세스를 종료하지 않는다.

### 소개된 `ps` 옵션과 보충·정정

원문의 옵션을 빠뜨리지 않되 Ubuntu의 procps 기준으로 의미를 정리했다.

- `-A`, `-e`: 모든 프로세스 선택. 원문처럼 항상 커널 프로세스가 제외된다고 볼 수 없다.
- `-a`: 세션 리더와 터미널에 연결되지 않은 프로세스를 제외한다. 원문의 비터미널 관련 설명과 반대임에 주의.
- `-f`: 전체 상세 형식, `-l`: 긴 형식.
- `-M`: 보안 정보. 원문의 64비트 프로세스 선택 설명은 Ubuntu procps와 맞지 않는다.
- `-m`: 프로세스 뒤에 스레드 표시. 커널 스레드만 선택하는 의미가 아니다.
- `-p PID`: PID로 선택.
- `-u USER`: 유효 사용자 ID/이름으로 선택.
- 원문 `-r`: 실행 중 선택으로 소개하지만, procps에서 이 목적의 BSD 옵션은 **대시 없는 `r`**다. 대시 유무와 구현별 차이를 확인한다.

기본 `ps`는 보통 같은 사용자·현재 터미널 중심의 스냅샷을 보여 준다. `ps aux`와 `ps -aux`도 같은 형식으로 무조건 취급하지 않는다. [Ubuntu ps 매뉴얼](https://manpages.ubuntu.com/manpages/noble/man1/ps.1.html).

### Lab의 시그널 목록

아래 번호는 강의의 Linux x86 계열 예시다. 시그널 번호는 아키텍처에 따라 다를 수 있으며 `kill -l`과 이름을 확인한다.

- 1 SIGHUP: 터미널 연결 종료 등. 기본 종료.
- 2 SIGINT: `Ctrl+C`와 같은 인터럽트. 기본 종료.
- 3 SIGQUIT: 종료 요청. 기본 코어 덤프를 동반한 종료.
- 4 SIGILL: 잘못된 명령 실행. 기본 코어 덤프 종료.
- 5 SIGTRAP: 중단점·추적 트랩. 기본 코어 덤프 종료.
- 6 SIGABRT: `abort()` 등에 의한 중단. 기본 코어 덤프 종료.
- 7 SIGBUS: 잘못된 메모리 접근·정렬 등 버스 오류. 기본 코어 덤프 종료.
- 8 SIGFPE: 잘못된 산술 연산. 기본 코어 덤프 종료.
- 9 SIGKILL: 강제 종료.
- 11 SIGSEGV: 잘못된 메모리 참조. 기본 코어 덤프 종료.
- 13 SIGPIPE: 읽는 쪽이 없는 파이프 등에 쓰기. 기본 종료.
- 14 SIGALRM: 설정된 타이머 만료. 기본 종료.
- 15 SIGTERM: 종료 요청. 기본 종료.
- 17 SIGCHLD: 자식의 상태 변경을 부모에게 통지. 기본 무시.
- 19 SIGSTOP: 프로세스 정지.
- 31 SIGSYS: 잘못된 시스템 콜. Linux의 기본 동작은 코어 덤프 종료이므로 원문의 단순 종료 표기를 보충한다.

코어 파일은 제한·설정에 따라 실제 디스크에 생성되지 않을 수도 있다. 시그널의 기본 동작과 프로그램이 설치한 핸들러는 구별하며 SIGKILL/SIGSTOP은 포착·차단할 수 없다. [Linux signal(7)](https://man7.org/linux/man-pages/man7/signal.7.html).

<a id="fd"></a>
## 4. File Descriptor Lab — 3과제

출처: [Lab: Linux - File Descriptor](https://learn.dreamhack.io/labs/ce3230e0-de58-474f-8661-6c5e9b72cf7e)

### 과제 1. 셸에 열린 FD 개수

각 프로세스는 자신의 파일 디스크립터 테이블을 갖는다. `/proc/PID/fd`는 열린 FD를 번호별 심볼릭 링크로 보여 주는 인터페이스다. 이는 커널 테이블이 디스크 파일로 저장된다는 뜻은 아니다.

```bash
# Bash의 $$는 현재 셸 PID를 가리킨다.
ls -l /proc/$$/fd
```

이 목록에서 해당 셸에 열린 FD 수를 확인한다. 0, 1, 2만 있을 것이라고 답을 고정하지 않는다. Lab은 `/proc/self/fd`도 소개하지만, **외부 프로그램 `ls`가 이 경로를 읽으면 self는 ls 자신**이다. 셸의 FD를 묻는 과제에는 `/proc/$$/fd`처럼 대상 PID를 명확히 한다.

### 과제 2. Pipe & grep

`cat`, 파이프, `grep`을 조합하여 `pipe_grep_file`에서 **대문자 I 다섯 개(`IIIII`)로 시작하고 소문자 l 다섯 개(`lllll`)로 끝나는 문자열**을 찾는다. I와 l의 글꼴이 비슷하므로 복사 전에 구별한다.

```bash
cat pipe_grep_file | grep -o 'IIIII.*lllll'
```

`-o`는 일치 부분만, 기본 `grep`은 일치하는 행 전체를 보여 준다. Lab의 패턴은 큰 행 안에 포함된 부분 문자열을 추출하는 데 쓰인다. 만약 **행 전체**의 시작·끝을 검사하려면 `^IIIII.*lllll$`처럼 앵커를 추가하는 별도 조건이 된다.

제공된 `grep` 옵션 전체:

- `-c`: 일치하는 **행 수**.
- `-o`: 일치한 부분만 출력.
- `-i`: 대소문자 구별 없이 검색.
- `-h`: 출력에 파일명 접두어를 붙이지 않음.
- `-l`: 일치가 있는 파일의 이름만 출력.
- `-v`: 일치하지 않는 행 출력.
- `-r`: 하위 디렉터리의 파일을 재귀 검색. 모든 내용을 무조건 출력하는 옵션은 아니다.
- `-n`: 행 번호 표시.
- `-w`: 단어 단위 일치.

제공된 정규식 기호 전체:

- `^x`: 행 시작의 x, `x$`: 행 끝의 x.
- `x+`: x가 1번 이상, `x?`: 0번 또는 1번, `x*`: 0번 이상.
- `.`: 보통 줄바꿈을 제외한 임의 문자 하나.
- `(x)`: 그룹, `[xy]`: x 또는 y 한 글자, `[^x]`: x가 아닌 한 글자.

> 보충: 기본 `grep`은 BRE다. 표의 `+`, `?`, `()`를 저 형태 그대로 사용하려면 `grep -E`로 ERE를 선택하거나 BRE용 이스케이프 규칙을 사용해야 한다. 셸의 파일명 와일드카드와 정규식도 다르다. [GNU grep 정규식](https://www.gnu.org/s/grep/manual/grep.html#Regular-Expressions).

### 과제 3. Redirection

`redirect_test`의 PID를 `ps`로 찾고, 그 프로세스의 **4번 FD**에 `hello redirection!` 뒤 **널 바이트(`\x00`)**를 보내는 과제다.

```bash
ps -el | grep redirect_test
# TARGET_PID를 실제 Lab PID로 바꿔 사용. 다음은 명령 구조 예시다.
printf 'hello redirection!\0' > /proc/TARGET_PID/fd/4
```

Lab의 안내는 `echo`와 `-e`를 이용한 escape 해석이다. Bash에서는 `echo -ne 'hello redirection!\x00'`처럼 `-n`으로 불필요한 줄바꿈도 막을 수 있다. `printf`는 출력 바이트를 명확하게 표현하는 대안이다.

`/proc/PID/fd/N`에 접근하려면 권한과 해당 FD의 대상 종류가 맞아야 한다. 모든 프로세스의 모든 FD에 임의로 쓸 수 있는 기능은 아니며, 이 과제의 지정된 Lab 프로세스에만 사용한다.

<a id="user"></a>
## 5. User Lab — 2과제

출처: [Lab: Linux - User](https://learn.dreamhack.io/labs/c256e8db-5cdc-4d7d-a199-5470c80a765d)

### 과제 1. 일반 권한

홈에 `chmod_file`을 만들고 권한을 `-rw--wxr-x`로 바꾼다.

- 종류 `-`: 일반 파일.
- 소유자 `rw-`: 6.
- 그룹 `-wx`: 3.
- 기타 `r-x`: 5.
- 특수 비트 없음: 0. 전체 모드는 **0635**.

```bash
cd ~
touch chmod_file
chmod 0635 chmod_file
ls -l chmod_file
```

일반 권한은 `r=4`, `w=2`, `x=1`이고 숫자는 비트 OR 또는 합으로 구성한다. `chmod u+x` 같은 기호 형식과 숫자 형식을 구별한다.

### 과제 2. 특수 비트

앞 파일에 **setuid와 sticky bit**를 추가한다. setuid 4, setgid 2, sticky 1이므로 요구된 특수 자리의 값은 `4 | 1 = 5`다.

```bash
chmod u+s,o+t chmod_file
# 일반 권한 635를 유지하는 숫자 표현: chmod 5635 chmod_file
ls -l chmod_file
```

소유자 실행 비트가 없어서 setuid는 대문자 `S`, 기타 실행 비트가 있어서 sticky는 소문자 `t`로 표시된다. 예상 권한 문자열은 `-rwS-wxr-t`다.

이 과제는 비트 설정과 표시를 익히는 것이다. Linux에서 일반 파일에 sticky bit를 붙였다고 삭제 보호가 생기는 것은 아니며, 공유 디렉터리의 삭제 제한과 구별해야 한다.

<a id="wargame"></a>
## 6. baby-linux 워게임

출처: [baby-linux 문제 안내](https://dreamhack.io/wargame/challenges/837)

- 리눅스 명령어를 실행하는 웹 서비스가 제공된다.
- 첨부 파일에 서비스 코드가 들어 있다.
- 목표는 `flag.txt`를 찾아 내용을 출력하는 것이다.
- 플래그 형식은 `DH{...}`다.
- 페이지에는 문제 파일 다운로드, VM 서버 생성, 플래그 제출 기능이 있다. 서버 부팅에는 시간이 걸릴 수 있다.
- Dreamhack CTF Season 3 Round #2 (Div2)에 출제된 문제라는 안내가 있다.

학습 연결점은 경로 탐색, 파일 목록·내용 확인, 서비스가 명령을 처리하는 방식 이해다. **이 노트는 문제 안내 요약이지 첨부 코드 분석이나 검증된 풀이 보고서가 아니다.** 서버 생성·파일 다운로드·플래그 제출을 하지 않았고 실제 플래그를 만들어 넣지 않았다.

<a id="lab-notes"></a>
## 7. 실습 환경 사용 시 주의

- Lab은 설명과 별도 터미널을 함께 제공하고 과제 진행 상태를 표시한다.
- 페이지에는 제한 시간, 환경 연장, 강제 새로고침, 강제 재시작 기능이 있다. 재시작은 작성 중인 작업을 잃게 할 수 있다.
- 답 입력형 과제는 실제 환경에서 관찰한 값이 필요하다. 노트의 예시 문자열이나 PID 자리표시자를 답으로 제출하지 않는다.
- 자동 검사형 과제는 경로·파일명·내용을 요구대로 유지한다.
- 실습용 삭제·시그널·FD 쓰기 명령을 일반 PC나 타인의 시스템에 적용하지 않는다.
- 사이트 기준 Lab은 Path 진도율에 포함되지 않는다.

---

[Linux 101 전체 목차](README.md) · 다음 과정: [Linux 201](../Linux201/README.md)
