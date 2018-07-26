---
title: HanoiFactory[CF]
date: 2017-02-24 20:43:46
tags: 数据结构
mathjax: true
---

## [题目链接](http://codeforces.com/problemset/problem/777/E)

### 题意

汉诺塔由无数个空心圆柱堆积而成。现有$N$个空心圆柱,第$i$个圆柱的内径，外径高分别是$a_i,b_i,h_i$,问这些圆柱最高能堆多高的塔，堆积方式是从下往上:
- 第$x$个的外径$b\_x $要大于等于第$x+1$个的外径$b\_{x+1}(b\_x \ge b\_{x+1})$
- 第$x$个的内径$a\_x $要小于第$x+1$个的外径$b\_{x+1}(a\_x \lt b\_{x+1})$

### 题外话
---
这题的解法并不难，不过有个有意思的trick。
<!-- more -->
而且这是**第一次**打CF能够ak。所以随便记录一下.

### 解法:
---
可以贪心dp等方式，我用的是线段树.
首先把ab离散化。按b从大到小排序，这样就能保证排序后的第i个圆柱来说，前面i-1个圆柱都满足第一个条件.
然后我们就可以用线段树维护前i-1个中每一个a。能够堆的最高的塔.对于第i个圆柱来说.只要找前面所有的a中小于b的。能够堆得最高的.再加上第i个的高度，就是第i个圆柱能够堆的最高高度了.同时把ai在线段树中更新一下.时间复杂度$O(nlgn)$.

然后有意思的trick来了.这个trick让很多人都wa了至少一次.
当圆柱的b不同的时候，从大到小排序.那么当b相同的时候呢?这里正确的是把a也从大到小排序。因为这样才能让后面能堆得尽量高，有个例子
```cpp
4
3 7 2
5 7 2
4 7 2
4 4 2
```
这里如果a不从大到小排序，那么对于[3 7 2]这个圆柱来说，最高的是2，后面[4 4 2]的时候只能搭在[3 7 2]上面结果为4.这是不对的。
为当a从大到小排后[3 7 2]可以搭在另外两个上面高度为6.然后[4 4 2]再塔在[3 7 2]上面。结果为8.

---
```cpp
#include <bits/stdc++.h>

typedef long long LLONG;
typedef std::pair<int, int> PII;
typedef std::map<int, int> MII;
typedef std::pair<PII, int> PIIAI;

const int N = 1e5 + 10;

PIIAI towers[N];
MII hs;
int fp[N];
LLONG sg[N << 4];

inline void modify(int l, int r, int x, LLONG mv, int v) {
  if (l >= r) {
    sg[v] = mv;
    return;
  }
  int mid = l + r >> 1;
  if (mid >= x) {
    modify(l, mid, x, mv, v << 1);
  } else {
    modify(mid + 1, r, x, mv, v << 1 | 1);
  }
  sg[v] = std::max(sg[v << 1], sg[v << 1 | 1]);
}

inline LLONG query(int l, int r, int x, int y, int v) {
  if (l >= x && r <= y) {
    return sg[v];
  }
  int mid = l + r >> 1;
  LLONG rlt = 0;
  if (mid >= x) {
    rlt = query(l, mid, x, y, v << 1);
  }
  if (mid < y) {
    rlt = std::max(rlt, query(mid + 1, r, x, y, v << 1 | 1));
  }
  return rlt;
}

int main() {
  int n;
  scanf("%d", &n);
  for (int i = 0; i < n; ++i) {
    int a, b, h;
    scanf("%d%d%d", &a, &b, &h);
    hs[a] = 1;
    hs[b] = 1;
    towers[i] = {{a, b}, h};
  }
  int pn = 0;
  for (MII::iterator iter = hs.begin(); iter != hs.end(); ++iter) {
    iter->second = ++pn;
  }
  std::sort(towers, towers + n, [](PIIAI const & a, PIIAI const & b) {
    if (a.first.second == b.first.second) { return a.first.first > b.first.first; }
    return a.first.second > b.first.second;
  });
  for (int i = 0; i < n; ++i) {
    int a = towers[i].first.first;
    int b = towers[i].first.second;
    int f = hs[b];
    int ff = hs[a];
    LLONG cur = query(1, pn, 1, f - 1, 1);
    modify(1, pn, ff, cur + towers[i].second, 1);
  }
  printf("%I64d\n", sg[1]);
  return 0;
}
```

---
顺便记录下第一次ak截图.
![cffirstak](/img/cfak.png)