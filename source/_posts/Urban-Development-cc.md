---
title: Urban-Development
date: 2016-11-05 18:33:01
tags: 数据结构
mathjax: true
---

## [题目链接](https://www.codechef.com/NOV16/problems/URBANDEV)

--- 

### 题意
有$N$条平行于$x$轴或者$y$轴的线段，在两条线段的交点需要安放交通灯，如果是两个端点的交点，则不放。任意两条方向相同的线段不相交。求有需要放置多少个灯，并输出每条线段上有多少个
- $1 \le N \le 1e5$
- 所有坐标$1 \le x_i, y_i \le 1e5 $

---
<!-- more -->
### 解法
我们记平等于$x$轴的线段为$sgx$，平行于$y$轴的为$sgy$.
我们先求平行于$x$轴的线段上面有多少个灯($y$轴同理)。
记$f(x, y)=$与线段$(1, y)->(x, y)$相交的线段数量, 等于$x_i<=x$的所有$sgy$除去两边的$sgy$
对于线段$(x, y)->(x1, y)$，与之相交的平行于$y$轴的线段数量$ans=f(x1, y)-f(x-1, y)$.
我们把$sgx, sgy$按从左到右排序，用两个树状数组维护$y$轴方向每个端点的数量($sgy$的两个端点分别维护)，这样$f(x, y)$可以$O(lgn)$算出。
从左到右扫描过去，可算出所有$sgx$的交点数量.

把$x,y$轴算出来后，最后再减把两个端点相交的情况，因为这是不需要放灯的.这个比较简单，我们把$sgx$的两个端点保存下来，再枚举所有$sgy$的所有端点，如果找到对应的交点，则两条线段都减1. 我用`map`复杂度$O(lgn)$

最后总的灯的数量，把$sgx，sgy$选一个加起来就是了.
总的复杂度算$O(nlgn)$

---

```cpp
#include <bits/stdc++.h>

typedef long long LLONG;
typedef std::pair<int, int> PII;
const int N = 1e5 + 10;

struct Seg {
  int x, y, xx, yy, idx;
  void assign(int x1, int y1, int xx1, int yy1, int idx1) {
    x = x1; y = y1; xx = xx1; yy = yy1; idx = idx1;
  }
} sgx[N], sgy[N];
int answer[N];
int fbit[N];
int sbit[N];

inline int lowBit(int k) { return k & -k; }

inline void update(int bit[], int p, int v) {
  for(; p < N; p += lowBit(p)) {
    bit[p] += v;
  }
}

inline int query(int bit[], int p) {
  int rlt = 0;
  for(; p > 0; p -= lowBit(p)) {
    rlt += bit[p];
  }
  return rlt;
}

inline void solveX(int topx, int topy) {
  std::sort(sgy, sgy + topy, [](Seg const & a, Seg const & b) { return a.x < b.x; });
  std::sort(sgx, sgx + topx, [](Seg const & a, Seg const & b) { return a.x < b.x; });
  memset(fbit, 0, sizeof(fbit)); memset(sbit, 0, sizeof(sbit));
  for(int i = 0, j = 0; i < topx; ++i) {
    auto& v = sgx[i];
    for(; j < topy && sgy[j].x < v.x; ++j) {
      update(fbit, sgy[j].y, 1);
      update(sbit, sgy[j].yy, 1);
    }
    answer[v.idx] = -(j - (query(fbit, N - 10) - query(fbit, v.y))  - query(sbit, v.y - 1));
  }
  memset(fbit, 0, sizeof(fbit)); memset(sbit, 0, sizeof(sbit));
  std::sort(sgx, sgx + topx, [](Seg const & a, Seg const & b) { return a.xx < b.xx; });
  for(int i = 0, j = 0; i < topx; ++i) {
    auto& v = sgx[i];
    for(; j < topy && sgy[j].x <= v.xx; ++j) {
      update(fbit, sgy[j].y, 1);
      update(sbit, sgy[j].yy, 1);
    }
    answer[v.idx] += (j - (query(fbit, N - 10) - query(fbit, v.y))  - query(sbit, v.y - 1));
  }
}

inline void solveY(int topx, int topy) {
  std::sort(sgy, sgy + topy, [](Seg const & a, Seg const & b) { return a.y < b.y; });
  std::sort(sgx, sgx + topx, [](Seg const & a, Seg const & b) { return a.y < b.y; });
  memset(fbit, 0, sizeof(fbit)); memset(sbit, 0, sizeof(sbit));
  for(int i = 0, j = 0; i < topy; ++i) {
    auto& v = sgy[i];
    for(; j < topx && sgx[j].y < v.y; ++j) {
      update(fbit, sgx[j].x, 1);
      update(sbit, sgx[j].xx, 1);
    }
    answer[v.idx] = -(j - (query(fbit, N - 10) - query(fbit, v.x))  - query(sbit, v.x - 1));
  }
  memset(fbit, 0, sizeof(fbit)); memset(sbit, 0, sizeof(sbit));
  std::sort(sgy, sgy + topy, [](Seg const & a, Seg const & b) { return a.yy < b.yy; });
  for(int i = 0, j = 0; i < topy; ++i) {
    auto& v = sgy[i];
    for(; j < topx && sgx[j].y <= sgy[i].yy; ++j) {
      update(fbit, sgx[j].x, 1);
      update(sbit, sgx[j].xx, 1);
    }
    answer[v.idx] += (j - (query(fbit, N - 10) - query(fbit, v.x))  - query(sbit, v.x - 1));
  }
}

std::map<PII, int>hs;

inline void remove(int topx, int topy) {
  for(int i = 0; i < topx; ++i) {
    auto& v = sgx[i];
    hs[ {v.x, v.y}] = v.idx;
    hs[ {v.xx, v.yy}] = v.idx;
  }
  for(int i = 0; i < topy; ++i) {
    auto& v = sgy[i];
    PII p = {v.x, v.y};
    if(hs.find(p) != hs.end()) {
      --answer[v.idx];
      --answer[hs[p]];
    }
    p = {v.xx, v.yy};
    if(hs.find(p) != hs.end()) {
      --answer[v.idx];
      --answer[hs[p]];
    }
  }
}

int main() {
  int t;
  for(t = 1; t--;) {
    int n;
    scanf("%d", &n);
    int topx = 0, topy = 0;
    for(int i = 0; i < n; ++i) {
      int x, y, xx, yy;
      scanf("%d%d%d%d", &x, &y, &xx, &yy);
      if(x == xx) {
        if(y > yy) { std::swap(y, yy); }
        sgy[topy++].assign(x, y, xx, yy, i);
      } else {
        if(x > xx) { std::swap(x, xx); }
        sgx[topx++].assign(x, y, xx, yy, i);
      }
    }
    solveX(topx, topy);
    solveY(topx, topy);
    remove(topx, topy);
    LLONG sm = 0;
    for(int i = 0; i < topx; ++i) {
      sm += answer[sgx[i].idx];
    }
    printf("%lld\n", sm);
    for(int i = 0; i < n; ++i) {
      printf("%d ", answer[i]);
    }
  }
  return 0;
}

```

