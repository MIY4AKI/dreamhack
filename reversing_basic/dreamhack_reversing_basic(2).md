# 프로그램 제작

## 소스 코드 작성하기
- wsl로 한 것 기록

<br>

- 프로그램은 기계어로 작성되어 있기에 사람이 이해할 수 없음. 그래서 프로그래밍을 기계어로 하는것은 매우 힘들기에 C, Python 과 같은 프로그래밍 언어가 개발되었음

    - 여기서부터는<span style="color:red">vim</span>을 이용해 실습하였음

1. 리눅스에서 terminal 열기
2. `vi helloworld.c` 를 입력해 vim 편집기를 열기
3. `i` 를 눌러 편집 모드(INSERT 라는 문자열이 보이면 편집 모드)
4. 내용 작성
5. 작성 후 `ESC` 키로 빠져나온 후 `:wq` 를 입력해 종료
6. `cat` 명령어롤 통해 내용 확인 가능

<br>

=> 여기까지 진행하면 소스 코드가 만들어진 것(프로그램이 아님)

=> 컴파일 필요!! 

<br>

7. `gcc helloworld.c -o helloworld` 를 입력해서 소스코드를 컴파일 할 수 있음
8. `.\helloworld` 를 입력하면 컴파일 된 프로그램 실행 가능

*`xxd` 명령어는 terminal 에서 HxD 편집기와 비슷한 역할을 함*

=> `xxd helloworld | head -3`를 입력해서 초반 세줄만 출력하도록 해보면 첫 바이트가
`\x7f\x45\x4c\x46` 임. 뒤의 `\x45\x4c\x46` 은 ASCII 코드에서 ELF 를 뜻함

**ELF는 `Executable and Linkable Format`의 약자로 Linux나 여러 Unix 계열 운영체제에서 사용하는 표준 파일 형식임. 