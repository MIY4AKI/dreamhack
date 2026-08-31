# Dreamhack Assembly Language — GDB

## 1. GDB와 pwndbg

GDB(GNU Debugger)는 리눅스의 대표적인 디버거다. 프로그램을 실행하며 특정 지점에서 멈추고, 레지스터·메모리·어셈블리 명령·호출 스택을 관찰하거나 값을 바꿀 수 있다. 리버스 엔지니어링에서는 소스 코드 없이 바이너리의 실제 동작을 확인하는 핵심 도구다.

`pwndbg`는 GDB 위에서 동작하는 플러그인으로, 레지스터·디스어셈블리·스택·백트레이스를 보기 좋은 형태로 묶어 보여주며 시스템 해킹에 유용한 명령을 추가한다.

### 설치

```bash
sudo apt-get update
sudo apt-get install -y gdb git
git clone https://github.com/pwndbg/pwndbg
cd pwndbg
./setup.sh
```

설치 방식과 지원 버전은 바뀔 수 있으므로 실제 설치 시에는 pwndbg 공식 저장소의 현재 안내를 우선한다. 터미널에서 `gdb`를 실행했을 때 프롬프트가 `pwndbg>`로 나타나면 플러그인이 로드된 것이다.

비슷한 플러그인으로 GEF, PEDA, Pwngdb 등이 있지만 이 필기는 pwndbg 기준이다.

---

## 2. 실습용 프로그램

```c
// gcc -o debugee debugee.c -no-pie
#include <stdio.h>

int add(int a, int b) {
    return a + b;
}

int main(void) {
    int a = 2;
    int b = 3;
    printf("%d + %d = %d\n", a, b, add(a, b));
    return 0;
}
```

컴파일러 버전, 옵션, PIE 여부에 따라 주소와 생성되는 명령어는 달라질 수 있다. 디버깅에서는 예제의 주소를 외우지 말고 현재 바이너리에서 다시 확인해야 한다.

디버그 정보를 포함하려면 `-g`를 사용한다.

```bash
gcc -g -O0 -no-pie -o debugee debugee.c
```

실전 바이너리에는 심볼과 디버그 정보가 없을 수 있다. `(No debugging symbols found ...)`는 파일 로드 실패가 아니라 소스 수준 심볼 정보가 없다는 뜻이다.

---

## 3. 바이너리 로드와 실행 중 프로세스 연결

```bash
gdb ./debugee
```

또는 GDB 안에서:

```gdb
file ./debugee
```

이미 실행 중인 프로세스에 연결할 때:

```bash
gdb -p <PID>
```

다른 프로세스에 attach하려면 같은 사용자 권한, ptrace 설정 등의 조건이 필요할 수 있다.

---

## 4. 실행 흐름 제어

### 실행

```gdb
run                 # 단축: r
run arg1 arg2       # 프로그램 인자와 함께 실행
start               # main에서 임시 중단
entry               # pwndbg: ELF entry point에서 중단
```

`start`는 보통 `main`부터 보고 싶을 때, `entry`는 `_start`와 런타임 초기화 흐름까지 보고 싶을 때 유용하다.

### 중단점

```gdb
break main           # 단축: b main
break *0x401156      # 주소에 중단점을 걸 때 * 사용
break *main+20
info breakpoints     # 단축: i b
disable 1
enable 1
delete 1             # 단축: d 1
```

주소가 ASLR/PIE에 의해 재배치될 수 있으므로 가능한 경우 `break main`, `break *main+offset`처럼 심볼 기준 표현이 편하다.

### 계속 실행

```gdb
continue             # 단축: c
continue 5           # 현재 중단점을 여러 번 건너뛰는 데 유용
```

### 한 명령씩 추적

| 명령 | 의미 |
|---|---|
| `ni` | next instruction. 한 어셈블리 명령 실행, `call` 내부로 들어가지 않음 |
| `si` | step instruction. 한 어셈블리 명령 실행, `call` 내부로 들어감 |
| `finish` | 현재 함수가 반환할 때까지 실행하고 호출자에서 멈춤 |

함수 내부까지 분석할 필요가 없으면 `ni`, 호출된 함수의 세부 동작이 필요하면 `si`를 사용한다. 너무 깊이 들어갔다면 `finish`로 현재 함수를 빠져나온다.

---

## 5. 레지스터와 중단점 정보

```gdb
info registers       # 단축: i r
info registers rax rip rsp
print/x $rax         # 단축: p/x $rax
break *$rdi          # rdi가 코드 주소라면 그 위치에 중단점
```

GDB에서 레지스터 이름 앞에는 `$`를 붙인다. 함수 반환 직후의 `rax`, 다음 명령 주소인 `rip`, 현재 스택인 `rsp`는 특히 자주 확인한다.

---

## 6. 디스어셈블

```gdb
disassemble main     # 단축: disass main
disassemble 0x401000, 0x401080
```

pwndbg는 다음과 같은 가독성 높은 디스어셈블 명령도 제공한다.

```gdb
u main
nearpc
pdisass main
```

도구 버전에 따라 명령 이름이나 출력 형식이 달라질 수 있다.

---

## 7. 메모리 확인: `x`(examine)

기본 형식:

```text
x/<개수><표현 형식><단위 크기> <주소>
```

### 표현 형식

