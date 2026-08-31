# Dreamhack Assembly Language — x86 Assembly Essential Part (1)

## 1. 어셈블리어의 위치

CPU는 기계어 바이트를 직접 해석한다. 어셈블리어는 그 명령을 사람이 읽을 수 있는 니모닉과 피연산자로 표현하며, 어셈블러가 이를 기계어로 바꾼다. 반대로 디스어셈블러는 기계어를 어셈블리어 표현으로 되돌려 바이너리 분석을 돕는다.

ISA마다 명령어 인코딩과 레지스터가 다르므로 x86-64 어셈블리, ARM64 어셈블리 등도 서로 다른 언어에 가깝다. 이 강의는 x86-64의 **Intel 문법**을 사용한다.

---

## 2. Intel 문법의 기본 구조

```text
opcode destination, source
```

명령은 동사에 해당하는 opcode와 0개 이상의 피연산자로 구성된다.

```asm
mov eax, 3              ; 2개
push rbp                ; 1개
ret                     ; 0개
imul rax, rdx, 3        ; 3개
```

Intel 문법은 일반적으로 목적지가 먼저 온다. AT&T 문법은 보통 원본과 목적지 순서가 반대이고 레지스터 앞에 `%`, 즉시값 앞에 `$`를 붙이는 등 차이가 있다.

### 명령어 분류

| 분류 | 대표 명령 |
|---|---|
| 데이터 이동 | `mov`, `lea` |
| 산술 | `inc`, `dec`, `add`, `sub`, `mul`, `imul`, `div`, `idiv` |
| 논리 | `and`, `or`, `xor`, `not` |
| 비교 | `cmp`, `test` |
| 분기 | `jmp`, `je`, `jne`, `jg`, `jl` 등 |
| 스택 | `push`, `pop` |
| 함수 | `call`, `leave`, `ret` |
| 커널 요청 | `syscall` |

---

## 3. 피연산자

피연산자는 크게 세 종류다.

1. 즉시값(Immediate): `10`, `0xdeadbeef`
2. 레지스터(Register): `rax`, `ecx`, `al`
3. 메모리(Memory): `[rax]`, `[rbp-8]`, `[rbx+rcx*4]`

### 메모리 크기 지정자

| 지정자 | 크기 |
|---|---:|
| `BYTE` | 1바이트 |
| `WORD` | 2바이트 |
| `DWORD` | 4바이트 |
| `QWORD` | 8바이트 |

```asm
mov DWORD [rdi], eax
mov BYTE  [var], 0x10
mov rax, QWORD [rbp-8]
```

Dreamhack의 디스어셈블 예시에는 `DWORD PTR`처럼 표시되기도 한다. NASM 소스에서는 보통 `dword [rdi]`처럼 `PTR` 없이 작성한다. 도구 문법의 차이를 구분한다.

### 매뉴얼의 표기

| 표기 | 의미 |
|---|---|
| `r8`, `r16`, `r32`, `r64` | 해당 비트 크기의 범용 레지스터 |
| `imm8`~`imm64` | 해당 크기로 표현되는 즉시값 |
| `r/m32` | 32비트 레지스터 또는 32비트 메모리 피연산자 |

예를 들어 `MOV r/m32, r32`는 목적지에 32비트 레지스터 또는 메모리, 원본에 32비트 레지스터가 올 수 있다는 뜻이다.

### 유효 주소 형식

x86-64 메모리 주소는 다음 조합을 자주 사용한다.

```text
[base + index * scale + displacement]
```

- `scale`: 1, 2, 4, 8
- 배열 원소 접근: `[rbx + rcx*4]`
- 지역 변수: `[rbp - 0x10]`
- 구조체 필드: `[rdi + 8]`

---

## 4. 데이터 이동 — `mov`

```asm
mov destination, source
```

원본의 값을 목적지로 복사한다. 일반적인 정수 `mov`는 메모리에서 메모리로 직접 복사할 수 없으므로 중간 레지스터가 필요하다.

```asm
mov rdi, rax            ; 레지스터 → 레지스터
mov eax, 0x31337        ; 즉시값 → 레지스터
mov DWORD [var], eax    ; 레지스터 → 메모리
mov eax, DWORD [var]    ; 메모리 → 레지스터
```

