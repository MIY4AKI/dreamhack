# Dreamhack Assembly Training — 구구단 구현하기

## 1. 프로그램 목표

이용자로부터 `1`~`99` 범위의 10진수 문자열을 입력받아 정수로 변환하고, 해당 수에 1부터 9까지를 곱한 결과를 출력한다.

이 실습에서 연습하는 요소:

- `read`, `write`, `exit` 시스템 콜
- `.data`와 `.bss`
- 문자열의 개행 제거
- ASCII 숫자 검증과 문자열→정수 변환
- 비교·분기·반복문
- `imul`
- 외부 libc 함수 `printf` 호출과 링크

---

## 2. 전체 처리 흐름

```text
안내 문구 출력
   ↓
최대 3바이트 입력("99\n")
   ↓
개행을 NUL로 치환
   ↓
각 문자가 '0'~'9'인지 검증
   ↓
Horner 방식으로 정수 변환
   ↓
1 <= n <= 99 검증
   ↓
i = 1..9 반복: n * i 출력
   ↓
exit(0)
```

입력을 숫자 타입으로 직접 받는 것은 아니다. `read`는 바이트를 읽으므로 문자열 파싱을 직접 구현해야 한다.

---

## 3. 데이터 정의

```asm
section .data
    prompt_msg db "Enter a number (1-99): "
    prompt_len equ $ - prompt_msg

    invalid_msg db "Invalid number", 10
    invalid_len equ $ - invalid_msg

    fmt_mul db "%d * %d = %d", 10, 0

    STDIN     equ 0
    STDOUT    equ 1
    SYS_READ  equ 0
    SYS_WRITE equ 1
    SYS_EXIT  equ 60
    MAX_LEN   equ 3
    EOL_CHAR  equ 10

section .bss
    input_str resb MAX_LEN
    input_num resb 1
```

- `db`: 초기화된 바이트 정의
- `equ`: 상수 정의
- `resb`: 초기화되지 않은 바이트 공간 예약

입력값 최대가 99이므로 저장된 정수에는 1바이트가 충분하지만, 계산 중에는 범위와 호출 규약을 맞추기 위해 더 큰 레지스터로 확장한다.

---

## 4. 개행 제거: `strip_eol`

터미널에서 Enter를 누르면 `read` 결과에 `0x0a`가 포함될 수 있다. 파서가 문자열 끝을 알 수 있도록 이를 `0x00`으로 바꾼다.

```asm
; rdi = buffer address
strip_eol:
    xor ecx, ecx
.loop:
    cmp ecx, MAX_LEN
    jae .done
    cmp byte [rdi+rcx], EOL_CHAR
    je  .replace
    inc ecx
    jmp .loop
.replace:
    mov byte [rdi+rcx], 0
.done:
    ret
```

종료 조건에 버퍼 크기가 반드시 들어가야 한다. 개행이 없을 때 무한히 다음 메모리를 읽으면 Out-of-Bounds가 된다.

더 안전한 구현은 `read`의 반환 바이트 수를 함수 인자로 전달하고 그 범위만 검사한다.

---

## 5. 문자열을 정수로 변환

문자 `'0'`~`'9'`는 연속된 ASCII 값이므로 `digit = ch - '0'`으로 숫자를 얻을 수 있다.

여러 자리 수는 Horner 방식으로 누적한다.

```text
"9876"
(((0 * 10 + 9) * 10 + 8) * 10 + 7) * 10 + 6
```

```asm
; rdi = NUL-terminated string
; rsi = base (10)
; return rax = parsed value
parse_number:
    xor eax, eax
    mov r8, rdi
.loop:
    mov dl, [r8]
    test dl, dl
    jz .done

    cmp dl, '0'
    jb  .error
    cmp dl, '9'
    ja  .error

    sub dl, '0'
    movzx ecx, dl
    xor edx, edx
    mul rsi             ; rdx:rax = rax * 10
    add rax, rcx
    inc r8
    jmp .loop
.done:
    ret
.error:
    ; 에러 출력 후 종료 루틴으로 분기
```

검증 순서가 중요하다. `'0'`을 빼기 전에 문자가 유효 범위인지 확인한다.

이 실습의 입력은 두 자리로 제한되므로 오버플로가 없지만, 범용 파서라면 `rax * base + digit` 전 최대값 검사를 추가해야 한다.

---

## 6. 입력 범위 검증

```asm
cmp rax, 1
jb  invalid_input      ; unsigned: 1보다 작음
cmp rax, 99
ja  invalid_input      ; unsigned: 99보다 큼
mov [input_num], al
```

파서가 음수 문자를 허용하지 않으므로 unsigned 비교가 자연스럽다. 강의 코드처럼 `0` 또는 `100`과 비교해도 동일 범위를 표현할 수 있다.

