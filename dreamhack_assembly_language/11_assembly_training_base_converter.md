# Dreamhack Assembly Training — 진법 변환기 구현하기

## 1. 프로그램 목표

다음 세 입력을 받아 진법 표기를 변환한다.

1. 원본 문자열의 진법: 2, 8, 10, 16 중 하나
2. 해당 진법으로 작성된 숫자 문자열
3. 출력할 목표 진법: 2, 8, 10, 16 중 하나

```text
source base + digit string
          │ parse_number
          ▼
     64-bit integer
          │ convert_value_to_base
          ▼
target-base digit string
```

핵심은 모든 표기를 먼저 하나의 정수 값으로 통일한 뒤 목표 진법 문자열로 다시 표현하는 것이다.

---

## 2. 변환 원리

### 문자열 → 정수: Horner 방식

16진수 문자열 `1234`:

```text
(((1 × 16) + 2) × 16 + 3) × 16 + 4 = 4660
```

일반식:

```text
value = 0
for digit in input:
    value = value * source_base + digit
```

### 정수 → 문자열: 나눗셈과 나머지

10진수 4660을 16진수로 바꾸기:

```text
4660 / 16 = 291 ... 4
 291 / 16 =  18 ... 3
  18 / 16 =   1 ... 2
   1 / 16 =   0 ... 1
```

나머지는 `4,3,2,1`처럼 낮은 자리부터 나오므로 역순으로 조립해야 `1234`가 된다. 구현에서는 출력 버퍼 끝에서 앞으로 문자를 채우는 방법이 편리하다.

---

## 3. 데이터와 버퍼

```asm
section .data
    prompt_src_base db "Enter source base (2, 8, 10, 16): "
    prompt_src_len  equ $ - prompt_src_base

    prompt_value db "Enter number in that base: "
    prompt_value_len equ $ - prompt_value

    prompt_tgt_base db "Enter target base (2, 8, 10, 16): "
    prompt_tgt_len  equ $ - prompt_tgt_base

    invalid_base_msg db "Error: Unsupported base.", 10
    invalid_base_len equ $ - invalid_base_msg

    invalid_digit_msg db "Error: Invalid digit.", 10
    invalid_digit_len equ $ - invalid_digit_msg

    MAX_BASE_LEN  equ 8
    MAX_VALUE_LEN equ 65

section .bss
    base_input_buffer    resb MAX_BASE_LEN
    base_output_buffer   resb MAX_BASE_LEN
    number_input_buffer  resb MAX_VALUE_LEN
    number_output_buffer resb MAX_VALUE_LEN
    source_base          resd 1
    target_base          resd 1
    input_base_value     resq 1
```

`resb`, `resw`, `resd`, `resq`는 각각 1, 2, 4, 8바이트 단위로 공간을 예약한다.

65바이트 출력 버퍼는 64비트 값을 2진수로 표현한 최대 64자리와 문자열 종단/여유 공간을 고려한 크기다.

---

## 4. 입력 진법 검증: `validate_base`

진법 입력 자체는 10진수 문자열로 받는다. 먼저 `parse_number(buffer, 10)`으로 정수화한 뒤 2, 8, 10, 16인지 비교한다.

```asm
; rdi = base string
; esi = 10
; rdx = destination address
validate_base:
    mov r10, rdx
    call parse_number

    cmp eax, 2
    je  .valid
    cmp eax, 8
    je  .valid
    cmp eax, 10
    je  .valid
    cmp eax, 16
    je  .valid
    jmp invalid_base

.valid:
    mov [r10], eax
    ret
```

목적지에 쓰는 시점은 검증 성공 후로 두는 것이 좋다. 유효하지 않은 값을 먼저 저장하면 오류 경로가 해당 변수를 잘못 사용할 수 있다.

`r10`은 System V에서 caller-saved이므로 `parse_number`가 덮어쓸 수 있다. 직접 작성한 함수 사이에서 사용할 경우 “parse_number가 r10을 보존한다”는 자체 약속이 필요하며, 더 일반적인 구현은 스택이나 callee-saved 레지스터에 목적지 포인터를 보관한다.

---

## 5. 확장된 `parse_number`

2~16진수는 `0`~`9`, `A`~`F`, `a`~`f`를 지원해야 한다.

```text
'0'..'9' → ch - '0'          (0..9)
'A'..'F' → ch - 'A' + 10     (10..15)
'a'..'f' → ch - 'a' + 10     (10..15)
```

문자 자체가 16진수 범위에 있더라도 실제 source base보다 작은 digit인지 추가 확인한다. 예를 들어 `'8'`은 10진수에서는 유효하지만 8진수에서는 유효하지 않다.

```asm
; rdi = string, rsi = source base, return rax
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
    jbe .number
    cmp dl, 'A'
    jb  .error
    cmp dl, 'F'
    jbe .upper
    cmp dl, 'a'
    jb  .error
    cmp dl, 'f'
    ja  .error
    sub dl, 'a' - 10
    jmp .got_digit

.upper:
    sub dl, 'A' - 10
    jmp .got_digit
.number:
    sub dl, '0'

.got_digit:
    movzx ecx, dl
    cmp rcx, rsi
    jae .error

    xor edx, edx
    mul rsi
    add rax, rcx
    inc r8
    jmp .loop
.done:
    ret
.error:
    jmp invalid_digit
```

### 오버플로 주의

64비트 누적값이 `UINT64_MAX`를 넘으면 상위 결과가 `rdx`에 생기고 `rax`가 잘린다. 강의는 이 범위를 단순화하지만 안전한 파서는 곱셈 전 다음을 검사한다.

