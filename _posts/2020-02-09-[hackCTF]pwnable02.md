---
layout: post
title:  "[hackCTF]Basic BOF #2"
date:   2020-02-09
---

hackCTF pwnable 두번째 문제
!(favicon)(https://drive.google.com/uc?id=1EPkDaLZatWWYaPyJ3wVlOrAu-eubvG9c)

### ida - main
***
```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  char s; // [esp+Ch] [ebp-8Ch]
  void (*v5)(void); // [esp+8Ch] [ebp-Ch]

  v5 = (void (*)(void))sup; // sup 함수의 주소를 v5에 저장
  fgets(&s, 133, stdin); // stdin으로부터 133바이트를 입력 받아 s에 저장
  v5(); // v5가 가리키는 함수 실행
  return 0;
}
```
### ida - sup
***
```c
int sup()
{
  return puts(&s); // s에 저장된 문자열 출력
}
```

### ida - shell
***
```c
int shell()
{
  return system("/bin/dash"); // 목표!
}
```
// 그냥 함수 실행했을 때 이미지 출력  
fgets로 133바이트를 입력 받을 때 v5에 shell의 주소로 덮으면 v5()를 실행하면 shell 함수가 실행된다.

### exploit.py
***
```python

```
문제 해결
