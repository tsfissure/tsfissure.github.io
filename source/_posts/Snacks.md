---
title: Snacks[HDU5692]
date: 2016-05-21 23:15:42
tags: 数据结构
mathjax: true
---

## [题目链接](http://acm.hdu.edu.cn/showproblem.php?pid=5692)

### 题意
给一$N$节点棵树，根节点为0,每个节点有权值$A_i$每次操作为以下二者之一:
- 给定节点X，求根节点到X和其子树中的某个点上权值和，最大是多大
- 修改节点X的值为Y

<!-- more -->

### 吐槽
这是2016百度之星初赛第一场的C题，守题人大坑比啊，题目提示说要加栈，也把加栈代码在题目中给了出来，于是大多参赛的人应该都是为了方便直接复制了这行代码，我也是。结果就是，不断的RE(STACK_OVERFLOW),检查了代码好多遍，没有发现有问题，于是各种尝试，试到17次的时候我把所有递归都去掉了，发现还是RE，这才反应过来，申请的栈空间还是太大了，改小一点后AC了。然后又发现，，题目描述改过了，提示加栈的那行代码，变小了，mdzz真是坑，改了题面也不说一声，我这类没有怀疑题面的人，就白白浪费了一个多小时，本来可以一A的.

### 解法
题目描述说找一条不重复走的路径必须经过点X,所以也就是到X的它的子树中的某个节点了，所以把树转成线后，只要知道所有节点到根节点的区间，区间维护一下最大值就OK了，因为点X和其子树节点，是一段连续区间,查询的时候直接区间最大值查询即可，修改的时候，也是区间同时变化，用线段树维护一下即可。

```cpp
#include <stdio.h>
#include <string.h>
#include <iostream>
#include <algorithm>
#include <map>
#include <vector>

#define DEFMID  int mid = l + r >> 1
#define L_SON    (v << 1)
#define R_SON    (v << 1 | 1)
#define L_RANGE  l, mid
#define R_RANGE  mid + 1, r

typedef long long LLONG;
const int mod = 1e9 + 7;
const int N = 1e5 + 10;

struct EdgeSet {
  int to, next;
} edges[N << 1];
int head[N], etot;

int p[N], fp[N], maxr[N]; //maxr为点u和子树形成区间的右端点
LLONG dis[N], A[N];
int stk[N], parent[N];

LLONG maxToRoot[N << 2];
LLONG lazy[N << 2];

inline void addEdge(int u, int v) {
  edges[etot].to = v;
  edges[etot].next = head[u];
  head[u] = etot++;
}

inline void assignPos(int u, int pos) {
  p[u] = pos;
  fp[pos] = u;
}

inline void assignParent(int u, int v) {
  parent[v] = u;
  if (~u)
    dis[v] = dis[u] + A[v];
  else dis[v] = A[v];
}

inline void getPos(int u) {
  int toper =  -1;
  stk[++toper] = u;
  assignPos(u, 1);
  assignParent(-1, u);
  int pos = 1;
  for (; ~toper;) {
    int u = stk[toper];
    int e = head[u];
    if (~e) {
      int v = edges[e].to;
      if (v != parent[u]) {
        stk[++toper] = v;
        assignPos(v, ++pos);
        assignParent(u, v);
      }
      head[u] = edges[e].next;
    } else {
      maxr[u] = pos;
      --toper;
    }
  }
}

inline void pushUp(int v) {
  maxToRoot[v] = std::max(maxToRoot[L_SON], maxToRoot[R_SON]);
}

inline void build(int l, int r, int v) {
  lazy[v] = 0;
  if (l >= r) {
    maxToRoot[v] = dis[fp[l]];
    return;
  }
  DEFMID;
  build(L_RANGE, L_SON);
  build(R_RANGE, R_SON);
  pushUp(v);
}

inline void pushDown(int v) {
  if (lazy[v]) {
    lazy[L_SON] += lazy[v];
    lazy[R_SON] += lazy[v];
    maxToRoot[L_SON] -= lazy[v];
    maxToRoot[R_SON] -= lazy[v];
    lazy[v] = 0;
  }
}

inline void modify(int l, int r, int x, int y, int z, int v) {
  if (l >= x && r <= y) {
    maxToRoot[v] -= z;
    lazy[v] += z;
    return;
  }
  pushDown(v);
  DEFMID;
  if (mid >= x) modify(L_RANGE, x, y, z, L_SON);
  if (mid < y) modify(R_RANGE, x, y, z, R_SON);
  pushUp(v);
}

inline LLONG query(int l, int r, int x, int y, int v) {
  if (l >= x && r <= y) {
    return maxToRoot[v];
  }
  pushDown(v);
  LLONG rlt = -1LL << 60;
  DEFMID;
  if (mid >= x) rlt = query(L_RANGE, x, y, L_SON);
  if (mid < y) rlt = std::max(rlt, query(R_RANGE, x, y, R_SON));
  pushUp(v);
  return rlt;
}

int main() {
  int n, q;
  int u, v, w;
  int t, tt = 0;
  scanf("%d", &t);
  for (; t--;) {
    printf("Case #%d:\n", ++tt);
    int q;
    scanf("%d%d", &n, &q);
    memset(head, -1, n * sizeof(int));
    etot = 0;
    for (int i = 1; i < n; ++i) {
      scanf("%d%d", &u, &v);
      addEdge(u, v);
      addEdge(v, u);
    }
    for (int i = 0; i < n; ++i)
      scanf("%I64d", A + i);
    getPos(0);
    build(1, n, 1);
    int oper = 0;
    for (; q--;) {
      scanf("%d", &oper);
      if (1 == oper) {
        scanf("%d", &u);
        printf("%I64d\n", query(1, n, p[u], maxr[u], 1));
      } else {
        scanf("%d%d", &u, &v);
        modify(1, n, p[u], maxr[u], A[u] - v, 1);
        A[u] = v;
      }
    }
  }

  return 0;
}

```

### 彩蛋

为了避免再次这种情况发生，我决定温习一遍用非递归的树形转线形，参赛的时候想写，但感觉不太稳，万一debug半天，不划算，现在在复习一遍，下次应该没问题了吧。

非递归的原理就是，用栈来维护模拟递归，对于一个点u，和它相连的节点有 $v_1,v_2,v_3...v_k$,我们每次只加入一个点，加进配重，然后进行下次循环，如果k个点加完了，就说明点u的子树处理完了，弹出点u.

PS:这样的代码，边只能遍历一次，如果后面还需要遍历边，记得要把head数组先备份.


```cpp
inline void getPos(int u) { //根节点u
  int toper =  -1; 
  stk[++toper] = u;  //这三行是准备根节点的信息
  assignPos(u, 1);
  assignParent(-1, u);
  int pos = 1;
  for (; ~toper;) {
    int u = stk[toper];
    int e = head[u];
    if (~e) {
      int v = edges[e].to;  //下一个连接的点v
      if (v != parent[u]) {
        stk[++toper] = v; //如果是新节点，加进栈
        assignPos(v, ++pos);
        assignParent(u, v);
      }
      head[u] = edges[e].next;  //记得把head指向下一条边方便下次读取
    } else {
      maxr[u] = pos;  //如果读完了，说明点u的子树，就是这个区间了.
      --toper;  //读完了就弹出一个点
    }
  }
}
```