주의할 점:

- 원본과 목적지 크기가 맞아야 한다.
- 작은 값을 큰 레지스터로 옮길 때 부호 확장/제로 확장이 필요하면 `movsx`, `movzx` 계열을 사용한다.
- `mov eax, value`는 `rax` 상위 32비트를 0으로 만든다.

---

## 5. 주소 계산 — `lea`

`lea`(Load Effective Address)는 대괄호 안의 주소식을 계산하지만 그 주소의 메모리를 읽지는 않는다.

```asm
lea rax, [rbx + rcx*4]
```

`rbx`가 `int` 배열 시작 주소이고 `rcx`가 인덱스라면 `rax = &arr[rcx]`와 같다.

```c
int value = arr[2];     // mov: 메모리의 값을 읽음
int *ptr = &arr[2];     // lea: 주소를 계산함
```

```asm
mov rax, [rbx + rcx*4]  ; 해당 주소의 값
lea rax, [rbx + rcx*4]  ; 해당 주소 자체
```

`lea`는 메모리 접근 없이 덧셈과 제한적인 곱셈을 한 번에 계산할 수 있어 일반 산술 최적화에도 등장한다.

---

## 6. 덧셈과 뺄셈

```asm
add destination, source     ; destination += source
sub destination, source     ; destination -= source
inc destination             ; +1
dec destination             ; -1
```

예시:

```asm
mov rax, 100
add rax, 50        ; 150
sub rax, 20        ; 130
inc rax            ; 131
dec rax            ; 130
```

`add`와 `sub`는 여러 상태 플래그를 갱신한다. `inc`와 `dec`는 대부분의 산술 플래그를 바꾸지만 `CF`는 바꾸지 않는다는 차이가 있어 다중 정밀도 연산에서 중요하다.

---

## 7. 곱셈 — `mul`과 `imul`

### `mul`: 부호 없는 곱셈

한 피연산자 형식은 누산기 레지스터를 암시적으로 사용하며 결과가 두 배 크기가 될 수 있다.

| 피연산자 크기 | 암시적 곱셈 | 결과 |
|---:|---|---|
| 8비트 | `AL × src8` | `AX` |
| 16비트 | `AX × src16` | `DX:AX` |
| 32비트 | `EAX × src32` | `EDX:EAX` |
| 64비트 | `RAX × src64` | `RDX:RAX` |

```asm
mov eax, 6
mov ecx, 7
mul ecx             ; edx:eax = 42
```

### `imul`: 부호 있는 곱셈

```asm
imul source
imul destination, source
imul destination, source, immediate
```

```asm
imul rax, rdx          ; rax *= rdx
imul rax, rdx, 10      ; rax = rdx * 10
```

2·3피연산자 형식은 목적지 크기로 결과가 잘릴 수 있다. 결과가 해당 크기에 맞지 않으면 `CF`와 `OF`가 설정된다.

---

## 8. 나눗셈 — `div`와 `idiv`

- `div`: 부호 없는 나눗셈
- `idiv`: 부호 있는 나눗셈

| 제수 크기 | 피제수 | 몫 | 나머지 |
|---:|---|---|---|
| 8비트 | `AX` | `AL` | `AH` |
| 16비트 | `DX:AX` | `AX` | `DX` |
| 32비트 | `EDX:EAX` | `EAX` | `EDX` |
| 64비트 | `RDX:RAX` | `RAX` | `RDX` |

부호 없는 64비트 예시:

```asm
mov rax, 100
xor rdx, rdx       ; 피제수 상위 64비트 = 0
mov rcx, 7
div rcx            ; rax=14, rdx=2
```

부호 있는 나눗셈은 피제수를 올바르게 부호 확장해야 한다.

```asm
mov rax, -100
cqo                ; rax의 부호를 rdx까지 확장
mov rcx, 7
idiv rcx
```

제수가 0이거나 몫이 목적 레지스터에 들어가지 않으면 divide error 예외가 발생한다.

---

## 9. 비트 논리 연산

```asm
and destination, source
or  destination, source
xor destination, source
not destination
```

