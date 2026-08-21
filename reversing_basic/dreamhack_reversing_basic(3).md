# 동적 분석
- **프로그램을 실행시키며**분석하는 것. 예를들어 튜토리얼을 스킵하고 게임하는 것. 게임하며 게임의 규칙을 익힌다면 이게 일종의 동적 분석이라고 할 수 있음

실습은 간단한 패스워드 검증 프로그램으로 진행함

=>[Dreamhack_Reversing_Practice](https://dreamhack-lecture.s3.amazonaws.com/uploads/reversing/intro/main)


리눅스 터미널에서 프로그램이 존재하는 경로에서 `.\main`을 실행하면 아래오 같이 됨
![alt text](image.png)

1. 이후 패스워드 입력을 `AAAA`로 넣어보면 너무 짧다고 뜬다
2. 그래서 이번엔 길게 `AAAAAAAAAAAAA` 이런식으로 입력해보면 너무 길다고 뜬다 
3. 그러다 `AAAAAAAA` 을 입력하면 
> 'your input is not a number'

라는 문구가 출력된다. 이를 통해 8글자의 패스워드라고 예측할 수 있다

4. 그리고 숫자가 아니라고 했기에 이번엔 숫자 8글자를 넣어보면 

![alt text](image-1.png)

이런식으로 출력된다. 즉 `output` 과 `wanted`가 출력되며 맞은 개수를 알려준다. 이를 통해 `output` 값이 `wanted` 값과 같아야 한다고 예측할 수 있음

5. '0' 을 입력했을때의 값이 0xb9가 나오고 '1'을 입력했을 때는 0x62가 되니 이번엔 `23456789`를 입력해서 각 숫자에 대한 `output` 값을 알아보자

|입력 숫자|`output`|
|---|---|
|0|0xb9|
|1|0x62|
|2|0xee|
|3|0x65|
|4|0x3b|
|5|0xb1|
|6|0xbd|
|7|0xb7|
|8|0xe8|
|9|0xe3|

6. 이제 이를 통해서 `output`에 `wanted`와 동일한 값을 넣으면 된다. 그러면 password가 `15728329`가 된다. 이제 이 값을 입력해보면 올바른 패스워드라는 메시지가 출력된다

![alt text](image-2.png)

