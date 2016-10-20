---
title: 商品推荐跑马灯[计蒜客]
date: 2016-01-17 23:28:35
tags: 数据结构
mathjax: true
---

### [题目链接](http://nanti.jisuanke.com/t/441)

### 题意
- 有长度为$N$的数列$A[1...N]$,对于每次询问$L,R$,求:
$$ answer = \sum\_{i=l}^r\sum\_{j=i}^rf(ij)=回文序列?\sum\_{k=i}^jA\_k:0 $$
<!-- more -->

### 算法
- `manacher`回文串 & 线段树

### 思路
- 我们利用`manacher`算法插入辅助数据后，每一个回文序列$[l,r]$都有一个中点$p$,如果$p$在询问$[L,R]$的**中间偏左**，那么我们可以get新区间$[max(L,l),p]$，这个区间内每一位置作为回文序列的左端点，$p$为中点形成的序列都对询问有贡献.贡献为：如果位置$p1$是插入的辅助数据,贡献为0,否则贡献为$value=sum(A\_{p1..p})\*2-A\_p=(preSum\_p-preSum\_{p1-1})\*2-A\_p$,用段树来维护一下这个公式就可以算了， **中间偏右**同理.

- 对于询问$[L,R]$, 我们可以拆分成两部分$[L,Mid]$和$[Mid+1,R]\(Mid = L + R >> 1)$,从左到右和从右到左分别排序离线询问一次，加起来就是结果了。

