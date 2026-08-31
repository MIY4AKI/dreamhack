# Dreamhack Assembly Language Path — 강의별 필기 목차

이 폴더는 Dreamhack의 `Assembly Language` Path에 포함된 **실제 강의 12개**를 강의별 Markdown 문서로 나누어 정리한 개인 학습 노트다. Path의 퀴즈 8개, Lab 1개, 워게임 2개는 강의 수에 포함되지 않으므로 이 목차에서는 제외했다.

## Path 개요

- 대상: x86-64 어셈블리어를 처음 배우는 학습자
- 핵심 흐름: 컴퓨터 기초 → 메모리와 디버거 → NASM → x86 명령어 → 호출 규약 → 프로그램 작성
- 원본 Path: [Dreamhack — Assembly Language](https://dreamhack.io/lecture/paths/assembly-language)
- 확인 당시 구성: 8개 Unit, 12개 강의

## 강의별 필기

| 순서 | Unit | 실제 강의 | 필기 파일 |
|---:|---|---|---|
| 1 | 컴퓨터 과학 기초 | 컴퓨터 과학 기초 | [dreamhack_cs_basic_summary.md](./01_dreamhack_cs_basic_summary.md) |
| 2 | 컴퓨터 아키텍처 기초 | Background: Computer Architecture | [02_computer_architecture.md](./02_computer_architecture.md) |
| 3 | Linux 메모리 레이아웃 | Background: Linux Memory Layout | [03_linux_memory_layout.md](./03_linux_memory_layout.md) |
| 4 | GDB | GDB | [04_gdb.md](./04_gdb.md) |
| 5 | GDB | Exercise: GDB | [05_gdb_exercise.md](./05_gdb_exercise.md) |
| 6 | NASM | NASM | [06_nasm.md](./06_nasm.md) |
| 7 | x86 어셈블리 | x86 Assembly: Essential Part (1) | [07_x86_assembly_part1.md](./07_x86_assembly_part1.md) |
| 8 | x86 어셈블리 | x86 Assembly: Essential Part (2) | [08_x86_assembly_part2.md](./08_x86_assembly_part2.md) |
| 9 | 함수 호출 규약 | Background: Calling Convention | [09_calling_convention.md](./09_calling_convention.md) |
| 10 | Assembly Training | 구구단 구현하기 | [10_assembly_training_gugudan.md](./10_assembly_training_gugudan.md) |
| 11 | Assembly Training | 진법 변환기 구현하기 | [11_assembly_training_base_converter.md](./11_assembly_training_base_converter.md) |
| 12 | Assembly Training | 텍스트 편집기 구현하기 | [12_assembly_training_text_editor.md](./12_assembly_training_text_editor.md) |

## 추천 학습 방법

1. 1~3강에서 숫자 표현, 레지스터, 메모리 세그먼트를 먼저 이해한다.
2. 4~5강의 GDB 명령은 읽기만 하지 말고 작은 바이너리에 직접 실행한다.
3. 6강의 `nasm → object file → ld → ELF` 흐름을 한 번 손으로 재현한다.
4. 7~9강은 명령어를 외우기보다 작은 코드가 레지스터·플래그·스택을 어떻게 바꾸는지 추적한다.
5. 10~12강은 완성 코드를 그대로 복사하기보다 함수 하나씩 다시 작성하고 GDB로 검증한다.

## 이용 안내

각 문서는 원 강의를 그대로 옮긴 자료가 아니라, 학습을 위해 내용을 요약·재구성하고 필요한 기술적 주의점을 덧붙인 개인 필기다. 강의의 시각 자료, 실습 파일, 퀴즈 및 정확한 문맥은 각 문서 마지막에 적힌 Dreamhack 원문 링크에서 확인한다.
