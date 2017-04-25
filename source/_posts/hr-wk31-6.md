---
title: Nominating Group Leaders[HR]
date: 2017-04-18 11:50:35
tags: 数据结构
mathjax: true
---

## [题目链接](https://www.hackerrank.com/contests/w31/challenges/nominating-group-leaders)

### 题意

t组测试
n个数g个询问
每次询问l r x，表示区间[l,r]里的数出现x次的数，是哪个，如果有多个输出最小的，没有则输出-1

$1\le t \le 5$
$1\le n, g \le 10^5$
$0 \le l \le r \le n - 1$
$0 \le a_i \le n - 1$
$1 \le x \le n$

题意是不是简单明了？简化了的，原描述非常复杂.
<!-- more -->

### 解法
---
auther的解法是平方分割,思路非常复杂,代码量也不小.
我的解法很简单，是用主席树，询问的时候在树上枚举下去，加个剪枝即可保证时间复杂度.
首先建树的方式和常规主席树一样，对每个数(1~1e5)简历版本.
1,询问的时候树上枚举每个数出现了多少次.这样时间复杂度是n，有些没有出现的数也遍历了。所以第一个剪枝就是去掉没有出现过的数，剪枝的方法就是区间内出现了数的数量和，如果小于x，就跳过这区间就跳过，间接去掉一些不满足条件的.
2,如果数据是1 2 3 4 5 ... n - 1, n - 1.然后询问是0 n 2这样的话。第一个剪枝就不管用了，因为除开叶子节点的所有区间出现的数量和都大于等于2.时间还是n，所以第二个剪枝就是，加个最大最小剪枝。就是在区间保存一个出现过的数最大次数和最小次数，如果x没有在这区间内，就跳过。这样的话，这样的数据就是logn了。
第一个剪枝能不能去掉呢？可以，就是在初始的时候没有出现过的数， 假设出现了n+1次，这样的话，就一定不是满足的.

cdde:
```cpp
#include <bits/stdc++.h>

typedef long long LLONG;
typedef std::pair<int, int> PII;
typedef std::map<int, int> MII;
typedef std::pair<PII, int> PIIAI;
typedef std::vector<int> VI;
typedef std::vector<PII> VPII;

//#define TDEBUG
#ifdef TDEBUG
#define tout(x) std::cerr << #x << " = " << x << std::endl;
#define tdebug(args...) { std::cerr << "(" << #args << ") = > "; dbg, args; std::cerr << std::endl; }
#else
#define tout(x) //skip all
#define tdebug(args...)
#endif
struct Debuger {
  template<typename T>
  Debuger& operator, (const T& v) {
    std::cerr << v << " ";
    return *this;
  }
} dbg;
template<typename T>inline void scaf(T&v) {
  char ch = getchar();
  int sgn = 1;
  for (; '-' != ch && (ch < '0' || ch > '9'); ch = getchar());
  if ('-' == ch) sgn = -1, v = 0;
  else v = ch - '0';
  for (ch = getchar(); ch >= '0' && ch <= '9'; ch = getchar()) v = (v << 1) + (v << 3) + ch - '0';
  v *= sgn;
}

const int N = 2e5 + 10;
const double eps = 1e-8;

struct QueryParam {
  int l, k, idx;
  QueryParam() { }
  QueryParam(int ll, int kk, int idxx) {
    l = ll;
    k = kk;
    idx = idxx;
  }
};
std::vector<QueryParam> qs[N];

struct SegTree {
  int ls, rs, votes;
  int mx, mn;
  void assign(int lss, int rss, int votess) {
    ls = lss;
    rs = rss;
    votes = votess;
  }
  void setMx(int mxx, int mnn) {
    tdebug(mxx, mnn);
    mx = mxx;
    mn = mnn;
  }
  static int stTotoal;
} tree[N * 10];
int troot[N];
int SegTree::stTotoal = 0;

inline void init(int n) {
  SegTree::stTotoal = 0;
  for (int i = 0; i <= n; ++i) {
    qs[i].clear();
  }
}

inline int build(int l, int r) {
  int v = SegTree::stTotoal++;
  tree[v].votes = 0;
  tree[v].setMx(0, N);
  if (l < r) {
    int mid = l + r >> 1;
    tree[v].ls = build(l, mid);
    tree[v].rs = build(mid + 1, r);
  }
  return v;
}

inline int modify(int l, int r, int x, int last) {
  int v = SegTree::stTotoal++;
  int curVotes = tree[last].votes + 1;
  tree[v].assign(tree[last].ls, tree[last].rs, curVotes);
  if (l < r) {
    int mid = l + r >> 1;
    if (mid >= x) {
      tree[v].ls = modify(l, mid, x, tree[last].ls);
    } else {
      tree[v].rs = modify(mid + 1, r, x, tree[last].rs);
    }
    int ls = tree[v].ls, rs = tree[v].rs;
    int mxx = std::max(tree[ls].mx, tree[rs].mx);
    int mnn = std::min(tree[ls].mn, tree[rs].mn);
    tree[v].setMx(std::max(tree[last].mx, mxx), std::min(tree[last].mn, mnn));
  } else {
    tree[v].setMx(std::max(tree[last].mx, curVotes), std::min(tree[last].mn, curVotes));
  }
  return v;
}

inline int query(int l, int r, int tl, int tr, int k) {
  int cv = tree[tr].votes - tree[tl].votes;
  tdebug(tree[tr].mx, tree[tr].mn);
  if (tree[tr].mx < k || tree[tr].mn > k) { return 0; }
  if (cv < k) { return 0; }
  if (l >= r) {
    if (cv == k) { return l; }
    return 0;
  } else {
    int mid = l + r >> 1;
    int rlt = query(l, mid, tree[tl].ls, tree[tr].ls, k);
    if (0 == rlt) {
      rlt = query(mid + 1, r, tree[tl].rs, tree[tr].rs, k);
    }
    return rlt;
  }
  return 0;
}

int main() {
  //freopen("data.in", "r", stdin);
  //freopen("data.out", "w", stdout);
  int t;
  for (scaf(t); t--;) {
    int n, g;
    scaf(n);
    init(n);
    troot[0] = build(1, n);
    for (int i = 1; i <= n; ++i) {
      scaf(g);
      troot[i] = modify(1, n, g + 1, troot[i - 1]);
    }
    scaf(g);
    int l, r, k;
    for (int i = 0; i < g; ++i) {
      scaf(l); scaf(r); scaf(k);
      printf("%d\n", query(1, n, troot[l], troot[r + 1], k) - 1);
    }
  }
  return 0;
}

```