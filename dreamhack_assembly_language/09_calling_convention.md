# Dreamhack Assembly Language — 함수 호출 규약(Calling Convention)

## 1. 호출 규약이 필요한 이유

함수를 호출할 때 caller와 callee는 다음 사항에 대해 같은 약속을 사용해야 한다.

- 인자를 어떤 순서로 어디에 전달하는가?
- 반환값은 어디에 두는가?
- 반환 주소와 스택 프레임은 어떻게 보관하는가?
- 어떤 레지스터를 caller/callee 중 누가 보존하는가?
- 인자에 사용한 스택 공간을 누가 정리하는가?
- 스택 정렬은 어떻게 유지하는가?

컴파일러는 소스 코드를 대상 ABI에 맞게 변환하지만, 어셈블리를 직접 작성하거나 디스어셈블된 코드를 분석할 때는 규약을 사람이 이해해야 한다.

용어:

- **caller**: 다른 함수를 호출하는 함수
- **callee**: 호출되는 함수
- **return address**: callee가 끝난 뒤 caller에서 이어서 실행할 주소
- **SFP(saved frame pointer)**: 호출자의 프레임 포인터를 보존한 값
- **ABI(Application Binary Interface)**: 호출 규약뿐 아니라 실행 파일 형식, 데이터 배치, 링킹 등 바이너리 수준의 약속

---

## 2. x86 32비트 호출 규약 비교

| 규약 | 주요 환경 | 인자 전달 | 스택 정리 | 특징 |
|---|---|---|---|---|
| `cdecl` | 32비트 C, Linux GCC 등 | 오른쪽 인자부터 스택에 push | caller | 가변 인자를 구현하기 쉬움 |
| `stdcall` | 32비트 Windows API | 오른쪽 인자부터 스택 | callee | `ret n`으로 인자 공간까지 정리 |
| `fastcall` | 컴파일러/Windows 계열 | 첫 인자 일부를 레지스터, 나머지는 스택 | 보통 callee | 레지스터로 호출 비용 감소 |
| `thiscall` | 32비트 C++ 멤버 함수 | `this`는 보통 `ecx`, 나머지는 스택 | 구현에 따라 callee | 객체 필드를 `[ecx+offset]`으로 접근 |

세부 구현은 컴파일러와 플랫폼에 따라 차이가 있을 수 있다. 바이너리 분석에서는 파일 형식, 컴파일러, 코드 패턴을 함께 보고 판단한다.

---

## 3. cdecl

```c
void callee(int a1, int a2, int a3);
callee(1, 2, 3);
```

전형적인 caller:

```asm
push 3
push 2
push 1
call callee
add  esp, 12       ; caller가 인자 3개 × 4바이트 정리
```

callee는 `[ebp+8]`, `[ebp+12]`, `[ebp+16]`처럼 인자를 읽을 수 있다.

```text
[ebp+16]  a3
[ebp+12]  a2
[ebp+8]   a1
[ebp+4]   return address
[ebp]     saved ebp
```

caller가 정리하므로 `printf`처럼 인자 개수가 가변적인 함수에도 적합하다. callee는 자신이 받은 인자의 정확한 전체 개수를 몰라도 caller가 정리할 수 있다.

---

## 4. stdcall

인자 전달은 cdecl과 비슷하지만 callee가 복귀하면서 인자 공간까지 정리한다.

```asm
callee:
    push ebp
    mov  ebp, esp
    ; ...
    leave
    ret 12          ; 반환 주소 pop + 추가로 12바이트 정리
```

디스어셈블러가 `retn 12`로 표시할 수도 있다. caller의 `call` 뒤에 `add esp, 12`가 없고 callee의 `ret 12`가 보인다면 stdcall 계열을 의심할 수 있다.

---

## 5. fastcall

강의의 전형적인 32비트 fastcall에서는 첫 두 인자를 `ecx`, `edx`, 나머지를 오른쪽부터 스택으로 전달한다.

```asm
mov  ecx, 1
mov  edx, 2
push 3
call callee
```

정확한 레지스터와 정리 주체는 컴파일러별 변형이 있으므로 “fastcall”이라는 이름만으로 모든 세부 규칙을 단정하면 안 된다.

---

## 6. thiscall

32비트 C++ 멤버 함수에서는 숨은 `this` 포인터를 `ecx`로 전달하는 패턴을 자주 본다.

```cpp
class C {
public:
    int c;
    int d;
    unsigned long e;
    int foo(int a, int b) { return c + d + e + a + b; }
};
```

전형적인 접근:

```asm
mov eax, [ebp+8]     ; a
add eax, [ebp+12]    ; b
add eax, [ecx]       ; this->c
add eax, [ecx+4]     ; this->d
add eax, [ecx+8]     ; this->e
ret 8
```

객체의 시작 주소가 `this`이고 멤버는 클래스 레이아웃에 따른 오프셋으로 배치된다. 상속, 가상 함수, 정렬, 컴파일러 ABI 때문에 실제 레이아웃은 더 복잡할 수 있다.

---

## 7. x86-64 Linux System V AMD64 ABI

Linux x86-64 ELF에서 가장 중요한 함수 호출 규약이다.

### 정수·포인터 인자