활용:

- `and`: 마스크와 겹치는 비트만 남김
- `or`: 특정 비트를 1로 설정
- `xor`: 다른 비트만 1, 토글·간단한 변환
- `not`: 모든 비트 반전

```asm
and eax, 0xff       ; 하위 8비트만 보존
or  eax, 0x20       ; 특정 비트 설정
xor eax, eax        ; eax=0, rax 상위도 0
not eax             ; 32비트 반전
```

`xor reg, reg`는 레지스터를 0으로 만드는 흔한 관용구이며 플래그도 갱신한다.

---

## 10. 비교 — `cmp`와 `test`

### `cmp`

```asm
cmp destination, source
```

내부적으로 `destination - source`를 계산하여 플래그만 갱신하고 피연산자 값은 보존한다.

```asm
cmp rax, rbx
je  equal
```

### `test`

```asm
test destination, source
```

두 값을 AND하여 플래그만 갱신한다. 값이 0인지 확인하거나 특정 비트를 검사할 때 자주 사용한다.

```asm
test rax, rax       ; rax == 0 ?
jz   is_zero

test eax, 1         ; 최하위 비트가 0인가?
jz   is_even
```

---

## 11. 분기 명령

### 무조건 분기

```asm
jmp label
```

### 같음과 다름

| 명령 | 조건 |
|---|---|
| `je`, `jz` | `ZF == 1` |
| `jne`, `jnz` | `ZF == 0` |

### 부호 있는 대소 비교

| 명령 | 의미 | 플래그 조건 |
|---|---|---|
| `jg` | greater | `ZF=0 && SF=OF` |
| `jge` | greater or equal | `SF=OF` |
| `jl` | less | `SF!=OF` |
| `jle` | less or equal | `ZF=1 || SF!=OF` |

### 부호 없는 대소 비교

| 명령 | 의미 | 대표 조건 |
|---|---|---|
| `ja` | above | `CF=0 && ZF=0` |
| `jae` | above or equal | `CF=0` |
| `jb` | below | `CF=1` |
| `jbe` | below or equal | `CF=1 || ZF=1` |

같은 `cmp` 뒤에도 값의 의미가 signed라면 `jg/jl`, unsigned라면 `ja/jb` 계열을 사용한다.

---

## 12. 반복문 구현

### while 형태

```asm
xor rax, rax        ; sum = 0
xor rcx, rcx        ; cnt = 0

loop_start:
    cmp rcx, 101
    jge loop_end
    inc rax
    inc rcx
    jmp loop_start

loop_end:
```

### 고정 횟수 반복

```asm
mov rcx, 10
loop_start:
    ; 반복할 코드
    loop loop_start  ; rcx-- 후 0이 아니면 분기
```

`loop` 명령은 간결하지만 현대 x86-64에서 반드시 가장 빠른 방식은 아니다. 실제 컴파일러는 `dec`/`cmp`와 조건 분기를 조합하는 경우가 많다.

### do-while 형태

```asm
do_loop:
    ; 한 번은 반드시 실행
    inc rax
    cmp rax, 10
    jl do_loop
```

### 반복문 점검

- 카운터 초기화가 있는가?
- 종료 조건이 정확한가? (`<`와 `<=`, off-by-one)
- 매 반복마다 카운터가 변하는가?
- 배열 접근은 버퍼 범위를 넘지 않는가?
- 호출 함수가 카운터 레지스터를 덮어쓰지 않는가?

## 핵심 체크

- `mov`는 값, `lea`는 주소 계산이다.
- 메모리 피연산자의 크기를 정확히 지정한다.
- `mul/div`는 암시적으로 `rax`와 `rdx`를 사용한다.
- `cmp`는 뺄셈, `test`는 AND의 결과로 플래그만 갱신한다.
- signed와 unsigned 분기 명령을 구분한다.
- 반복문은 비교·조건 분기·무조건 분기의 조합이다.

## 출처 및 이용 안내

이 문서는 Dreamhack 강의를 학습 목적으로 요약·재구성한 개인 학습 노트다. 원문은 [x86 Assembly: Essential Part (1)](https://learn.dreamhack.io/37)에서 확인한다.