---

## 7. 곱셈 반복문

루프 카운터를 1에서 시작해 9까지 출력한다.

```asm
mov r14d, 1

mul_loop:
    cmp r14d, 10
    je  exit_program

    movzx eax, byte [input_num]
    imul  eax, r14d
    ; eax = input_num * r14

    ; printf("%d * %d = %d\n", input_num, r14, result)
    lea  rdi, [rel fmt_mul]
    movzx esi, byte [input_num]
    mov  edx, r14d
    mov  ecx, eax
    xor  eax, eax       ; variadic 호출에서 vector argument 수 = 0
    call printf

    inc r14d
    jmp mul_loop
```

### `printf` 호출 시 주의

System V AMD64에서 인자는 `rdi,rsi,rdx,rcx` 순으로 전달된다. `printf`는 가변 인자 함수이므로 `al`에 사용한 벡터 인자 수를 전달하며 정수 인자만 있다면 보통 `eax=0`으로 만든다.

또한 외부 함수 호출 전 스택 정렬을 지켜야 한다. `_start`에서 직접 libc 함수를 부르는 구조는 C 런타임이 제공하는 `main` 환경과 다르므로 링크 방식과 초기 스택 상태를 의식한다.

---

## 8. `_start`의 역할

```asm
section .text
global _start
extern printf

_start:
    ; write(1, prompt_msg, prompt_len)
    mov eax, SYS_WRITE
    mov edi, STDOUT
    lea rsi, [rel prompt_msg]
    mov edx, prompt_len
    syscall

    ; read(0, input_str, MAX_LEN)
    xor eax, eax
    xor edi, edi
    lea rsi, [rel input_str]
    mov edx, MAX_LEN
    syscall
    test rax, rax
    jle exit_program

    lea rdi, [rel input_str]
    call strip_eol

    lea rdi, [rel input_str]
    mov esi, 10
    call parse_number

    ; 범위 검사 후 mul_loop로 진행
```

`read`의 반환값은 실제로 읽은 바이트 수이므로 0(EOF)과 음수(오류)를 검사한다.

---

## 9. 종료와 오류 경로

```asm
invalid_input:
    mov eax, SYS_WRITE
    mov edi, STDOUT
    lea rsi, [rel invalid_msg]
    mov edx, invalid_len
    syscall

exit_program:
    mov eax, SYS_EXIT
    xor edi, edi
    syscall
```

실전 프로그램에서는 성공은 0, 입력 오류는 0이 아닌 종료 코드를 반환하는 편이 좋다.

---

## 10. 빌드

`printf`를 사용하므로 libc와 동적 로더를 링크한다.

```bash
nasm -f elf64 gugudan.asm -o gugudan.o
ld -dynamic-linker /lib64/ld-linux-x86-64.so.2 \
   -o gugudan gugudan.o -lc
./gugudan
```

배포판과 아키텍처에 따라 동적 로더 경로가 다를 수 있다. `readelf -l /bin/ls | grep interpreter`로 현재 환경의 interpreter 경로를 확인할 수 있다.

대안으로 `main`을 정의하고 GCC를 링크 드라이버로 사용하면 C 런타임 초기화를 더 자연스럽게 처리할 수 있다.

---

## 11. GDB 검증 포인트

```gdb
break strip_eol
break parse_number
break mul_loop
run
```

확인할 내용:

- `read` 후 `input_str`에 실제로 어떤 바이트가 있는가?
- 개행이 NUL로 바뀌었는가?
- 각 반복에서 `rax = old_rax * 10 + digit`인가?
- `r14`가 1부터 9까지 증가하는가?
- `printf` 호출 직전 인자 레지스터가 올바른가?

```gdb
x/3bx &input_str
info registers rax rdi rsi rdx rcx r14
```

## 핵심 체크

- 시스템 콜 입력은 문자열이므로 파싱이 필요하다.
- 숫자 검증 후 `'0'`을 빼서 digit으로 변환한다.
- Horner 방식은 왼쪽부터 `acc = acc*base + digit`을 반복한다.
- 반복문은 카운터·비교·조건 분기·증가로 구성한다.
- 외부 `printf` 호출은 System V 인자 순서, `eax=0`, 스택 정렬을 확인한다.
- 버퍼 길이는 반드시 실제 읽은 바이트 수와 함께 관리한다.

## 출처 및 이용 안내

이 문서는 Dreamhack 강의를 학습 목적으로 요약·재구성한 개인 학습 노트다. 전체 실습 코드와 실행 화면은 [구구단 구현하기](https://learn.dreamhack.io/792)에서 확인한다.