| 순서 | 레지스터 |
|---:|---|
| 1 | `rdi` |
| 2 | `rsi` |
| 3 | `rdx` |
| 4 | `rcx` |
| 5 | `r8` |
| 6 | `r9` |
| 7 이후 | 스택 |

일반적인 정수·포인터 반환값은 `rax`에 있다. 부동소수점·벡터·구조체는 타입 분류에 따라 XMM 레지스터나 메모리 반환 규칙이 적용될 수 있다.

### 보존 레지스터

| caller-saved(호출로 깨질 수 있음) | callee-saved(사용하면 callee가 복원) |
|---|---|
| `rax`, `rcx`, `rdx`, `rsi`, `rdi`, `r8`~`r11` | `rbx`, `rbp`, `r12`~`r15` |

`rsp`는 정상 복귀 시 caller가 기대하는 위치로 되돌아와야 한다.

### 스택 인자 정리

System V AMD64에서는 caller가 스택으로 전달한 추가 인자 공간을 정리한다.

```asm
push 7                    ; 7번째 인자
mov  r9d, 6
mov  r8d, 5
mov  ecx, 4
mov  edx, 3
mov  esi, 2
mov  rdi, first_arg
call callee
add  rsp, 8               ; caller가 스택 인자 정리
```

실제 코드는 호출 전 16바이트 스택 정렬을 맞추기 위해 패딩을 추가할 수 있다.

---

## 8. System V 호출을 단계별로 추적

예시 C 코드:

```c
unsigned long long callee(
    unsigned long long a1,
    int a2, int a3, int a4, int a5, int a6, int a7
) {
    return a1 + a2 + a3 + a4 + a5 + a6 + a7;
}
```

### 1단계 — 인자 전달

첫 여섯 개는 `rdi,rsi,rdx,rcx,r8,r9`, 일곱 번째는 스택에 둔다.

GDB 확인:

```gdb
break *caller+<call 오프셋>
run
info registers rdi rsi rdx rcx r8 r9
x/4gx $rsp
```

### 2단계 — 반환 주소 저장

`call callee`가 실행되면 다음 명령의 주소가 `[rsp]`에 저장된다.

```gdb
si
x/gx $rsp
x/i *(void **)$rsp
```

### 3단계 — caller 프레임 저장

전형적인 callee 프롤로그:

```asm
push rbp
mov  rbp, rsp
```

첫 명령은 caller의 `rbp`를 스택에 보존한다. callee가 `rbp`를 사용했으므로 반환 전에 원래 값으로 복원해야 한다.

### 4단계 — 새 프레임 공간 할당

지역 변수가 필요하면:

```asm
sub rsp, 0x30
```

컴파일러가 변수를 레지스터에만 유지하면 별도 공간이 없을 수도 있다.

### 5단계 — 반환값 배치

계산 결과를 `rax`에 둔다.

```gdb
break *callee+<ret 직전>
continue
print/x $rax
```

### 6단계 — 복귀

```asm
leave           ; 또는 pop rbp 등 최적화된 형태
ret
```

caller는 필요하면 추가 인자용 스택 공간을 되돌리고 `rax`의 반환값을 사용한다.

---

## 9. 스택 정렬

System V AMD64는 함수 호출 경계의 스택 정렬 규칙을 정의한다. 실무적으로 caller는 `call`을 실행하기 전 `rsp`가 16바이트 정렬 상태가 되도록 준비한다. `call`이 8바이트 반환 주소를 push하므로 callee 진입 직후의 `rsp` 하위 비트는 그에 맞게 달라진다.

정렬을 어기면 일반 코드가 우연히 동작할 수 있지만, 정렬된 SIMD 메모리 접근을 사용하는 라이브러리 함수에서 충돌할 수 있다.

---

## 10. 함수 호출 규약과 시스템 콜 규약 구분

| 항목 | System V 함수 호출 | Linux x86-64 시스템 콜 |
|---|---|---|
| 번호 | 없음, 함수 주소로 호출 | `rax`에 syscall 번호 |
| 인자 1~3 | `rdi,rsi,rdx` | `rdi,rsi,rdx` |
| 인자 4 | `rcx` | `r10` |
| 다음 인자 | `r8,r9,stack` | `r8,r9` |
| 호출 명령 | `call` | `syscall` |
| 반환 | 보통 `rax` | `rax` |

두 규약을 섞지 않는 것이 중요하다.

## 핵심 체크

- cdecl은 caller, stdcall은 callee가 인자용 스택을 정리한다.
- 32비트 fastcall은 일부 인자를 레지스터로 전달한다.
- thiscall의 `ecx`는 흔히 `this` 포인터다.
- System V AMD64의 정수 인자 6개는 `rdi,rsi,rdx,rcx,r8,r9`다.
- 일반 정수 반환값은 `rax`다.
- caller-saved와 callee-saved 레지스터의 책임을 구분한다.
- 외부 함수 호출 전 스택 정렬을 지킨다.

## 출처 및 이용 안내

이 문서는 Dreamhack 강의를 학습 목적으로 요약·재구성한 개인 학습 노트다. 원문과 GDB 분석 예시는 [Background: Calling Convention](https://learn.dreamhack.io/54)에서 확인한다.