``` cpp
#include <bits/stdc++.h>

typedef long long LLONG;
typedef std::pair<int, int> PII;
const int mod = 1e9 + 7;
const int ciBufferSize = 2e5 + 10;
#define DEFMID  int mid = l + r >> 1
#define LSON    (v * 2)
#define RSON    (v * 2 + 1)

int A[ciBufferSize];
int pA[ciBufferSize];
int preSum[ciBufferSize];
LLONG answer[ciBufferSize];
LLONG preSumSum[ciBufferSize];

enum RunOrder {
  RO_LTOR,    // 左到右
  RO_RTOL,    // 右到左
  RO_MAX,
};
RunOrder order = RO_MAX;

struct SegTree {
  void build(int l, int r, int v) {
    add[v] = 0;
    add2[v] = 0;
    lazy[v] = 0;
    lazy2[v] = 0;
    if (l >= r) return;
    DEFMID;
    build(l, mid, LSON);
    build(mid + 1, r, RSON);
  }

  LLONG gather(int v, int l, int r) {
    int num = r - l + 1;
    switch (order) {
    case RO_LTOR:
      num = num + (l & 1) >> 1;
      break;
    case RO_RTOL:
      num = num + (!(l & 1)) >> 1;
      break;
    }
    return add[v] + lazy[v] * num;
  }

  LLONG gather2(int v, int l, int r) {
    return add2[v] + lazy2[v] * (preSumSum[r] - preSumSum[l - 1]);
  }

  void pushUp(int v, int l, int r) {
    DEFMID;
    add[v] = gather(LSON, l, mid) + gather(RSON, mid + 1, r);
    add2[v] = gather2(LSON, l, mid) + gather2(RSON, mid + 1, r);
  }

  void pushDown(int v) {
    if (lazy[v]) {
      lazy[LSON] += lazy[v];
      lazy[RSON] += lazy[v];
      lazy[v] = 0;
    }
    if (lazy2[v]) {
      lazy2[LSON] += lazy2[v];
      lazy2[RSON] += lazy2[v];
      lazy2[v] = 0;
    }
  }

  void update(int l, int r, int x, int y, int z, int v) {
    if (l >= x && r <= y) {
      lazy[v] += z;
      ++lazy2[v];
      return;
    }
    pushDown(v);
    DEFMID;
    if (mid >= x) update(l, mid, x, y, z, LSON);
    if (mid < y) update(mid + 1, r, x, y, z, RSON);
    pushUp(v, l, r);
  }

  LLONG query(int l, int r, int x, int y, int v) {
    if (l >= x && r <= y) {
      return gather(v, l, r) - gather2(v, l, r) * 2;
    }
    pushDown(v);
    LLONG result = 0;
    DEFMID;
    if (mid >= x) result = query(l, mid, x, y, LSON);
    if (mid < y) result += query(mid + 1, r, x, y, RSON);
    pushUp(v, l, r);
    return result;
  }

private:
  LLONG add[ciBufferSize << 2]; //presum * 2 - Ap
  LLONG lazy[ciBufferSize << 2];
  LLONG add2[ciBufferSize << 2];  //区间更新了多少次
  LLONG lazy2[ciBufferSize << 2];
} tree;

struct Query {
  int l, r, id;
  Query() {}
  Query(int ll, int rr, int idd): l(ll), r(rr), id(idd) {}
} qryL[ciBufferSize >> 1], qryR[ciBufferSize >> 1];

inline void computePA(int n) {
  int mr = 0, id = 0;
  for (int i = 1; i <= n; ++i) {
    pA[i] = mr > i ? std::min(pA[(id << 1) - i], mr - i) : 1;
    for (; A[i - pA[i]] == A[i + pA[i]]; ++pA[i]);
    if (i + pA[i] >= mr) {
      mr = i + pA[i];
      id = i;
    }
  }
}

inline void solve(Query qry[], int n, int q) {
  std::sort(qry, qry + q, [](const Query & a, const Query & b) {
    if (a.r == b.r) return a.l < b.l;
    return  a.r < b.r;
  });
  tree.build(1, n, 1);
  for (int i = 1, j = 0; i <= n; ++i) {
    int l = i - pA[i] + 1, v = preSum[i] * 2 - A[i];
    tree.update(1, n, l, i, v, 1);
    for (; j < q && qry[j].r <= i; ++j) {
      if (qry[j].l <= qry[j].r)
        answer[qry[j].id] += tree.query(1, n, qry[j].l, qry[j].r, 1);
    }
  }
}

inline void reverseData(int n) {
  for (int l = 1, r = n; l < r;) {
    std::swap(pA[l], pA[r]);
    std::swap(A[l++], A[r--]);
  }
}

inline bool isSynthesize(int p) {
  switch (order) {
  case RO_LTOR:
    return !(p & 1);
  case RO_RTOL:
    return p & 1;
  }
  return false;
}

inline void computePreSum(int n) {
  preSum[0] = 0;
  preSumSum[0] = 0;
  for (int i = 1; i <= n; ++i) {
    preSum[i] = preSum[i - 1] + A[i];
    preSumSum[i] = preSumSum[i - 1] + (isSynthesize(i) ? 0 : preSum[i - 1]);
  }
}

inline int computeRevPos(int n, int t) {
  return n - t + 1;
}

int main() {
  int n, q;
  for (; ~scanf("%d%d", &n, &q);) {
    A[0] = -111;
    for (int i = 0, j = 0, k; i < n; ++i) {
      scanf("%d", &k);
      A[++j] = k;
      A[++j] = 0;
    }
    n <<= 1;
    A[n | 1] = -222;
    for (int i = 0; i < q; ++i) {
      int l, r;
      scanf("%d%d", &l, &r);
      if (l > r) std::swap(l, r);
      l = (l << 1) - 1;
      r = (r << 1) - 1;
      DEFMID;
      qryL[i] = Query(l, mid, i);
      qryR[i] = Query(computeRevPos(n, r), computeRevPos(n, mid + 1), i);
    }
    computePA(n);
    order = RO_LTOR;
    computePreSum(n);
    solve(qryL, n, q);

    order = RO_RTOL;
    reverseData(n);
    computePreSum(n);
    solve(qryR, n, q);
    for (int i = 0; i < q; ++i)
      printf("%lld\n", answer[i]);
  }
  return 0;
}
```