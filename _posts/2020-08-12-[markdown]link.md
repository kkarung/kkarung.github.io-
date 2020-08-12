---
layout: post
title:  "[Markdown] link 안에 괄호 있을 시 문제 해결"
date:   2020-08-12
categories: [Others]
tags: [Others]
permalink: '/Others'
---

[이렇게](https://kkarung.github.io/%EC%9E%90%EB%A3%8C%EA%B5%AC%EC%A1%B0/03) 목차를 넣어서 포스팅하던 도중 아래와 같이 링크 안에 괄호가 있을 경우에는 이동이 불가능한 경우를 발견했습니다.

원래는 이렇게 링크를 만들면

```Markdown
    [이진 탐색 (binary search)](#이진-탐색-(binary-search))
```

이렇게 보여서

[이진 탐색 (binary search)](#이진-탐색-(binary-search))

클릭하면 링크된 곳으로 이동해야 합니다.

<br><br>

시도한 방법은 아래 세 가지 방법입니다.

```
Link #1 - [이진 탐색 (binary search)][1]
[1]: (실제 주소)
Link #2 - [이진 탐색 (binary search)](#이진-탐색-%28binary-search%29)
Link #3 - [이진 탐색 (binary search)](#이진-탐색-\binary-search\))
```

해보니까 이거 셋 다 안 됩니다. 다 된다고 하던데 왜인지는 가능성이 너무 많아서 모르겠음

<br>

이렇게 링크를 만들고

```
<a href="#1">이진 탐색 (binary search)</a>
```

이렇게 하면

```
<a id="1">이진 탐색</a>
```

위에서 건 링크로 아래로 이동 가능하긴 함

[link](https://meta.stackexchange.com/questions/13501/links-to-urls-containing-parentheses){:target="_blank"}