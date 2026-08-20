# Heading level 1
## Heading level 2
### Heading level 3

이렇게 해서 Heading level은 6까지 가능함
<#과 Heading 사이는 공백으로 구분>

줄바꿈을 한번만 하면 문단 구분 X => 이어서 출력

this
is

this is the right

answer

만약 줄바꿈을 원한다면 문장 종료 이후 공백 2개 이상 사용

this will  
be connected

볼드체나 기울임체를 사용하고 싶다면

This is **Bold**  
This is *Italic*  
this is ***Bold and Italic***  
this is ~~Strikethrough~~  

인용을 넣고 싶다면
> I am a quote
>> Nested qoutes are Inside
>>> the more quotes are Inside

=> **호환성을 위해 인용의 윗줄과 아랫줄은 비워두는 것을 권장**


리스트
1. 이렇게
    1. 중복 리스트는 tab으로 구분

- 이러면 새로운 리스트


## Code를 표시하는 방법
한줄 단위라면 ``한개

` printf("a") `

code box 생성은 ```
```python
import os
os.system(`sudo rm -rf /`)
```

## Links
- [Dreamhack](https://dreamhack.io) 형식으로 추가 가능

## images
- ![alt text](image path) 형식으로 추가 가능

![Dreamhack](https://dreamhack.io/assets/dreamhack_logo_sg.png)

=> image path 경로에는 해당하는 사진 파일이 존재해야 함(현재 Dreamhack.png 삭제됨)

## 이스케이핑 문자
- 문법에 사용되어 출력되지 않는 문자는 역슬래시 이용

\\ \` \* 이런식으로

<br> => 얘는 HTML 태그(줄바꿈)
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>


# Markdown 문법 적용 문서 예시(dreamhack)

# Why we should be using Dreamhack

## What is Dreamhack?
![Dreamhack](https://dreamhack.io/assets/dreamhack_logo_sq.png)

Lorem ipsum dolor sit amet, `consectetur` adipiscing elit. Sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.

## Features of Dreamhack

### Unordered Features
- **Lorem ipsum dolor sit amet,**
- *consectetur adipiscing elit.*
    - `Sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.`

### Ordered Features
1. First Item
2. Second Item
   1. Subitem 2.1
   2. Subitem 2.2

## Why Dreamhack?

> Lorem ipsum dolor sit amet, consectetur adipiscing elit.

Sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.

## Dreamhack Link

[Dreamhack](https://dreamhack.io)

## Accessing by Code

```python
import requests
url = 'https://dreamhack.io'
resp = requests.get(url)
print(resp.text)
```