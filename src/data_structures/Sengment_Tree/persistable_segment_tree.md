# 持久化線段樹

我們先來看一下這道題目：

> [CSES 1737 Range Queries and Copies](https://cses.fi/problemset/task/1737/)
>
> 題目大意是維護一個集合，集合裡面有很多 \\(array\\)（初始為一個），你要維護以下三種操作：
>
> 1. 把第 \\(k\\) 個版本的 \\(array\\) 的位置 \\(a\\) 的值改成 \\(x\\)
> 2. 求出第 \\(k\\) 個版本的 \\(array\\) 中位置 \\(a\\) 到 \\(b\\) 的區間和
> 3. 複製第 \\(k\\) 個版本的 \\(array\\)

## 什麼是持久化線段樹

持久化就是保存歷史版本，我們每次在線段樹上修改某個東西就會新建一個版本，這樣如果我們要知道修改之前資料的樣子是什麼，就可以跳回那個版本。

對線段樹上進行持久化，最簡單的想法就是每次修改的時候，直接複製一整顆線段樹，但這樣的話太浪費記憶體空間了。

仔細回想一下在線段樹的修改（以單點修改爲例），會發現其實每次有改到的點只有根到葉子的那一條鏈，而其他節點的值都不變。

（修改第 13 個點的值）

<img src= "image\persistable_segment_tree\persistable_segment_tree-1.png" />

所以每次遞迴修改的時候，遇到需要修改的子節點就新建一個指標給它，另一個未修改到的子節點就連回上一個版本的指標位置，其餘的部分都跟普通的線段樹一樣。

<img src= "image\persistable_segment_tree\persistable_segment_tree-2.png" />

### 空間複雜度

如果序列長度為 \\(N\\)，那每次單點修改需要增加 \\( log{ N }\\) 個點，如果有 \\(Q\\) 次修改操作，需要增加的節點數就是 \\(Q \times log{N}\\)，加上原本線段樹的節點 \\(2N - 1\\)，持久化線段樹的總空間會到 \\(2N -1 + Qlog{N}\\)。

### 時間複雜度

持久化線段樹的操作的時間複雜度與普通的線段樹是一樣的，以這題爲例，建樹的時間複雜度是 \\(O(N)\\)，單點修改（即題目的操作 \\(1\\)）的時間複雜度是 \\(O(log{N})\\)，區間查詢（即題目的操作 \\(2\\)）的時間為 \\(O(log{N})\\)，總時間複雜度為 \\(O(N) + O(Qlog{N})\\)。

<details><summary> Solution Code </summary>

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;

const int z = 2e5 + 1;
int n, q;
struct persistent_segment_tree {
  int lc[z << 5], rc[z << 5], tot = 0, rt[z];
  ll sum[z << 5];
  int node() {
    ++tot;
    lc[tot] = rc[tot] = sum[tot] = 0;
    return tot;
  }
  void build(int l, int r, int &o) {
    if (!o)
      o = node();
    if (l == r)
      return cin >> sum[o], void();
    int mid = l + r >> 1;
    build(l, mid, lc[o]);
    build(mid + 1, r, rc[o]);
    sum[o] = sum[lc[o]] + sum[rc[o]];
  }
  ll qry(int l, int r, int ql, int qr, int o) {
    if (ql <= l && r <= qr)
      return sum[o];
    ll mid = l + r >> 1, res = 0;
    if (ql <= mid)
      res = qry(l, mid, ql, qr, lc[o]);
    if (qr > mid)
      res += qry(mid + 1, r, ql, qr, rc[o]);
    return res;
  }
  int mdf(int l, int r, int x, int o, int v) {
    int p = node();
    lc[p] = lc[o], rc[p] = rc[o];
    if (l == r)
      return sum[p] = v, p;
    int mid = l + r >> 1;
    if (x <= mid)
      lc[p] = mdf(l, mid, x, lc[p], v);
    else
      rc[p] = mdf(mid + 1, r, x, rc[p], v);
    sum[p] = sum[lc[p]] + sum[rc[p]];
    return p;
  }
} seg;
signed main() {
  ios::sync_with_stdio(0), cin.tie(0), cout.tie(0);
  cin >> n >> q;
  seg.build(1, n, seg.rt[1]);
  for (int op, k, x, y, i = 1; q--;) {
    cin >> op;
    if (op == 3)
      cin >> k, seg.rt[++i] = seg.rt[k];
    else if (op == 1) {
      cin >> k >> x >> y;
      seg.rt[k] = seg.mdf(1, n, x, seg.rt[k], y);
    } else {
      cin >> k >> x >> y;
      cout << seg.qry(1, n, x, y, seg.rt[k]) << "\n";
    }
  }
  return 0;
}
```

</details>

## 經典應用 - 區間第 k 小

> [洛谷 P3834【模板】可持久化线段树 2](https://www.luogu.com.cn/problem/P3834)
>
> 給定長度為 \\(n\\) 整數序列 \\(a\\)，接著有 \\(q\\) 筆詢問，對於詢問的閉區間 \\([l, r]\\) 求出區間內的第 \\(k\\) 小值
>
> \\(n , q \le 2 \times 10^5 , |a_i| \le 10^9\\)

### 如何查詢第 \\(k\\) 小

在求區間第 \\(k\\) 小之前，先來看一下要如何求出全部數字（也就是詢問 \\([1, n]\\)）的第 \\(k\\) 小，我們考慮對線段樹做一點修改，原本的線段樹存的是一個序列的值，我們把線段樹的下標改成存某個數字出現的次數（即下標為 \\(x\\) 的葉節點存的是數字 \\(x\\) 的出現次數），而其他節點則是存左右子樹的和。

<img src = "image\persistable_segment_tree\persistable_segment_tree-3.png" />

由於每個節點是存的是區間和，要求第 \\(k\\) 小，我們從根節點開始，每次判斷左子樹的和是否**大於等於** \\(k\\)，如果是的話表示有至少 \\(k\\) 個數字在左子樹，所以往左子樹找。如果左子樹的和**小於** \\(k\\)，表示答案在右子樹，先把 \\(k\\) 減掉左子樹的和，再往右子樹找答案，直到找到葉節點，所對應的下標就是答案。

```cpp
int query_kth(int l, int r, int rt, int k){
    if(l == r) return l;
    int sum = seg[rt << 1], mid = l + r >> 1;
    if(sum >= k) return query_kth(l, mid, rt << 1, k);
    return query_kth(mid + 1, r, rt << 1 | 1, k - sum);
}
```

## 回到題目

我們由左往右掃過整個序列，每個數字都新建一個版本並插入持久化線段樹，這樣第 \\(r\\) 個版本就有了 \\([1, r]\\) 的線段樹的樣子

那對於區間詢問 \\([l,r]\\)，我們可以利用前綴和的特性，用第 \\(r\\) 個版本的線段樹減掉第 \\(l-1\\) 個版本就可以求出 \\([l,r]\\) 這個區間的線段樹的樣子，接下來就用上面提到的方法在線段樹上找第 \\(k\\) 小即可。

時間複雜度：\\(O(Nlog{N}) + O(Qlog{N})\\)

空間複雜度：\\( O(Nlog{N}) \\)

<details><summary> Solution Code </summary>

```cpp
#include <bits/stdc++.h>
using namespace std;

const int maxn = 200001;
int n, m, a[maxn], b[maxn];
struct persistent_segment_tree {
  int ls[maxn << 5], rs[maxn << 5], sum[maxn << 5], root[maxn] = {}, cnt;
  int node() {
    ++cnt;
    ls[cnt] = rs[cnt] = sum[cnt] = 0;
    return cnt;
  }
  void build(int l, int r, int &o) {
    o = node();
    int mid = l + r >> 1;
    if (l == r)
      return;
    build(l, mid, ls[o]);
    build(mid + 1, r, rs[o]);
  }
  void add(int l, int r, int &o, int lst, int x) {
    o = node();
    ls[o] = ls[lst];
    rs[o] = rs[lst];
    sum[o] = sum[lst] + 1;
    int mid = l + r >> 1;
    if (l == r)
      return;
    if (x <= mid)
      add(l, mid, ls[o], ls[lst], x);
    else
      add(mid + 1, r, rs[o], rs[lst], x);
  }
  int query(int L, int R, int l, int r, int k) {
    if (l == r)
      return l;
    int x = sum[ls[R]] - sum[ls[L]], mid = l + r >> 1;
    if (x >= k)
      return query(ls[L], ls[R], l, mid, k);
    else
      return query(rs[L], rs[R], mid + 1, r, k - x);
  }
} seg;
int main() {
  ios::sync_with_stdio(0), cin.tie(0), cout.tie(0);
  cin >> n >> m;
  for (int i = 1; i <= n; ++i)
    cin >> a[i], b[i] = a[i];
  sort(b + 1, b + 1 + n);
  int len = unique(b + 1, b + 1 + n) - b - 1;
  seg.build(1, len, seg.root[0]);
  for (int i = 1; i <= n; ++i)
    seg.add(1, len, seg.root[i], seg.root[i - 1],
            lower_bound(b + 1, b + 1 + len, a[i]) - b);
  for (int l, r, k; m--;) {
    cin >> l >> r >> k;
    cout << b[seg.query(seg.root[l - 1], seg.root[r], 1, len, k)] << "\n";
  }
  return 0;
}
```

</details>

## 其他練習題

>[TIOJ 1827 Yet another simple task \^____\^](https://tioj.ck.tp.edu.tw/problems/1827)
>
>給定一個長度為 \\(N\\) 的整數序列 \\(B\\)。有 \\(M\\) 筆詢問，對於每組詢問 \\((P, K)\\)，求最小的答案 \\(S\\) 滿足至少有 \\(K\\) 個元素滿足以下條件
>
>- \\(|P - i| \leq S\\)
>- \\(B_i \leq S\\)
>
>\\(1 \leq N, M \leq 10^5 \\)
>\\(1 \leq K, B_i \leq N \\)
>\\(0 \leq P \lt N\\)

### 想法

考慮對答案二分搜，我們二分搜 \\(S\\)，至於二分搜的判斷就是要求出區間 \\([P - S, P +S]\\) 中小於等於 \\(S\\) 的元素有多少個，這個問題可以用持久化線段樹解決。

時間複雜度：\\( O(Nlog{N} + Mlog^2{N}) \\)

<details><summary> Solution Code </summary>

```cpp
#include <bits/stdc++.h>
using namespace std;
#define F first
#define S second
typedef long long ll;

int n, m;
const int z = 1e5 + 5;
struct persistent_segment_tree {
  int rt[z] = {}, cnt = 0, lc[z * 70], rc[z * 70], sum[z * 70];
  int node() {
    ++cnt;
    lc[cnt] = rc[cnt] = sum[cnt] = 0;
    return cnt;
  }
  void build(int l, int r, int &o) {
    if (!o)
      o = node();
    if (l == r)
      return;
    int mid = l + r >> 1;
    build(l, mid, lc[o]), build(mid + 1, r, rc[o]);
  }
  void mdf(int prv, int &o, int l, int r, int x) {
    o = node();
    lc[o] = lc[prv], rc[o] = rc[prv], sum[o] = sum[prv] + 1;
    if (l == r)
      return;
    int mid = l + r >> 1;
    if (x <= mid)
      mdf(lc[prv], lc[o], l, mid, x);
    else
      mdf(rc[prv], rc[o], mid + 1, r, x);
  }
  int qry(int l, int r, int o, int ql, int qr) {
    if (ql <= l && r <= qr)
      return sum[o];
    int mid = l + r >> 1, s = 0;
    if (ql <= mid)
      s = qry(l, mid, lc[o], ql, qr);
    if (qr > mid)
      s += qry(mid + 1, r, rc[o], ql, qr);
    return s;
  }
  int get_(int l, int r, int k) {
    int res = qry(1, n, rt[r], 1, k) - qry(1, n, rt[l - 1], 1, k);
    return res;
  }
} seg;
signed main() {
  ios::sync_with_stdio(0), cin.tie(0), cout.tie(0);
  cin >> n >> m;
  seg.build(1, n, seg.rt[0]);
  for (int x, i = 1; i <= n; ++i) {
    cin >> x;
    seg.mdf(seg.rt[i - 1], seg.rt[i], 1, n, x);
  }
  for (int mid, l, r, p, k, o; m; --m) {
    cin >> p >> k;
    ++p;
    l = 1, r = n;
    while (l < r) {
      mid = l + r >> 1;
      if (seg.get_(max(1, p - mid), min(n, p + mid), mid) < k)
        l = mid + 1;
      else
        r = mid;
    }
    cout << l << "\n";
  }
  return 0;
}
```

</details>

>[洛谷 P2617 Dynamic Rankings (動態區間第 k 小)](https://www.luogu.com.cn/problem/P2617)
>
>給定一個含有 \\(n\\) 個數的序列 \\(a_1,a_2 \dots a_n\\)，需要支援 \\(m\\) 個操作，操作有两種：  
>
>- `Q l r k` 表示查詢下標在區間 \\([l,r]\\) 中的第 \\(k\\) 小的數  
>- `C x y` 表示將 \\(a_x\\) 改為 \\(y\\)
>
>\\(1\le n,m \le 10^5\\)，\\(1 \le l \le r \le n\\)，\\(1 \le k \le r-l+1\\)，\\(1\le x \le n\\)，\\(0 \le a_i,y \le 10^9\\)。

### 想法

這題其實也可以用整體二分來解，但本章節不討論此解法。

仔細思考發現靜態的區間第 \\(k\\) 小利用的是持久化線段樹前綴和的特性，而這題的差別多了一個修改操作，所以我們需要一個支援單點修改和區間求和的資料結構：**樹狀數組**，我們可以考慮樹狀數組套權值線段樹，樹狀數組的每個節點都代表一個線段樹的根節點。

在修改的時候，樹狀數組的單點修改複雜度為 \\(O(log{N})\\)，每個線段樹的修改也要 \\(O(log{N})\\)，所以一次修改所要花費的時間複雜度為 \\(O(log^2{N})\\)。

對於區間查詢 \\([l, r]\\)，我們直接從樹狀數組取出相對應的 \\(log{N}\\) 個節點 ( \\(l-1\\) 和 \\( r \\) 都各有 \\(log{N}\\)) 個，之後遞迴詢問就跟一般的區間第 \\(k\\) 小一樣，只是這次要對 \\(log{N}\\) 個線段樹相減。

時間複雜度: \\(O(Mlog^2{N})\\)

空間的部分，因為我們是採用動態開點來實作，所需的空間複雜度跟修改是同一階的，所以是 \\(O(Mlog^2{N})\\)

<details><summary> Solution Code </summary>

```cpp
#include <bits/stdc++.h>
using namespace std;
#define lb(x) x & -x

const int maxn = 100001;
int n, m, a[maxn], b[maxn << 1], len;
struct query {
  int op, x, y, z;
} q[maxn];
struct persistent_segment_tree {
  int ls[maxn * 400], rs[maxn * 400], sum[maxn * 400], root[maxn], cnt;
  vector<int> ql, qr;
  void mdf(int l, int r, int x, int &o, int val) {
    if (!o)
      o = ++cnt;
    sum[o] += val;
    if (l == r)
      return;
    int mid = l + r >> 1;
    if (x <= mid)
      mdf(l, mid, x, ls[o], val);
    else
      mdf(mid + 1, r, x, rs[o], val);
  }
  void mdf_all(int x, int val) {
    int idx = lower_bound(b + 1, b + 1 + len, a[x]) - b;
    for (; x <= n; x += lb(x))
      mdf(1, len, idx, root[x], val);
  }
  int qry(int l, int r, int k) {
    if (l == r)
      return l;
    int mid = l + r >> 1, S = 0;
    for (auto &i : ql)
      S -= sum[ls[i]];
    for (auto &i : qr)
      S += sum[ls[i]];
    if (k <= S) {
      for (int i = 0, j = ql.size(); i < j; ++i)
        ql[i] = ls[ql[i]];
      for (int i = 0, j = qr.size(); i < j; ++i)
        qr[i] = ls[qr[i]];
      return qry(l, mid, k);
    } else {
      for (int i = 0, j = ql.size(); i < j; ++i)
        ql[i] = rs[ql[i]];
      for (int i = 0, j = qr.size(); i < j; ++i)
        qr[i] = rs[qr[i]];
      return qry(mid + 1, r, k - S);
    }
  }
  int qry_all(int l, int r, int k) {
    ql.clear(), qr.clear();
    for (; r; r -= lb(r))
      qr.emplace_back(root[r]);
    for (--l; l; l -= lb(l))
      ql.emplace_back(root[l]);
    return qry(1, len, k);
  }
} seg;
int main() {
  ios::sync_with_stdio(0), cin.tie(0), cout.tie(0);
  cin >> n >> m;
  for (int i = 1; i <= n; ++i)
    cin >> a[i], b[++len] = a[i];
  for (int i = 1; i <= m; ++i) {
    char c;
    cin >> c;
    q[i].op = c == 'Q';
    if (c == 'Q')
      cin >> q[i].x >> q[i].y >> q[i].z;
    else
      cin >> q[i].x >> q[i].y, b[++len] = q[i].y;
  }
  sort(b + 1, b + 1 + len);
  len = unique(b + 1, b + 1 + len) - b - 1;
  for (int i = 1; i <= n; ++i)
    seg.mdf_all(i, 1);
  for (int i = 1; i <= m; ++i) {
    if (q[i].op)
      cout << b[seg.qry_all(q[i].x, q[i].y, q[i].z)] << "\n";
    else {
      seg.mdf_all(q[i].x, -1);
      a[q[i].x] = q[i].y;
      seg.mdf_all(q[i].x, 1);
    }
  }
  return 0;
}
```

</details>

>[洛谷 P2633 Count on a tree](https://www.luogu.com.cn/problem/P2633)
>
>給定一棵 \\(n\\) 個節點的樹，每個點有一個權值。有 \\(m\\) 個詢問，每次給你 \\(u,v,k\\)，你需要回答 \\(u \text{ xor last}\\) 和 \\(v\\) 這兩個節點間第 \\(k\\) 小的點權。  
>
>其中 \\(\text{last}\\) 是上一個詢問的答案，定義其初始為 \\(0\\)。
>
>\\(1\le n,m \le 10^5\\)，點權在 \\([1,2^{31}-1]\\) 之間。

### 想法

轉換一下題目會發現這是區間第 \\(k\\) 小的樹上版本：每次詢問樹上某兩點的路徑上第 \\(k\\) 小的值。

考慮持久化線段樹具有前綴和的性質，我們可以利用樹上差分來解這題，我們定義 \\(p[u]\\) 代表從根節點到點 \\(u\\) 的持久化線段樹版本，那在點 \\(u\\) 到點 \\(v\\) 路徑之間的 "權值線段樹" 的樣子就是

\\(p[u] + p[v] - p[LCA(u,v)] - p[fa[LCA(u, v)]]\\)

只要用上面的公式算出相對應的權值線段樹，接下來的詢問就跟一般的一樣。

時間複雜度：\\( O(Qlog{N} + Nlog{N}) \\)

<details><summary> Solution Code </summary>

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;

const int z = 1e5 + 5, Z = z * 120;
struct edge {
  int to, nxt;
} e[z << 1];
int tot = 0, cnt = 0, h[z], n, m, dep[z], lca[20][z], a[z], b[z], val[Z], len;
void add(int u, int v) {
  e[++cnt] = edge{v, h[u]};
  h[u] = cnt;
}
void build_lca() {
  for (int j = 1; j <= 17; ++j)
    for (int i = 1; i <= n; ++i)
      lca[j][i] = lca[j - 1][lca[j - 1][i]];
}
int LCA(int a, int b) {
  if (dep[a] < dep[b])
    swap(a, b);
  for (int i = dep[a] - dep[b], k = 0; i; i >>= 1, ++k)
    if (i & 1)
      a = lca[k][a];
  if (a == b)
    return a;
  for (int i = 17; ~i; --i)
    if (lca[i][a] != lca[i][b])
      a = lca[i][a], b = lca[i][b];
  return lca[0][a];
}
struct persistent_segment_tree {
  int rs[Z], ls[Z], p[z];
  void build_tree(int l, int r, int &rt) {
    rt = ++tot;
    val[rt] = 0;
    if (l == r)
      return;
    int mid = l + r >> 1;
    build_tree(l, mid, ls[rt]), build_tree(mid + 1, r, rs[rt]);
  }
  void inse(int l, int r, int x, int &rt, int pre) {
    rt = ++tot;
    rs[rt] = rs[pre], ls[rt] = ls[pre], val[rt] = val[pre] + 1;
    if (l == r)
      return;
    int mid = l + r >> 1;
    if (x <= mid)
      inse(l, mid, x, ls[rt], ls[pre]);
    else
      inse(mid + 1, r, x, rs[rt], rs[pre]);
  }

  int query(int l, int r, int A, int B, int lc, int fa, int k) {
    if (l == r)
      return l;
    int x = val[ls[A]] + val[ls[B]] - val[ls[lc]] - val[ls[fa]],
        mid = l + r >> 1;
    if (k <= x)
      return query(l, mid, ls[A], ls[B], ls[lc], ls[fa], k);
    return query(mid + 1, r, rs[A], rs[B], rs[lc], rs[fa], k - x);
  }
} seg;
void dfs(int x, int pa) {
  seg.inse(1, len, lower_bound(b + 1, b + 1 + len, a[x]) - b, seg.p[x],
           seg.p[pa]);
  for (int i = h[x]; i; i = e[i].nxt) {
    int v = e[i].to;
    if (dep[v])
      continue;
    dep[v] = dep[x] + 1;
    lca[0][v] = x;
    dfs(v, x);
  }
}
int main() {
  ios::sync_with_stdio(0), cin.tie(0), cout.tie(0);
  cin >> n >> m;
  for (int i = 1; i <= n; ++i)
    cin >> a[i], b[i] = a[i];
  sort(b + 1, b + 1 + n);
  len = unique(b + 1, b + 1 + n) - b - 1;
  int x, y, k, last = 0;
  for (int i = 0; i < n - 1; ++i) {
    cin >> x >> y;
    add(x, y);
    add(y, x);
  }
  dep[1] = 1;
  seg.build_tree(1, len, seg.p[0]);
  dfs(1, 0);
  build_lca();
  while (m--) {
    cin >> x >> y >> k;
    x ^= last;
    int lc = LCA(x, y), fa = lca[0][lc];
    last = b[seg.query(1, len, seg.p[x], seg.p[y], seg.p[lc], seg.p[fa], k)];
    cout << last << "\n";
  }
  return 0;
}
```

</details>

>[Codefroces 1171F](https://codeforces.com/problemset/problem/1771/F)
>
>題目大意是給定一個長度為 \\(n\\) 的非負整數序列，接下來有 \\(q\\) 筆詢問，每次詢問一個區間 \\([l, r]\\) 中出現次數為**奇數**的數字中，最小的那個是多少。題目強制在線，輸入的 \\(l, r\\) 需要\\(xor\\) \\(lastans\\) 才是真正要詢問的區間。
>
>\\(1\le n,q \le 10^5\\) , \\(1\le a_i \le 10^9\\) , \\(1\le l,r \le 2 \times 10^9\\)

### 想法

題目要求出現次數為奇數，可以考慮使用 xor hash，對每個數產生一個隨機的權重，如果一個數字出現次數為偶數，則 xor 起來會是 \\(0\\)。所以在持久化線段樹上做詢問的時候只要判斷兩棵左子樹的 xor 值是不是 \\(0\\) 就可以了，如果不是就往左子樹找，否則往右子樹找。

需要注意的點是如果 random seed 使用的是 `srand(time(0))` 的話會發生 hash conflict，改用 `mt19937 random(time(0))` 就可以 AC 了。

時間複雜度：\\( O(Qlog{N} + Nlog{N}) \\)

<details><summary> Solution Code </summary>

```cpp
#include <bits/stdc++.h>
using namespace std;

const int z = 2e5 + 5, inf = 1e9 + 2;
int n, m, cnt, lst;
struct persistent_segment_tree {
  int seg[z << 6], rt[z], lc[z << 6], rc[z << 6];
  void mdf(int l, int r, int x, int v, int &o, int pre) {
    o = ++cnt;
    lc[o] = lc[pre], rc[o] = rc[pre], seg[o] = seg[pre] ^ v;
    if (l == r)
      return;
    int mid = l + (r - l) / 2;
    if (x <= mid)
      mdf(l, mid, x, v, lc[o], lc[pre]);
    else
      mdf(mid + 1, r, x, v, rc[o], rc[pre]);
  }
  int qry(int l, int r, int L, int R) {
    if (l == r)
      return l;
    int mid = l + (r - l) / 2;
    if (seg[lc[R]] ^ seg[lc[L]])
      return qry(l, mid, lc[L], lc[R]);
    return qry(mid + 1, r, rc[L], rc[R]);
  }
} seg;
unordered_map<int, int> mp;
signed main() {
  ios::sync_with_stdio(0), cin.tie(0), cout.tie(0);
  mt19937 rand(time(0));
  cin >> n;
  for (int x, i = 1; i <= n; ++i) {
    cin >> x;
    if (!mp[x])
      mp[x] = rand() % inf + 1;
    seg.mdf(0, inf, x, mp[x], seg.rt[i], seg.rt[i - 1]);
  }
  cin >> m;
  for (int l, r; m--;) {
    cin >> l >> r;
    l ^= lst, r ^= lst;
    lst = seg.qry(0, inf, seg.rt[l - 1], seg.rt[r]);
    if (lst == inf)
      lst = 0;
    cout << lst << "\n";
  }
}
```

</details>

> [Codeforces 813E](https://codeforces.com/problemset/problem/813/E)
>
> 給定一個長度為 \\(n\\) 的正整數序列 \\(a\\) 和一個正整數 \\(k\\)，接下來有 \\(q\\) 筆詢問，每次詢問一個區間 \\([l, r]\\) 最多可以選多少個數，使得同一個數的出現次數不超過 \\(k\\)，本題强制在線。
>
> \\(1 \le n, q, a_i, k \le 10^5\\)

### 想法

考慮對索引值建立持久化線段樹。我們掃過序列，並把數字的索引值插入到持久化線段樹中（假設現在數字的索引值是 \\(i\\)，就把線段樹中的第 \\(i\\) 個位置設為 \\(1\\)，順便記錄序列中數字出現的位置，當數字出現超過 \\(k\\) 次之後，還要把現在線段樹版本下的第上 \\( k \\) 個位置改成 \\(0\\)。這樣對於一組詢問 \\([l, r]\\)，我們只要詢問第 \\(r\\) 個版本的持久化線段樹中，大於等於 \\(l\\) 的索引值有多少是 \\(1\\) 即可。

時間複雜度：\\(O(Nlog{N}+Mlog{N})\\)
  
<details><summary> Solution Code </summary>

```cpp
#include <bits/stdc++.h>
using namespace std;
const int maxn = 1e5 + 5;

int n, m, k;
vector<int> pos[maxn];
struct persistent_segment_tree {
  int ls[maxn * 40], rs[maxn * 40], root[maxn], cnt, sum[maxn * 40];
  void insert(int &o, int pre, int l, int r, int x, int v) {
    o = ++cnt;
    sum[o] = sum[pre] + v, ls[o] = ls[pre], rs[o] = rs[pre];
    if (l == r)
      return;
    int mid = l + r >> 1;
    if (x <= mid)
      insert(ls[o], ls[pre], l, mid, x, v);
    else
      insert(rs[o], rs[pre], mid + 1, r, x, v);
  }
  int query(int o, int l, int r, int x) {
    if (!o)
      return 0;
    if (l == r)
      return sum[o];
    int mid = l + r >> 1;
    if (x <= mid)
      return sum[rs[o]] + query(ls[o], l, mid, x);
    return query(rs[o], mid + 1, r, x);
  }
} seg;
int main() {
  ios::sync_with_stdio(0), cin.tie(0), cout.tie(0);
  cin >> n >> k;
  for (int i = 1, x; i <= n; ++i) {
    cin >> x;
    pos[x].push_back(i);
    seg.insert(seg.root[i], seg.root[i - 1], 1, n, i, 1);
    if ((int)pos[x].size() > k)
      seg.insert(seg.root[i], seg.root[i], 1, n, pos[x][pos[x].size() - k - 1],
                 -1);
  }
  cin >> m;
  for (int l, r, last = 0; m--;) {
    cin >> l >> r;
    l = (l + last) % n + 1;
    r = (r + last) % n + 1;
    if (l > r)
      swap(l, r);
    cout << (last = seg.query(seg.root[r], 1, n, l)) << "\n";
  }
  return 0;
}
```

</details>

## References

[數據結構線段樹 -- 權值線段樹詳解](https://blog.csdn.net/yanweiqi1754989931/article/details/117380913)

[OIWiki 持久化線段樹](https://oi-wiki.org/ds/persistent-seg/)

[USACO Guide - Persistent Data Structures](https://usaco.guide/adv/persistent?lang=cpp)

[Illustrated persistent segment tree tutorial](https://codeforces.com/blog/entry/49777)

[Persistent segment tree ( Problems )](https://codeforces.com/blog/entry/56880)

[Preserving the history of its values (Persistent Segment Tree)](https://cp-algorithms.com/data_structures/segment_tree.html#compression-of-2d-segment-tree)

[洛谷題單 - 持久化線段樹](https://www.luogu.com.cn/problem/list?tag=233)