```text
value > (UINT64_MAX - digit) / base  → overflow
```

---

## 6. 정수 → 목표 진법 문자열

함수 인자:

- `rdi`: 변환할 정수
- `rsi`: 목표 진법
- `rdx`: 출력 버퍼

버퍼 끝에 NUL을 놓고 뒤에서 앞으로 한 자리씩 기록한다.

```asm
; return rax = first digit address
; return rcx = output length
convert_value_to_base:
    mov rax, rdi
    lea r8, [rdx + MAX_VALUE_LEN - 1]
    mov byte [r8], 0
    xor ecx, ecx

    test rax, rax
    jnz .loop
    dec r8
    mov byte [r8], '0'
    mov ecx, 1
    jmp .done

.loop:
    xor edx, edx
    div rsi                 ; rax=quotient, rdx=remainder

    cmp edx, 9
    jbe .decimal_digit
    add dl, 'A' - 10
    jmp .store
.decimal_digit:
    add dl, '0'
.store:
    dec r8
    mov [r8], dl
    inc ecx
    test rax, rax
    jnz .loop

.done:
    mov rax, r8
    ret
```

### 중요한 구현 포인트

버퍼 뒤쪽부터 채웠으므로 출력은 `number_output_buffer` 시작이 아니라 **첫 번째로 생성된 숫자 위치**에서 시작해야 한다. 위 예시는 그 주소를 `rax`, 길이를 `rcx`로 반환한다.

```asm
call convert_value_to_base
mov rsi, rax          ; 실제 첫 글자
mov rdx, rcx          ; 실제 길이
mov eax, 1
mov edi, 1
syscall
```

고정된 버퍼 시작부터 65바이트 전체를 쓰면 앞부분의 NUL 바이트와 사용하지 않은 데이터까지 출력하려 하므로 원하는 결과가 나오지 않는다.

---

## 7. `_start`의 전체 흐름

```text
1. source base 프롬프트 출력
2. base_input_buffer에 read
3. read 반환 길이 안에서 개행 제거
4. validate_base → source_base
5. 숫자 문자열 프롬프트 출력
6. number_input_buffer에 read
7. 개행 제거
8. parse_number(input, source_base) → input_base_value
9. target base 프롬프트 출력
10. base_output_buffer에 read
11. 개행 제거
12. validate_base → target_base
13. convert_value_to_base(value, target_base, output buffer)
14. 실제 시작 주소와 길이만 write
15. 개행 출력 후 exit(0)
```

반복되는 프롬프트 출력과 입력 읽기를 `read_line` 같은 함수로 추출하면 코드 중복을 줄일 수 있다.

---

## 8. 입력 종료 처리

`read`의 반환값은 실제 길이다.

```asm
syscall
test rax, rax
jle exit_program

; rax 바이트 범위 안에서 '\n'을 찾거나,
; 버퍼에 여유가 있을 때 [buffer+rax]에 NUL 저장
```

주의:

- 버퍼를 정확히 가득 읽은 경우 `[buffer+rax]`는 버퍼 밖일 수 있다.
- NUL 종단을 직접 추가하려면 요청 크기를 `capacity-1`로 제한하거나 별도 여유 바이트를 둔다.
- 개행 없는 긴 입력은 나머지가 다음 `read`에 남을 수 있으므로 실전에서는 초과 입력을 비우는 로직이 필요하다.

---

## 9. 빌드와 테스트

외부 libc 함수를 사용하지 않으므로 단순 링크가 가능하다.

```bash
nasm -f elf64 base_translator.asm -o base_translator.o
ld base_translator.o -o base_translator
./base_translator
```

테스트 케이스:

| 입력 진법 | 입력 | 출력 진법 | 예상 |
|---:|---|---:|---|
| 10 | `4660` | 16 | `1234` |
| 16 | `deadBEEF` | 10 | 해당 unsigned 10진수 |
| 2 | `101010` | 8 | `52` |
| 8 | `19` | 10 | 오류(`9`는 8진수 digit 아님) |
| 3 | 임의 값 | 10 | 지원하지 않는 진법 오류 |
| 10 | `0` | 2 | `0` |

경계 테스트:

- 빈 문자열
- 최대 64비트 값
- 최대값을 1 초과한 입력
- 대문자·소문자 16진수
- 버퍼 최대 길이 입력

---

## 10. GDB 검증 포인트

```gdb
break parse_number
break validate_base
break convert_value_to_base
run
```

관찰:

- 각 digit 변환 후 `ecx` 값이 올바른가?
- `cmp ecx, esi`가 현재 진법보다 큰 digit을 거부하는가?
- `mul` 후 `rdx`가 0인가? 오버플로가 생기지 않았는가?
- `div`마다 `rax`가 몫, `rdx`가 나머지인가?
- 출력 포인터가 첫 번째 digit을 가리키는가?

## 핵심 체크

- 문자열→정수는 `value = value*base + digit`이다.
- 정수→문자열은 반복 나눗셈의 나머지를 역순으로 조립한다.
- 문자 범위뿐 아니라 `digit < base`인지 검증한다.
- 값 0은 변환 루프에 들어가지 않으므로 별도 처리한다.
- 버퍼 뒤에서 만든 문자열은 실제 첫 글자 주소와 길이를 함께 반환한다.
- 범용 구현에는 64비트 오버플로와 입력 버퍼 경계를 검사해야 한다.

## 출처 및 이용 안내

이 문서는 Dreamhack 강의를 학습 목적으로 요약·재구성한 개인 학습 노트다. 전체 코드와 실행 예시는 [진법 변환기 구현하기](https://learn.dreamhack.io/793)에서 확인한다.