| 문자 | 표현 |
|---|---|
| `x` | 16진수 |
| `d` | 부호 있는 10진수 |
| `u` | 부호 없는 10진수 |
| `t` | 2진수 |
| `i` | 명령어 |
| `c` | 문자 |
| `s` | 문자열 |
| `f` | 부동소수점 |
| `a` | 주소 |

### 단위 크기

| 문자 | 크기 |
|---|---:|
| `b` | 1바이트(byte) |
| `h` | 2바이트(halfword) |
| `w` | 4바이트(word) |
| `g` | 8바이트(giant word) |

예시:

```gdb
x/10gx $rsp          # rsp부터 8바이트 값 10개를 16진수로
x/5i $rip            # rip부터 명령어 5개
x/s 0x402000         # 해당 주소의 널 종료 문자열
x/16bx $rdi          # rdi부터 바이트 16개
```

`x`는 프로세스의 가상 메모리를 읽으므로 일반적으로 프로그램이 실행되어 주소 공간이 만들어진 뒤 사용한다.

---

## 8. pwndbg의 메모리 도구

### telescope

```gdb
telescope $rsp       # 단축 예: tele $rsp
```

지정한 위치의 포인터를 재귀적으로 따라가며 값, 문자열, 코드 주소 등을 함께 표시한다. 스택에 저장된 `argv`, 반환 주소, 다른 스택 포인터를 빠르게 파악할 때 좋다.

### vmmap

```gdb
vmmap
```

프로세스의 가상 메모리 매핑을 보여준다.

- 시작/끝 주소
- `rwx` 권한
- 매핑 크기와 오프셋
- 실행 파일, libc, 동적 로더, heap, stack 등의 이름

실행 파일과 공유 라이브러리는 ELF 로더에 의해 메모리에 매핑된다. `printf` 같은 함수는 libc 매핑에서 실행될 수 있다.

---

## 9. 호출 흐름: backtrace

```gdb
backtrace             # 단축: bt
```

현재 함수가 어떤 함수들로부터 호출됐는지 콜 스택을 보여준다. 충돌 위치에서 버그의 입력이 어디에서 왔는지 역으로 추적할 때 유용하다.

```text
#0 add
#1 main
#2 __libc_start_call_main
#3 __libc_start_main_impl
#4 _start
```

최적화, 프레임 포인터 생략, 스택 손상, 심볼 제거 때문에 완전한 이름이나 프레임을 얻지 못할 수도 있다.

---

## 10. 메모리 덤프

```gdb
dump memory <파일명> <시작주소> <끝주소>
```

예:

```gdb
entry
vmmap
dump memory code.bin 0x401000 0x402000
```

`vmmap`으로 정확한 범위와 권한을 확인한 뒤 덤프해야 한다. 끝 주소는 보통 포함되지 않는 경계로 취급한다.

---

## 11. context

pwndbg의 `context`(`ctx`)는 중단 시점의 핵심 정보를 한 화면에 정리한다.

- `REGISTERS`: 레지스터 값과 변경 표시
- `DISASM`: `rip` 주변 명령어
- `STACK`: `rsp` 주변 값과 포인터 해석
- `BACKTRACE`: 현재 호출 경로

```gdb
context
ctx
```

명령 한 줄을 실행한 뒤 context에서 어떤 레지스터·메모리·플래그가 바뀌었는지 비교하는 습관이 중요하다.

---

## 12. 값 변경: `set`

레지스터 변경:

```gdb
set $rax = 0
set $rsp = $rbp
```

메모리 변경:

```gdb
set *(unsigned int *)0x404000 = 10
set *(float *)0x404010 = 3.14
```

검증:

```gdb
x/wu 0x404000
x/wf 0x404010
```

자료형 캐스팅은 몇 바이트를 어떤 형식으로 기록할지 결정한다. 잘못된 주소나 크기에 값을 쓰면 디버깅 대상이 즉시 충돌하거나 분석 상태를 망칠 수 있으므로 변경 전 원래 값을 기록해 둔다.

---

## 13. 추천 디버깅 순서

```gdb
file ./debugee
start
disassemble main
break *main+<관심 오프셋>
continue
context
info registers
x/16gx $rsp
ni
si
backtrace
vmmap
```

관찰 → 가설 → 한 단계 실행 → 변화 확인을 반복한다. 주소와 출력값을 강의 예시 그대로 기대하지 말고 자신의 바이너리에서 확인한다.

## 빠른 명령표

| 목적 | 명령 |
|---|---|
| 파일 로드 | `file ./a.out` |
| 실행 | `run`, `start`, `entry` |
| 중단점 | `b main`, `b *ADDR`, `i b`, `d N` |
| 계속 | `c` |
| 한 명령 실행 | `ni`, `si` |
| 현재 함수 탈출 | `finish` |
| 레지스터 | `i r`, `p/x $rax` |
| 디스어셈블 | `disass`, `u`, `nearpc` |
| 메모리 | `x/...`, `tele` |
| 매핑 | `vmmap` |
| 호출 스택 | `bt` |
| 상태 화면 | `context` |
| 값 변경 | `set` |
| 메모리 파일 저장 | `dump memory` |

## 출처 및 이용 안내

이 문서는 Dreamhack 강의를 학습 목적으로 요약·재구성한 개인 학습 노트다. 원문과 실습 파일은 [GDB](https://learn.dreamhack.io/763)에서 확인한다.

