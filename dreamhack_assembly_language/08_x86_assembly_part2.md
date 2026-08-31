# Dreamhack Assembly Language — x86 Assembly Essential Part (2)

## 1. 강의 목표

Part 2는 함수가 호출되고 돌아오는 과정과 사용자 프로그램이 커널 기능을 요청하는 시스템 콜을 다룬다.

- 스택과 `push`, `pop`
- 함수 호출과 `call`, `leave`, `ret`
- x86/x86-64 함수 작성과 호출
- 사용자 모드와 커널 모드
- `syscall`과 `int 0x80`

---

## 2. 스택과 `rsp`

스택은 LIFO(Last In, First Out) 자료구조다. x86-64 프로세스에서는 보통 높은 주소에서 낮은 주소 방향으로 자라며 `rsp`가 현재 최상단을 가리킨다.

### `push`

```asm
push value
```

개념적 동작:

```text
rsp = rsp - 8
[rsp] = value
```

64비트 모드의 일반적인 `push`는 스택을 8바이트 이동시킨다.

### `pop`

```asm
pop rax
```

개념적 동작:

```text
rax = [rsp]
rsp = rsp + 8
```

`pop`은 값을 읽은 뒤 `rsp`를 이동할 뿐 과거 스택 바이트를 0으로 지우지 않는다. 해당 영역은 스택 범위 밖이 되어 더 이상 현재 값으로 간주하지 않을 뿐이다.

---

## 3. 함수, caller, callee

함수는 특정 동작을 묶은 코드 블록이다. 어셈블리에서는 라벨로 함수 시작점을 정의하고 다음 두 역할을 구분한다.

- **Caller**: 다른 함수를 호출하는 쪽
- **Callee**: 호출되어 실행되는 함수

함수 호출에는 다음 약속이 필요하다.

- 인자를 어디에 둘 것인가?
- 반환 주소를 어떻게 보관할 것인가?
- 어떤 레지스터를 누가 보존할 것인가?
- 반환값을 어디에 둘 것인가?
- 스택을 누가 정리할 것인가?

이 약속이 호출 규약(Calling Convention/ABI)이다.

---

## 4. `call`

```asm
call target
```

개념적 동작:

```text
push address_of_next_instruction
rip = target
```

즉 `call`은 단순한 `jmp`가 아니다. 함수가 끝난 뒤 돌아올 **반환 주소**를 스택에 저장하고 대상 주소로 분기한다.

```asm
0x400000: call add
0x400005: mov rdi, rax     ; 반환 후 이어서 실행할 위치
```

---

## 5. 스택 프레임과 `leave`

함수마다 지역 변수, 저장 레지스터 등을 위한 독립된 스택 영역을 논리적으로 스택 프레임이라 한다.

전형적인 프롤로그:

```asm
push rbp
mov  rbp, rsp
sub  rsp, 0x20
```

전형적인 에필로그:

```asm
leave
ret
```

`leave`의 개념적 동작:

```text
rsp = rbp
rbp = pop()
```

즉 현재 함수가 확보했던 스택 공간을 버리고 호출자의 프레임 포인터를 복원한다. 최적화된 함수는 `rbp`를 일반 레지스터로 사용하고 프레임 포인터를 생략할 수 있으므로 모든 함수가 `leave`를 쓰는 것은 아니다.

---

## 6. `ret`

```asm
ret
```

개념적 동작:

```text
rip = [rsp]
rsp = rsp + 8
```

스택 최상단의 반환 주소를 꺼내 `rip`에 넣는다. 스택 메모리 손상으로 반환 주소가 바뀌면 제어 흐름도 바뀔 수 있어 Stack Buffer Overflow와 ROP의 핵심 공격 대상이 된다.

---

## 7. 덧셈 함수 작성

### 32비트 x86 cdecl

인자를 스택으로 전달한다.

```asm
add_two:
    push ebp
    mov  ebp, esp
    mov  eax, [ebp+8]      ; 첫 번째 인자
    add  eax, [ebp+12]     ; 두 번째 인자
    leave
    ret
```

호출:

```asm
push dword 20
push dword 10
call add_two
add  esp, 8               ; caller가 인자 공간 정리
```

### 64비트 System V AMD64

첫 번째 두 정수 인자는 `rdi`, `rsi`, 반환값은 `rax`다.

```asm
add_two:
    mov rax, rdi
    add rax, rsi
    ret

_start:
    mov rdi, 10
    mov rsi, 20
    call add_two
    ; rax = 30
```

작은 leaf 함수는 지역 변수나 보존할 레지스터가 없다면 프롤로그 없이 바로 계산하고 `ret`할 수 있다.

---

## 8. 호출의 전체 흐름

```text
Caller
  1. 호출 규약에 따라 인자를 레지스터/스택에 배치
  2. call → 반환 주소를 스택에 push, callee로 분기

Callee
  3. 필요하면 프롤로그로 프레임과 보존 레지스터 준비
  4. 함수 본문 실행
  5. 반환값을 rax 등에 배치
  6. 에필로그로 스택/레지스터 복원
  7. ret → 반환 주소를 rip로 복원

Caller
  8. 호출 규약이 요구하면 인자용 스택 정리
  9. 반환값 사용
```

