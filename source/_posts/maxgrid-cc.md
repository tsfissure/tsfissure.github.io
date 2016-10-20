---
title: MAXGRID[CodeChef]
date: 2016-08-22 10:58:56
mathjax: true
tags: 数据结构
---



## [题目链接](https://www.codechef.com/AUG16/problems/MAXGRID)

### 题意
在二维平面上有$N$个带权值点，你需要找一个长度不大于$L$的矩形区域，使得区域内权值和最大，记为$S$，并且要求找一个最小边长的矩形区域，也使得区域内权值和为$S$，输出$S$和最小边长
- $1\le N,L\le1e5$
- 坐标$1\le x,y\le 1e5$

<!-- more -->

### 解法
我们把$S$求出来后，最小边长二分就可以了
求$S$的时候，我们把所有点按$x$轴从小到大排序
用一个线段树维护$x$轴为矩形右边界每一个$y$为右下角顶点边长为$L$时的权值和的最大值
也就是说对于点$(x_1,y_1)\lbrace x-L\le x_1\le x\rbrace$，将会给线段树的区间$[y_1-L,y_1]$都有贡献，因为这个区间内的点作为矩形右下角顶点时，矩形都能包含这个点.
我们把线段树从左到右扫描完所有点，就能算出边长为$L$时的最大和了。
```cpp
#include <bits/stdc++.h>

typedef long long LLONG;
typedef std::pair<int, int>PII;
#define LSON v * 2
#define RSON v * 2 + 1

const int N = 1e5 + 10;
const int MOD = 1e9 + 7;

std::pair<PII, int>pots[N];
LLONG mx[N << 2];
LLONG lazy[N << 2];

inline void pushDown(int v) {
    if (lazy[v]) {
        mx[LSON] += lazy[v];
        mx[RSON] += lazy[v];
        lazy[LSON] += lazy[v];
        lazy[RSON] += lazy[v];
        lazy[v] = 0;
    }
}

inline void pushUp(int v) {
    mx[v] = std::max(mx[LSON], mx[RSON]);
}

inline void prepare(int l, int r, int v) {
    mx[v] = lazy[v] = 0;
    if (l >= r) {
        return;
    }
    int mid = l + r >> 1;
    prepare(l, mid, LSON);
    prepare(mid + 1, r, RSON);
    pushUp(v);
}

inline void update(int l, int r, int x, int y, int z, int v) {
    if (l >= x && r <= y) {
        mx[v] += z;
        lazy[v] += z;
        return;
    }
    pushDown(v);
    int mid = l + r >> 1;
    if (mid >= x) update(l, mid, x, y, z, LSON);
    if (mid < y) update(mid + 1, r, x, y, z, RSON);
    pushUp(v);
}

inline LLONG computeMaxSum(int n, int l) {
    int i, j, last = 0;
    LLONG rlt = 0;
    const int M = N - 5;
    prepare(1, M, 1);
    for (i = 0; i < n; i = j) {
        for (j = i; j < n && pots[j].first.first == pots[i].first.first; ++j) {
            update(1, M, std::max(1, pots[j].first.second - l + 1), pots[j].first.second, pots[j].second, 1);
        }
        for (; last <= i && pots[last].first.first + l <= pots[i].first.first; ++last) {
            update(1, M, std::max(1, pots[last].first.second - l + 1), pots[last].first.second, -pots[last].second, 1);
        }
        rlt = std::max(rlt, mx[1]);
    }
    return rlt;
}

int main() {
    int n, m;
    scanf("%d%d", &n, &m);
    for (int i = 0; i < n; ++i) {
        int x, y, v;
        scanf("%d%d%d", &x, &y, &v);
        pots[i] = std::make_pair(std::make_pair(x, y), v);
    }
    std::sort(pots, pots + n);
    LLONG maxVal = computeMaxSum(n, m);
    int l = 1, r = m, mid;
    for (; l <= r;) {
        mid = l + r >> 1;
        if (computeMaxSum(n, mid) >= maxVal) r = mid - 1;
        else l = mid + 1;
    }
    printf("%lld %d\n", maxVal, r + 1);
    return 0;
}

```