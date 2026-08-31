# Dreamhack Assembly Training — 텍스트 편집기 구현하기

## 1. 프로그램 목표와 범위

명령줄 인자로 파일 이름을 받아 다음 기능을 제공하는 단순한 추가형 텍스트 편집기를 x86-64 NASM으로 구현한다.

- 파일이 있으면 열고, 없으면 생성한다.
- 기존 내용을 읽어 화면에 표시한다.
- 터미널을 문자 단위 입력 모드로 바꾼다.
- 입력 문자를 버퍼 뒤에 추가하고 즉시 화면에 표시한다.
- `ESC` 다음 `w`로 저장하고 계속, `ESC` 다음 `q`로 저장 후 종료한다.

이 실습에는 커서 이동, 삭제, 삽입 위치 변경, 검색, 실행 취소 같은 완전한 편집기 기능은 포함되지 않는다.

---

## 2. 전체 구조

```text
argc/filename 확인
      ↓
open(O_RDWR | O_CREAT)
      ↓
read file → buffer, buf_len
      ↓
현재 termios 백업
      ↓
non-canonical + no-echo 적용
      ↓
alternate screen 진입 및 화면 초기화
      ↓
기존 buffer 화면 출력
      ↓
한 글자씩 read하는 edit loop
      ├─ 일반 문자: buffer append + 화면 출력
      └─ ESC + w/q: 저장 또는 저장 후 종료
      ↓
termios/화면 복구 → close → exit
```

---

## 3. 사용 시스템 콜

| 시스템 콜 | x86-64 번호 | 용도 |
|---|---:|---|
| `read` | 0 | 파일/표준 입력에서 읽기 |
| `write` | 1 | 화면 또는 파일에 쓰기 |
| `open` | 2 | 파일 열기/생성 |
| `close` | 3 | 파일 디스크립터 닫기 |
| `lseek` | 8 | 파일 오프셋 이동 |
| `ioctl` | 16 | 터미널 속성 읽기·설정 |
| `exit` | 60 | 프로세스 종료 |

Linux 시스템 콜은 실패 시 `rax`가 음수다.

```asm
syscall
test rax, rax
js   error_path
```

---

## 4. 정규 모드와 비정규 모드

### Canonical mode

기본 터미널 모드다. 입력을 줄 단위로 버퍼링하며 Enter를 눌러야 프로그램의 `read`가 한 줄을 받는 형태가 일반적이다.

### Non-canonical mode

문자 단위 입력을 처리할 수 있다. 간단한 에디터에서는 한 키마다 즉시 로직을 실행하기 위해 이 모드가 필요하다.

입력 자체를 터미널이 자동 표시하지 않도록 `ECHO`도 끈다. 프로그램이 필요한 문자만 직접 `write`하여 화면에 표시한다.

---

## 5. termios와 ioctl

`termios` 구조체의 주요 필드:

- `c_iflag`: 입력 전처리 방식
- `c_oflag`: 출력 후처리 방식
- `c_cflag`: 장치·전송 설정
- `c_lflag`: canonical, echo 등 로컬 터미널 동작
- `c_cc[]`: VMIN, VTIME 등의 제어 문자 설정

### VMIN과 VTIME

- `VMIN=1`: 최소 1바이트가 입력될 때 `read` 반환
- `VTIME=0`: 별도 타임아웃 없음

### 현재 설정 백업

```asm
saved_termios:
    mov eax, SYS_IOCTL
    mov edi, STDIN
    mov esi, TCGETS
    lea rdx, [rel termios_old]
    syscall
    test rax, rax
    js fatal_exit
    ret
```

### 비정규·no-echo 설정

```asm
set_non_canonical_mode:
    mov eax, SYS_IOCTL
    mov edi, STDIN
    mov esi, TCGETS
    lea rdx, [rel termios_new]
    syscall
    test rax, rax
    js fatal_exit

    and dword [termios_new + 12], ~(ICANON_FLAG | ECHO_FLAG)
    mov byte  [termios_new + 22], 0    ; VTIME
    mov byte  [termios_new + 23], 1    ; VMIN

    mov eax, SYS_IOCTL
    mov edi, STDIN
    mov esi, TCSETS
    lea rdx, [rel termios_new]
    syscall
    ret
```

오프셋 `12`, `22`, `23`은 특정 Linux x86-64 `termios` 레이아웃을 가정한다. 아키텍처와 C 라이브러리 헤더를 확인해야 하며, 범용 코드는 C 래퍼나 생성된 상수를 사용해 하드코딩을 줄이는 편이 안전하다.

---

## 6. ANSI 이스케이프와 대체 화면