System V AMD64에서 외부 함수를 호출할 때는 스택 정렬 규칙도 지켜야 한다. 특히 libc 함수와 연동할 때 호출 직전 16바이트 정렬 조건을 확인한다.

---

## 9. 사용자 모드와 커널 모드

### 사용자 모드

일반 프로그램이 실행되는 제한된 권한 수준이다. 임의의 물리 메모리나 하드웨어를 직접 제어할 수 없다. `root` 사용자의 프로세스도 CPU 관점에서는 일반적으로 사용자 모드에서 실행되며, 필요한 작업을 시스템 콜로 커널에 요청한다.

### 커널 모드

운영체제 커널이 시스템 전체를 관리하는 높은 권한 수준이다. 메모리 관리, 파일 시스템, 장치, 네트워크, 프로세스 제어 등을 수행한다.

### 시스템 콜

사용자 프로그램이 커널에 파일 읽기·쓰기, 메모리 매핑, 프로세스 종료 등의 기능을 요청하는 인터페이스다.

```text
User program
    │ syscall number + arguments
    ▼
Kernel
    │ validate and perform operation
    ▼
return value / negative error code
```

---

## 10. 32비트 Linux 시스템 콜

32비트 x86에서는 전통적으로 `int 0x80`을 사용한다.

| 역할 | 위치 |
|---|---|
| 시스템 콜 번호 | `eax` |
| 인자 1~6 | `ebx`, `ecx`, `edx`, `esi`, `edi`, `ebp` |
| 반환값 | `eax` |

```asm
section .data
filename db "dreamhack.txt", 0

section .text
global _start
_start:
    mov eax, 5          ; 32-bit open
    mov ebx, filename
    xor ecx, ecx        ; O_RDONLY
    xor edx, edx
    int 0x80
```

32비트와 64비트 Linux의 시스템 콜 번호는 서로 다르다.

---

## 11. 64비트 Linux 시스템 콜

64비트 x86에서는 `syscall` 명령을 사용한다.

| 역할 | 위치 |
|---|---|
| 시스템 콜 번호 | `rax` |
| 인자 1~6 | `rdi`, `rsi`, `rdx`, `r10`, `r8`, `r9` |
| 반환값 | `rax` |

> System V **함수 호출**의 4번째 인자는 `rcx`지만 Linux x86-64 **시스템 콜**의 4번째 인자는 `r10`이다. `syscall` 명령이 `rcx`와 `r11`을 덮어쓰기 때문에 생기는 중요한 차이다.

### write 예시

```asm
section .data
msg db "Hello World", 10
len equ $ - msg

section .text
global _start
_start:
    mov eax, 1          ; write
    mov edi, 1          ; stdout
    lea rsi, [rel msg]
    mov edx, len
    syscall

    mov eax, 60         ; exit
    xor edi, edi
    syscall
```

개념적으로 `write(1, msg, len)`과 `exit(0)`을 호출한다.

### 주요 x86-64 Linux 시스템 콜

| 이름 | 번호 | 핵심 인자 |
|---|---:|---|
| `read` | 0 | fd, buf, count |
| `write` | 1 | fd, buf, count |
| `open` | 2 | filename, flags, mode |
| `close` | 3 | fd |
| `mprotect` | 10 | addr, len, prot |
| `execve` | 59 | filename, argv, envp |
| `exit` | 60 | status |

번호와 ABI는 아키텍처·운영체제별로 다르며 전체 표를 외우기보다 시스템 헤더와 매뉴얼을 확인한다. 현대 코드에서는 `openat`을 사용하는 경우도 많다.

### 오류 처리

Linux 시스템 콜은 실패 시 `rax`에 음수 오류 코드를 반환한다. 어셈블리 코드에서는 `test rax, rax; js error`처럼 부호를 검사할 수 있다.

```asm
syscall
test rax, rax
js   syscall_failed
```

---

## 12. 함수 호출과 시스템 콜 비교

| 구분 | 일반 함수 호출 | 시스템 콜 |
|---|---|---|
| 목적 | 같은 프로세스의 코드 호출 | 커널 서비스 요청 |
| 전환 | 보통 같은 사용자 모드 | 사용자→커널 모드 전환 |
| 명령 | `call` / `ret` | `syscall` / 커널 복귀 |
| x64 인자 4 | `rcx`(System V 함수) | `r10`(Linux syscall) |
| 반환 | 보통 `rax` | `rax`, 실패 시 음수 errno |

## 핵심 체크

- `call`은 반환 주소를 스택에 넣고 분기한다.
- `ret`은 스택에서 반환 주소를 꺼내 `rip`로 복원한다.
- `leave`는 현재 스택 프레임을 정리하는 축약 명령이다.
- 함수 호출 규약과 시스템 콜 규약은 서로 다르다.
- x86-64 Linux 시스템 콜의 번호는 `rax`, 인자는 `rdi,rsi,rdx,r10,r8,r9`다.
- 스택 정렬과 보존 레지스터 규칙을 지켜야 다른 코드와 안전하게 연동된다.

## 출처 및 이용 안내

이 문서는 Dreamhack 강의를 학습 목적으로 요약·재구성한 개인 학습 노트다. 원문은 [x86 Assembly: Essential Part (2)](https://learn.dreamhack.io/567)에서 확인한다.

