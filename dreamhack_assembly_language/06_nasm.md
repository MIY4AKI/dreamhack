# Dreamhack Assembly Language — NASM

## 1. 어셈블러와 NASM

어셈블리어는 사람이 읽기 쉬운 기호로 CPU 명령을 표현한 저수준 언어이고, 어셈블러는 이를 기계어가 담긴 목적 파일로 변환한다.

- **Assemble**: 어셈블리어 → 기계어/목적 파일
- **Disassemble**: 기계어 → 어셈블리어 표현
- **NASM(Netwide Assembler)**: x86/x86-64용 어셈블러, Intel 문법 사용
- **GAS(GNU Assembler)**: GNU 도구 체인의 어셈블러, 전통적으로 AT&T 문법 사용

같은 x86-64 명령이라도 Intel 문법과 AT&T 문법은 피연산자 순서, 레지스터 표기, 크기 표기 등이 다르다. 이 Path는 NASM의 Intel 문법을 중심으로 한다.

---

## 2. 소스에서 실행 파일까지

```text
example.asm
   │  nasm
   ▼
example.o          목적 파일(Object file)
   │  ld
   ▼
example            실행 가능한 ELF
```

### 목적 파일

목적 파일은 소스가 기계어로 번역된 결과와 함께 심볼, 재배치 정보, 아직 해결되지 않은 외부 참조 등을 담는다. 각 소스 파일을 독립적으로 어셈블/컴파일할 수 있어 일부 파일이 바뀔 때 전체 프로젝트를 다시 번역할 필요가 없다.

### 링커

링커는 하나 이상의 목적 파일과 라이브러리를 결합한다.

- 외부 심볼 참조를 실제 정의와 연결한다.
- 코드와 데이터의 주소를 배치한다.
- 재배치 정보를 적용한다.
- 운영체제가 로드할 수 있는 최종 실행 파일을 만든다.

> `ld`는 정적 링커다. 실행 시 공유 라이브러리를 적재하는 런타임 동적 로더(`ld-linux...`)와 역할을 구분한다.

---

## 3. Hello, Dreamhack

```asm
section .data
    message db "Hello, Dreamhack!", 0x0a
    msg_len equ $ - message

section .text
    global _start

_start:
    ; write(1, message, msg_len)
    mov eax, 1
    mov edi, 1
    mov rsi, message
    mov edx, msg_len
    syscall

    ; exit(0)
    mov eax, 60
    xor edi, edi
    syscall
```

### 사용된 지시어

- `section .data`: 초기화된 데이터 정의
- `db`: 바이트 단위 데이터 정의(Define Byte)
- `equ`: 심볼을 상수 표현식과 연결
- `$`: 현재 위치. `$ - message`는 문자열 시작부터 현재까지의 바이트 길이
- `section .text`: 실행 코드
- `global _start`: `_start` 심볼을 링커에 공개

### 어셈블과 링크

```bash
nasm -f elf64 example.asm -o example.o
ld example.o -o example
./example
```

출력 형식 `elf64`는 64비트 리눅스 ELF 목적 파일을 뜻한다.

---

## 4. 설치

Ubuntu 계열:

```bash
sudo apt-get update
sudo apt-get install -y nasm binutils
nasm -v
ld --version
```

`binutils`에는 `ld`, `objdump`, `readelf` 같은 도구가 포함된다.

---

## 5. 주요 NASM 옵션

| 옵션 | 용도 |
|---|---|
| `-f <format>` | 목적 파일 형식 지정: `elf`, `elf64`, `bin`, `win32`, `win64` 등 |
| `-o <file>` | 출력 파일 이름 지정 |
| `-g` | 디버깅 심볼 포함 |
| `-F <format>` | 디버그 정보 형식 지정. Linux ELF에서는 `dwarf`를 자주 사용 |
| `-l <file>` | 소스와 생성 바이트를 대응시킨 리스팅 파일 생성 |
| `-I <path>` | `%include` 검색 디렉터리 추가 |
| `-P <file>` | 파일을 미리 포함하여 전처리 |
| `-D<macro>` | 커맨드라인에서 매크로 정의 |
| `-E` | 전처리 결과만 출력 |
| `-M` | 의존성 정보 생성 |
| `-w+error` | 경고를 오류로 취급 |
| `-O<level>` | 어셈블러 최적화 패스 수준 지정 |
| `-v` | NASM 버전 출력 |

디버깅 친화적으로 만들기:

```bash
nasm -f elf64 -g -F dwarf example.asm -o example.o
ld example.o -o example
gdb ./example
```

---

## 6. `%include`와 상수 파일

`constants.inc`:

```asm
%define SYS_WRITE 1
%define SYS_EXIT  60
```

`example.asm`:

```asm
%include "constants.inc"

section .text
global _start
_start:
    mov eax, SYS_EXIT
    xor edi, edi
    syscall
```

빌드:

```bash
nasm -I ./ -f elf64 example.asm -o example.o
```

`-I`에는 보통 포함 파일 그 자체가 아니라 검색할 **디렉터리**를 준다. 여러 파일에서 시스템 콜 번호나 구조체 오프셋을 공유할 때 유지보수에 도움이 된다.

---

## 7. 결과 검증 도구

```bash
file example.o
file example
readelf -h example
readelf -S example.o
objdump -d -M intel example
nm example.o
```

- `file`: 파일 형식과 아키텍처 확인
- `readelf -h`: ELF 헤더, 진입점, 아키텍처 확인
- `readelf -S`: 섹션 구성 확인
- `objdump -d -M intel`: Intel 문법으로 디스어셈블
- `nm`: 목적 파일의 심볼 확인

---

## 8. 흔한 오류

### `parser: instruction expected`

지시어 철자, Intel 문법, 콜론, 괄호를 확인한다.

### `symbol ... not defined`

라벨 철자, `%include` 경로, 외부 심볼 선언을 확인한다.

### 링커의 `undefined reference`

필요한 목적 파일이나 라이브러리가 링크 명령에 포함됐는지 확인한다. `extern`은 “다른 곳에 정의돼 있다”고 알릴 뿐 실제 구현을 제공하지 않는다.

### 실행 시 `Permission denied`

링크된 ELF에는 보통 실행 권한이 생기지만, 없다면 파일 권한과 마운트 옵션을 확인한다.

### 잘못된 출력 형식

64비트 코드를 `-f elf`로 조립하거나 Windows 형식을 Linux에서 바로 링크하면 맞지 않는다. 대상 운영체제·비트 수에 맞는 형식을 선택한다.

## 핵심 체크

- NASM은 실행 파일을 직접 완성하는 도구가 아니라 먼저 목적 파일을 만든다.
- 목적 파일을 실행 파일로 만들려면 링크 단계가 필요하다.
- `-f elf64`와 `-o`는 기본 빌드에서 가장 자주 사용한다.
- 외부 C 함수를 사용하면 libc와 동적 로더를 포함한 별도 링크 설정이 필요하다.
- 결과는 `file`, `readelf`, `objdump`로 검증한다.

## 출처 및 이용 안내

이 문서는 Dreamhack 강의를 학습 목적으로 요약·재구성한 개인 학습 노트다. 원문은 [NASM](https://learn.dreamhack.io/790)에서 확인한다.