```asm
ENTER_ALT_SCREEN db 0x1b, "[?1049h"
EXIT_ALT_SCREEN  db 0x1b, "[?1049l"
CLEAR_SCREEN     db 0x1b, "[2J", 0x1b, "[H"
```

- `ESC[?1049h`: 대체 화면 진입
- `ESC[?1049l`: 원래 화면 복귀
- `ESC[2J`: 화면 지우기
- `ESC[H`: 커서를 홈 위치로 이동

대체 화면을 사용하면 기존 셸 화면을 덮어쓰지 않고 전체 화면 애플리케이션처럼 보이게 할 수 있다.

---

## 7. 데이터와 상태

```asm
section .bss
    buffer      resb 8192
    buf_len     resq 1
    termios_old resb 66
    termios_new resb 66
    key_buf     resb 1
```

- `buffer`: 파일 내용과 새 입력을 저장
- `buf_len`: 현재 유효 데이터 길이
- `termios_old`: 종료 시 복구할 원래 설정
- `termios_new`: 변경하여 적용할 설정
- `key_buf`: 한 번에 읽은 키 1바이트

고정 8192바이트 버퍼이므로 입력을 추가하기 전에 `buf_len < 8192`를 확인해야 한다. 원본 강의의 핵심 흐름을 확장해 실제로 구현할 때 반드시 추가할 경계 검사다.

---

## 8. 명령줄 인자와 파일 열기

리눅스 x86-64에서 `_start` 진입 시 초기 스택에는 대체로 다음 순서로 값이 있다.

```text
[rsp]      argc
[rsp+8]    argv[0]
[rsp+16]   argv[1]
...
```

```asm
_start:
    mov rax, [rsp]
    cmp rax, 2
    jne no_argument

    ; open(argv[1], O_RDWR|O_CREAT, 0644)
    mov eax, SYS_OPEN
    mov rdi, [rsp+16]
    mov esi, O_RDWR | O_CREAT
    mov edx, 0o644
    syscall
    test rax, rax
    js open_fail
    mov r12, rax           ; fd 보관
```

`r12`는 System V의 callee-saved 레지스터이므로 여러 내부 함수 호출 사이에서 파일 디스크립터를 유지하기 편리하다. 직접 작성한 각 함수도 callee-saved 규칙을 지켜야 한다.

### 기존 파일 읽기

```asm
mov eax, SYS_READ
mov rdi, r12
lea rsi, [rel buffer]
mov edx, 8192
syscall
test rax, rax
js open_fail
mov [buf_len], rax
```

8192바이트보다 큰 파일은 처음 일부만 읽는다. 완전한 편집기는 파일 크기 확인, 동적 할당, 반복 `read`가 필요하다.

---

## 9. 편집 루프

```asm
.edit_loop:
    xor eax, eax
    xor edi, edi
    lea rsi, [rel key_buf]
    mov edx, 1
    syscall
    cmp rax, 1
    jne fatal_after_terminal_change

    mov al, [key_buf]
    cmp al, 27                ; ESC
    je .handle_esc

.normal_key:
    mov rbx, [buf_len]
    cmp rbx, 8192
    jae .edit_loop            ; 또는 사용자에게 buffer full 표시
    mov [buffer+rbx], al
    inc rbx
    mov [buf_len], rbx

    mov eax, SYS_WRITE
    mov edi, STDOUT
    lea rsi, [rel key_buf]
    mov edx, 1
    syscall
    jmp .edit_loop
```

비정규/no-echo 모드이므로 화면 에코도 프로그램이 직접 한다.

### ESC 명령 처리

```asm
.handle_esc:
    xor eax, eax
    xor edi, edi
    lea rsi, [rel key_buf]
    mov edx, 1
    syscall
    cmp rax, 1
    jne exit_program

    mov al, [key_buf]
    cmp al, 'w'
    je .save_and_continue
    cmp al, 'q'
    je .save_and_exit
    jmp .normal_key

.save_and_continue:
    call save_file
    jmp .edit_loop

.save_and_exit:
    call save_file
    jmp exit_program
```

`ESC` 자체를 일반 문자로 저장할지, 알 수 없는 명령을 어떻게 처리할지는 프로그램 정책으로 명확히 정한다.

---

## 10. 파일 저장

기존 파일을 읽은 뒤 파일 오프셋은 읽은 길이만큼 이동해 있다. 그대로 쓰면 전체 버퍼를 뒤에 다시 추가해 내용이 중복될 수 있으므로 먼저 시작 위치로 옮긴다.

```asm
save_file:
    ; lseek(fd, 0, SEEK_SET)
    mov eax, SYS_LSEEK
    mov rdi, r12
    xor esi, esi
    xor edx, edx
    syscall
    test rax, rax
    js save_error

    ; write(fd, buffer, buf_len)
    mov eax, SYS_WRITE
    mov rdi, r12
    lea rsi, [rel buffer]
    mov rdx, [buf_len]
    syscall
    ; 실제로 전체 길이를 썼는지 확인 필요
    ret
```

### 실전 보완 1 — partial write

`write`는 요청한 전체 길이보다 적게 쓸 수 있다. 반환값만큼 포인터와 남은 길이를 조정해 반복해야 한다.

### 실전 보완 2 — 파일 축소

새 내용이 원본보다 짧아질 수 있는 편집기라면 `lseek(0)` 후 쓰기만 해서는 원본의 뒤쪽 데이터가 남는다. `ftruncate(fd, new_len)` 등을 사용해야 한다. 이 강의 구현은 뒤에 문자를 추가하는 형태라 일반 흐름에서는 길이가 줄지 않지만, 완전한 편집기로 확장할 때 중요하다.

---

## 11. 종료와 터미널 복구

```asm
restore_terminal:
    mov eax, SYS_IOCTL
    mov edi, STDIN
    mov esi, TCSETS
    lea rdx, [rel termios_old]
    syscall

    mov eax, SYS_WRITE
    mov edi, STDOUT
    lea rsi, [rel EXIT_ALT_SCREEN]
    mov edx, EXIT_ALT_SCREEN_LEN
    syscall
    ret

exit_program:
    call restore_terminal

    mov eax, SYS_CLOSE
    mov rdi, r12
    syscall

    mov eax, SYS_EXIT
    xor edi, edi
    syscall
```

가장 중요한 안전성 원칙은 **터미널 모드를 바꾼 뒤 발생하는 모든 오류 경로가 복구 루틴을 거치게 하는 것**이다. 복구 없이 종료하면 셸이 no-echo/non-canonical 상태로 남아 사용자가 입력을 볼 수 없게 된다.

SIGINT, SIGTERM, 예외 종료까지 안정적으로 복구하려면 신호 처리와 단일 cleanup 경로가 필요하다. 강제 종료 후 터미널이 망가졌다면 셸에서 `reset` 또는 `stty sane`으로 복구할 수 있다.

---

## 12. 빌드와 실행

```bash
nasm -f elf64 editor.asm -o editor.o
ld editor.o -o editor
./editor dreamhack.txt
```

동작 예:

- 인자 없이 실행: 사용법 출력 후 종료
- 파일 이름 전달: 파일 열기/생성, 기존 내용 표시
- 일반 키: 버퍼와 화면에 추가
- `ESC`, `w`: 저장 후 계속
- `ESC`, `q`: 저장·복구·종료

---

## 13. 테스트 체크리스트

### 정상 흐름

- 존재하지 않는 파일을 새로 만든다.
- 기존 파일을 열어 내용이 표시된다.
- 여러 문자를 입력한 뒤 저장한다.
- 종료 후 셸 화면과 입력 모드가 정상이다.

### 경계와 오류

- 인자 없음/인자 과다
- 읽기 전용 경로, 권한 없는 파일
- 8192바이트 파일과 그보다 큰 파일
- 버퍼가 가득 찬 뒤 추가 입력
- EOF 또는 `read` 실패
- 저장 중 partial write
- `ioctl` 실패 또는 터미널이 아닌 stdin
- 예상하지 못한 ESC 조합
- 비정상 종료 후 터미널 복구

### GDB 관찰

```gdb
break _start
break set_non_canonical_mode
break .edit_loop
break save_file
break restore_terminal
```

확인할 값:

- `r12`: 파일 디스크립터
- `[buf_len]`: 현재 길이
- `key_buf`: 방금 입력한 키
- termios의 `c_lflag`, VMIN, VTIME
- `lseek`과 `write`의 반환값

## 핵심 체크

- 문자 단위 입력을 위해 non-canonical 모드를 사용한다.
- `ECHO`를 끄면 프로그램이 화면 출력을 직접 해야 한다.
- `ioctl(TCGETS/TCSETS)`로 termios를 백업·변경·복구한다.
- 기존 파일의 오프셋을 0으로 되돌린 뒤 저장한다.
- 모든 오류 경로에서 터미널과 화면을 복구한다.
- 고정 버퍼 경계, partial read/write, 큰 파일을 고려한다.
- 실제 편집기로 확장할 때 파일 축소, 신호 처리, 동적 버퍼가 필요하다.

## 출처 및 이용 안내

이 문서는 Dreamhack 강의를 학습 목적으로 요약·재구성한 개인 학습 노트다. 전체 코드와 화면 예시는 [텍스트 편집기 구현하기](https://learn.dreamhack.io/794)에서 확인한다.

