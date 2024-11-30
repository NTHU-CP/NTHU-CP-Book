# CDQ 分治

## 介紹

CDQ 分治最早是由中國的選手陳丹奇所提出。在一般的分治中，我們將數列分割成左右區間去分治再合併答案，這時左右區間之間是各自不影響對方的答案的。而某些時候左區間會影響到右區間的答案，這時就會用到 CDQ 分治這個技巧。

在 CDQ 常見的應用中，大致上可以分成以下幾類：

+ 偏序問題
+ 動態問題轉為靜態問題 (操作分治)
+ 斜率優化

## 偏序問題

這類的問題通常的形式為 : 給定一些在 \\( d \\) 維空間下的 \\( n \\) 個點 \\( p_1, p_2 \dots p_n \\)，並且定義偏序關係 \\( \prec \\)，求數列中符合 \\( p_i \prec p_j \\) 的點對 \\( (i, j) \\) 的數量。

什麼是偏序關係呢 ? 偏序關係為一種二元關係 \\( \preceq \\) 符合：

+ 自反律：\\( a \preceq a \\)
+ 反對稱律：\\( a \preceq b, b \preceq a \implies a = b \\)
+ 遞移律：\\( a \preceq b, b \preceq c \implies a \preceq c \\)

舉個例子，在一張 DAG 中「節點 \\( u \\) 能走到節點 \\( v \\)」，以及在正整數中「\\( a \\) 能整除 \\( b \\)」皆為一種偏序關係。

而在 CDQ 分治中，對於給定的 \\( d \\) 維點集 \\( p_1, p_2 \dots p_n \\) 最常遇到的偏序關係即為：

\begin{align}
& p_i \prec p_j \equiv (p_{i, 1} \lesseqgtr p_{j, 1}) \land (p_{i, 2} \lesseqgtr p_{j, 2}) \ \dots \land \ (p_{i, d} \lesseqgtr p_{j, d})
\end{align}

其中一個最經典的例子就是逆序數對。將陣列中的 index 當作第一維，value 當作第二維，並且定義偏序關係為 :

\begin{align}
& p_i \prec p_j \equiv (p_{i, 1} < p_{j, 1}) \land (p_{i, 2} > p_{j, 2})
\end{align}

這樣一來就變成求 \\( p_1, p_2, \dots p_n \\) 中符合偏序關係的點對數量了。

### 二維偏序

就像前面說的，逆序數對就是其中一個二維偏序的例子，因為太過經典所以這邊就不做過多的解釋了，詳細的部分可以參考 [這份講義](https://drive.google.com/file/d/1SxuBzDDbgmpo2j2PyXEQ2Oe8BgVqM_DH/view)。解法就是在 merge sort 時去順便計算點對數量，並且在合併時利用雙指標去計算兩邊所貢獻的逆敘數對數量。

### 三維偏序

> [Atcoder abc309 F](https://atcoder.jp/contests/abc309/tasks/abc309_f)
>
> 有 \\( N \\) 個箱子，其長、寬、高分別為 \\( (d_i, w_i, h_i) \\)，詢問是否存在兩個箱子，其中一個箱子的能夠在翻轉幾次後 (也就是交換 \\( w_i, d_i, h_i \\) ) 長寬高皆嚴格大於另一個的。
>
> + \\( N \leq 2 \times 10^5 \\)
> + \\( 1 \leq h_i, d_i, w_i \leq 10^9 \\)

令一個箱子長寬高最小的、次小的以及最大的為 \\( a_i, b_i, c_i \\)，則可以發現題目的條件等價於存在一組數對 \\( (i, j) \\) 符合 \\( a_i < a_j, b_i < b_j, c_i < c_j \\)。我們可以將題目從詢問是否存在轉為計算數對數量，再去檢查答案是否大於 0。

考慮計算 \\( [L, R] \\) 中符合條件的點對數量。先將數列依照 \\( a_i \\) 排序，並從中點 \\( M = \frac{L + R}{2} \\) 去將區間切割成 \\( [L, M], [M + 1, R] \\) 分治處理，接下來的問題是要如何計算 \\( i \in [L, M], j \in [M + 1, R] \\) 的點對數量。

由於已經先依照 \\( a_i \\) 排序了，所以對於每個 \\( i \in [L, M], j \in [M + 1, R],\ a_i < a_j \\) (先假設所有的 \\( a_i \\) 皆不同)，於是只要考慮符合 \\( b_i < b_j, c_i < c_j \\) 就好了。

這樣一來可以先把左右區間照 \\( b_i \\) 排序，對於每個 \\( j \\)，我們可以用雙指標將所有符合 \\( b_i < b_j \\) 的點 \\( i \\)，在資料結構上 \\( c_i \\) 的位置加上 1，那麼 \\( j \\) 可以形成的點對數量就會是資料結構上 \\( [1, c_j - 1] \\) 的區間和。

現在將假設拿掉，也就是可能會有 \\( a_i = a_j \\) 的情況。如果依照上面的方式會把 \\( a_i = a_j, b_i < b_j, c_i < c_j \\) 的點對一起算進去，為了避免這個情形，在最一開始排序時若 \\( a_i = a_j \\)，則依照 \\( b \\) 由大到小排。這麼做就可以避免枚舉到 \\( a \\) 相同的點對了。

要注意的是，在遞迴結束後要將資料結構清空時，不能將資料結構中每個點直接歸零，要依照插入紀錄 undo 回去。另外由於 \\( a_i, b_i, c_i \\) 的範圍較大，所以需要做離散化。

最後，整體時間複雜度為 \\( T(n) = 2 \times T(\frac{n}{2}) + O(n\log n) = O(n\log^2 n) \\)。

```cpp
#include <bits/stdc++.h>
#define ll long long
using namespace std;

const int N = 2e5 + 10;

struct item {
    int a, b, c, id;
    item() {}
    item(int _h, int _w, int _d, int _id) : id(_id) {
        vector<int> x = {_h, _w, _d};
        sort(x.begin(), x.end());
        a = x[0];
        b = x[1];
        c = x[2];
    }
};

struct BIT {
    int b[3 * N];
    vector<pair<int, int>> history;

    void upd(int p, int v, bool to_his = true) {
        if (to_his)
            history.push_back(make_pair(p, v));
        while (p < 3 * N) {
            b[p] += v;
            p += p & -p;
        }
    }

    int query(int p) {
        int sum = 0;
        while (p) {
            sum += b[p];
            p -= p & -p;
        }
        return sum;
    }

    void undo() {
        while (!history.empty()) {
            auto [x, y] = history.back();
            upd(x, -y, false);
            history.pop_back();
        }
    }
} B;

int n;
ll ans;

void CDQ(vector<item> &a) {
    if (a.size() <= 1)
        return;
    int mid = (a.size() - 1) >> 1;
    vector<item> al(mid + 1), ar(a.size() - mid - 1);
    for (int i = 0; i < a.size(); i++) {
        if (i <= mid)
            al[i] = a[i];
        else
            ar[i - mid - 1] = a[i];
    }
    CDQ(al);
    CDQ(ar);
    int p = 0;
    for (auto i : ar) {
        while (p <= mid && al[p].b < i.b)
            B.upd(al[p++].c, 1);
        ans += (ll)B.query(i.c - 1);
    }
    sort(a.begin(), a.end(), [](item x, item y) { return x.b < y.b; });
    B.undo();
}

void solve() {
    cin >> n;
    vector<item> a(n);
    vector<int> v;
    for (int i = 0; i < n; i++) {
        int h, w, d;
        cin >> h >> w >> d;
        a[i] = item(h, w, d, i);
        v.push_back(h);
        v.push_back(w);
        v.push_back(d);
    }
    sort(v.begin(), v.end());
    v.resize(unique(v.begin(), v.end()) - v.begin());
    for (auto &i : a) {
        i.a = lower_bound(v.begin(), v.end(), i.a) - v.begin() + 1;
        i.b = lower_bound(v.begin(), v.end(), i.b) - v.begin() + 1;
        i.c = lower_bound(v.begin(), v.end(), i.c) - v.begin() + 1;
    }
    sort(a.begin(), a.end(), [](item x, item y) {
        if (x.a == y.a)
            return x.b > y.b;
        return x.a < y.a;
    });
    CDQ(a);
    cout << (ans > 0 ? "Yes\n" : "No\n");
}

int main() {
    ios::sync_with_stdio(0);
    cin.tie(0);
    solve();
}
```

如果不想用到資結的話，其實也可以再套一層 CDQ 把它取代掉。

在做合併的步驟時，先將 \\( b_i \\) 做排序，然後將所有點打上標記去紀錄他是在左區間或右區間，之後再做第二層 CDQ。會發現由於在第二層 CDQ 中，對於所有在左區間、被標記原本在左區間的點 \\( i \\) 以及在右區間、被標記為原本在右區間的點 \\( j \\)，一定會符合 \\( a_i \leq a_j, b_i \leq b_j \\)，因此只要考慮 \\( c_i < c_j \\) 這個條件就好了，我們可以在依照 \\( c \\) merge sort 時順便計算。

在進到第一層 CDQ 前，我們依照 \\( a \\) 的排序避免了計算到 \\( a_i = a_j, b_i < b_j, c_i < c_j \\) 的點對。所以同理，為了避免在第二層 CDQ 中算到 \\( a_i = a_j, b_i = b_j, c_i < c_j \\) 的點對，在第一層 CDQ 合併步驟的排序時若 \\( b_i = b_j \\)，則依照 \\( c \\) 由大排到小。

時間複雜度一樣為 \\( O(n\log ^2 n) \\)。

```cpp
#include <bits/stdc++.h>
#define ll long long
using namespace std;

const int N = 2e5 + 10;

struct item {
    int a, b, c, id;
    bool is_right;
    item() {}
    item(int _h, int _w, int _d, int _id) : id(_id) {
        vector<int> x = {_h, _w, _d};
        sort(x.begin(), x.end());
        a = x[0];
        b = x[1];
        c = x[2];
    }
};

int n;
ll ans;

void CDQ2(vector<item> &a) {
    if (a.size() <= 1)
        return;
    int mid = (a.size() - 1) >> 1;
    vector<item> al(mid + 1), ar(a.size() - mid - 1);
    for (int i = 0; i < a.size(); i++) {
        if (i <= mid)
            al[i] = a[i];
        else
            ar[i - mid - 1] = a[i];
    }
    CDQ2(al);
    CDQ2(ar);
    int p = 0, idx = 0, cnt = 0;
    for (auto i : ar) {
        while (p <= mid && al[p].c < i.c) {
            if (!al[p].is_right)
                cnt++;
            a[idx++] = al[p++];
        }
        if (i.is_right)
            ans += cnt;
        a[idx++] = i;
    }
    while (p <= mid)
        a[idx++] = al[p++];
}

void CDQ(vector<item> &a) {
    if (a.size() <= 1)
        return;
    int mid = (a.size() - 1) >> 1;
    vector<item> al(mid + 1), ar(a.size() - mid - 1);
    for (int i = 0; i < a.size(); i++) {
        if (i <= mid)
            al[i] = a[i], a[i].is_right = 0;
        else
            ar[i - mid - 1] = a[i], a[i].is_right = 1;
    }
    CDQ(al);
    CDQ(ar);
    sort(a.begin(), a.end(), [](item x, item y) {
        if (x.b == y.b)
            return x.c > y.c;
        return x.b < y.b;
    });
    CDQ2(a);
}

void solve() {
    cin >> n;
    vector<item> a(n);
    for (int i = 0; i < n; i++) {
        int h, w, d;
        cin >> h >> w >> d;
        a[i] = item(h, w, d, i);
    }
    sort(a.begin(), a.end(), [](item x, item y) {
        if (x.a == y.a)
            return x.b > y.b;
        return x.a < y.a;
    });
    CDQ(a);
    cout << (ans > 0 ? "Yes\n" : "No\n");
}

int main() {
    ios::sync_with_stdio(0);
    cin.tie(0);
    solve();
}
```

### 四維偏序

就像從二維擴展到三維時，多拿一個資結去解決多出來的那個維度，我們也可以用 CDQ 套 CDQ 做三維偏序的解法再加資結去完成四維的偏序問題。

或者把那個資結替換成一層 CDQ 分治也是沒問題的，此時就需要兩個標記去分別紀錄某個點在第一層及第二層的 CDQ 中是在哪個區間，第三層也是跟 merge sort 一樣，然後限制住只有兩個標記都在左邊的點才能對兩個標記都在右邊的點做貢獻。

最後其實也可以只用一層 CDQ 分治，這樣一來我們就會需要用到二維資結去維護第三維及第四維。

### D 維偏序

了解了前面從三維擴展到四維的方法後，大概就可以發現：

+ 透過排序可以解決其中一維
+ 套 \\( d \\) 次 CDQ 分治就能夠解決 \\( d \\) 維
+ 用 \\( d \\) 維資料結構也一樣能夠解決 \\( d \\) 維
  
因此對於 \\( D \\) 維偏序問題，我們可以用 \\( x \\) 層 CDQ 以及 \\( D - 1 - x \\) 維資結，就可以在 \\( O(n\log^{D - 1}n) \\) 的時間解決了。

## 操作分治

操作分治是指對動態插入查詢的題目做 CDQ 分治，以達到把動態轉為靜態的效果，來降低資料結構的實作難度，讓我們來看一些例題。

> [Library Checker - Point Add Rectangle Sum](https://judge.yosupo.jp/problem/point_add_rectangle_sum)
>
> 在一個二維平面上，有 \\( n \\) 個點以及 \\( q \\) 筆操作 (\\( n, q \leq 10^5 \\))，每個點的座標以及權重分別為 \\( x_i, y_i, w_i \\) (\\( x_i, y_i, w_i \leq 10^9 \\))。
>
> 接下來的每筆操作為以下其中一種：
>
> + 在某個點 \\( (x, y) \\) 上加值 \\( w \\) (\\( x, y, w \leq 10^9 \\))
> + 詢問在以 \\( (l, d) \\) 做為左下角並以 \\( (r, u) \\) 做為右上角的矩形內的值總和 (\\( l, d, r, u \leq 10^9 \\))

先來看靜態的版本，也就是給一個二維平面，某些點上有值，詢問一個矩形內的值總和。

我們令 \\( f(x_0, y_0) \\) 為點集 \\( \\{ (x, y_0) \mid 0 \leq x < x_0 \\} \\) 的值總和，可以發現對於每筆詢問，其答案就是 :
\begin{align}
& \sum_{y = d}^{u} f(r, y) - f(l - 1, y)
\end{align}

因此可以將每個詢問的左右界以及每個有值的點的 x 座標放在一起由小跑到大並同時維護 \\( f(x, y) \\)。當我們遇到詢問的左界時，就將該筆詢問的答案扣掉 \\( \sum_{y = d}^{u} f(l - 1, y) \\)，遇到右界時就將該筆詢問加上 \\( \sum_{y = d}^{u} f(r, y) \\)，遇到點時則去更新 \\( f(x, y) \\)。

至於要如何維護 \\( f(x, y) \\) 並快速求出一段區間和呢？把 \\( f(x, y) \\) 想成一個數列，其 index 為 y 軸座標，這樣一來問題就變成動態加值以及區間查詢了，可以用線段樹或 BIT 維護。

回到原問題，我們可以將所有操作存下來並依照時間排序。這時若將操作序列切成左右兩段，會發現對於右方的詢問而言，左方的修改操作可以視為是靜態的。因此，可以先遞迴將左方的詢問處理完，再透過前面靜態版本的做法去計算左邊修改對右邊的詢問的貢獻，接著再遞迴右方，去把右邊的詢問算完。

這樣一來整體的時間複雜度為 \\( O((n + q)\log^2 (n + q)) \\)，跟使用二維線段樹的複雜度一樣。

```cpp
#include <bits/stdc++.h>
#define pb push_back
#define int long long
using namespace std;

const int N = 1e5 + 10;

struct BIT {
    int b[N * 10];
    vector<pair<int, int>> history;

    void upd(int p, int v, bool to_his = true) {
        if (to_his)
            history.pb(make_pair(p, v));
        while (p < 10 * N) {
            b[p] += v;
            p += p & -p;
        }
    }

    int query(int l, int r) {
        int sum = 0;
        l--;
        while (r) {
            sum += b[r];
            r -= r & -r;
        }
        while (l) {
            sum -= b[l];
            l -= l & -l;
        }
        return sum;
    }

    void undo() {
        while (!history.empty()) {
            auto [x, y] = history.back();
            upd(x, -y, false);
            history.pop_back();
        }
    }
} B;

struct op {
    int x, y, w;
    int d, u;
    int type, id;
    op() {}
    op(int _x, int _y, int _w, int _id)
        : x(_x), y(_y), w(_w), id(_id), type(0) {}
    op(int _x, int _y, int _d, int _u, int _id, int _t)
        : x(_x), y(_y), d(_d), u(_u), id(_id), type(_t) {}
};

int ans[N];

void CDQ(vector<op> &a) {
    if (a.size() <= 1)
        return;
    int mid = (a.size() - 1) >> 1;
    vector<op> al(mid + 1), ar(a.size() - mid - 1);
    for (int i = 0; i < a.size(); i++) {
        if (i <= mid)
            al[i] = a[i];
        else
            ar[i - mid - 1] = a[i];
    }
    CDQ(al);
    CDQ(ar);
    sort(al.begin(), al.end(), [](op x, op y) { return x.x < y.x; });
    vector<op> ar2;
    for (auto i : ar) {
        if (i.type) {
            ar2.pb(op(i.x, i.x, i.d, i.u, i.id, -1));
            ar2.pb(op(i.y, i.y, i.d, i.u, i.id, 1));
        }
    }
    sort(ar2.begin(), ar2.end(), [](op x, op y) { return x.x < y.x; });
    int p = 0;
    for (auto i : ar2) {
        while (p <= mid && al[p].x <= i.x) {
            if (al[p].type == 0)
                B.upd(al[p].y, al[p].w);
            p++;
        }
        ans[i.id] += B.query(i.d, i.u) * i.type;
    }
    B.undo();
}

void solve() {
    int n, q;
    cin >> n >> q;
    vector<int> v;
    vector<op> a(n);
    for (int i = 0; i < n; i++) {
        int x, y, w;
        cin >> x >> y >> w;
        v.pb(x);
        v.pb(y);
        a[i] = op(x, y, w, -1);
    }
    for (int i = 0; i < q; i++) {
        int t;
        cin >> t;
        if (t == 0) {
            int x, y, w;
            cin >> x >> y >> w;
            v.pb(x);
            v.pb(y);
            a.pb(op(x, y, w, i));
            ans[i] = -1;
        } else {
            int l, d, r, u;
            cin >> l >> d >> r >> u;
            a.pb(op(l - 1, r - 1, d, u - 1, i, 1));
            ans[i] = 0;
            v.pb(l - 1);
            v.pb(r - 1);
            v.pb(d);
            v.pb(u - 1);
        }
    }
    sort(v.begin(), v.end());
    v.resize(unique(v.begin(), v.end()) - v.begin());
    for (auto &i : a) {
        if (i.type == 0) {
            i.x = lower_bound(v.begin(), v.end(), i.x) - v.begin() + 1;
            i.y = lower_bound(v.begin(), v.end(), i.y) - v.begin() + 1;
        } else {
            i.x = lower_bound(v.begin(), v.end(), i.x) - v.begin() + 1;
            i.y = lower_bound(v.begin(), v.end(), i.y) - v.begin() + 1;
            i.d = lower_bound(v.begin(), v.end(), i.d) - v.begin() + 1;
            i.u = lower_bound(v.begin(), v.end(), i.u) - v.begin() + 1;
        }
    }
    CDQ(a);
    for (int i = 0; i < q; i++)
        if (ans[i] != -1)
            cout << ans[i] << '\n';
}

signed main() {
    ios::sync_with_stdio(0);
    cin.tie(0);
    solve();
}
```

> [Codeforces 848C](https://codeforces.com/problemset/problem/848/C)
>
> 給定 \\( N \\) 個珠子，每個珠子一開始的形狀為 \\( x_i \\)
>
> 接下來，要處理 \\( Q \\) 個操作，每個操作為以下其中一種：
>
> + 將第 \\( p \\) 個珠子的形狀改為 \\( x \\) \\( (1 \leq x \leq N) \\)
> + 詢問 \\( \sum_{i = 1}^{N} f(i, l, r) \\)
>
> 其中 \\( f(i, l, r) \\) 是指 \\( [l, r] \\) 區間中，最後一個出現 \\( i \\) 的位置減去第一個出現 \\( i \\) 的位置
>
> \\( N, Q \leq 10^5, 1 \leq x_i \leq N \\)

令 \\( pre_i \\) 為往左第一個與 \\( i \\) 形狀一樣的珠子位置 (若無則 \\( pre_i = i \\) )，那麼可以發現對於 \\( (l, r) = (1, n) \\) 而言，第二種詢問的答案即為 \\( \sum_{i = 1}^{n} i - pre_i \\)。

那麼對於其他狀況呢？這時候就要保證 \\( l \leq pre_i \leq i \leq r \\)，所以我們可以將 \\( (i, pre_i) \\) 視為二維平面上的一個點，其點權為 \\( i - pre_i \\)，這樣一來在需要被計算進答案的點就會都在以 \\( (l, l) \\) 為左下角，\\( (r, r) \\) 為右上角的矩形中，而修改某個珠子 \\( i \\) 的形狀就相當於將 \\( (i, pre_i) \\) 的點權歸零，並在新的 \\( (i, pre_i) \\) 更新點權。於是這個題目就轉換成二維平面單點加值矩形總和查詢了，與上一個例題相同。

至於要如何維護 \\( pre_i \\)？會發現在改變一個珠子的形狀時，\\( pre_i \\) 改變的數量為 \\( O(1) \\)，也就是分別為更改前後向右第一個同形狀的珠子，還有更改形狀的珠子本身。所以我們可以開 \\( N \\) 個 set 去紀錄每種形狀的珠子有哪些，這樣一來就可以在 \\( O(\log N) \\) 的時間找到 \\( pre \\) 會改變的珠子並更新他們。

```cpp
#include <bits/stdc++.h>
#define pb push_back
#define int long long
using namespace std;

const int N = 1e5 + 10;

struct BIT {
    int b[N];
    vector<pair<int, int>> history;

    void upd(int p, int v, bool to_his = true) {
        if (to_his)
            history.pb(make_pair(p, v));
        while (p < N) {
            b[p] += v;
            p += p & -p;
        }
    }

    int query(int l, int r) {
        int sum = 0;
        l--;
        while (r) {
            sum += b[r];
            r -= r & -r;
        }
        while (l) {
            sum -= b[l];
            l -= l & -l;
        }
        return sum;
    }

    void undo() {
        while (!history.empty()) {
            auto [x, y] = history.back();
            upd(x, -y, false);
            history.pop_back();
        }
    }
} B;

struct op {
    int x, y, w;
    int d, u;
    int type, id;
    op() {}
    op(int _x, int _y, int _w, int _id)
        : x(_x), y(_y), w(_w), id(_id), type(0) {}
    op(int _x, int _y, int _d, int _u, int _id, int _t)
        : x(_x), y(_y), d(_d), u(_u), id(_id), type(_t) {}
};

int ans[N];
int s[N], pre[N];
set<int> shape[N];

void CDQ(vector<op> &a) {
    if (a.size() <= 1)
        return;
    int mid = (a.size() - 1) >> 1;
    vector<op> al(mid + 1), ar(a.size() - mid - 1);
    for (int i = 0; i < a.size(); i++) {
        if (i <= mid)
            al[i] = a[i];
        else
            ar[i - mid - 1] = a[i];
    }
    CDQ(al);
    CDQ(ar);
    sort(al.begin(), al.end(), [](op x, op y) { return x.x < y.x; });
    vector<op> ar2;
    for (auto i : ar) {
        if (i.type) {
            ar2.pb(op(i.x, i.x, i.d, i.u, i.id, -1));
            ar2.pb(op(i.y, i.y, i.d, i.u, i.id, 1));
        }
    }
    sort(ar2.begin(), ar2.end(), [](op x, op y) { return x.x < y.x; });
    int p = 0;
    for (auto i : ar2) {
        while (p <= mid && al[p].x <= i.x) {
            if (al[p].type == 0)
                B.upd(al[p].y, al[p].w);
            p++;
        }
        ans[i.id] += B.query(i.d, i.u) * i.type;
    }
    B.undo();
}

void solve() {
    int n, q;
    cin >> n >> q;
    vector<op> a;
    for (int i = 1; i <= n; i++) {
        cin >> s[i];
        if (shape[s[i]].empty())
            pre[i] = i;
        else {
            pre[i] = *(shape[s[i]].rbegin());
            a.pb(op(i, pre[i], i - pre[i], -1));
        }
        shape[s[i]].insert(i);
    }
    for (int i = 0; i < q; i++) {
        int type;
        cin >> type;
        if (type == 1) {
            int p, x;
            cin >> p >> x;
            auto it = shape[s[p]].lower_bound(p);
            if (next(it) != shape[s[p]].end()) {
                int t = *next(it);
                a.pb(op(t, p, -t + p, -1));
                if (p == pre[p])
                    pre[t] = t;
                else {
                    pre[t] = pre[p];
                    a.pb(op(t, pre[p], t - pre[p], -1));
                }
            }
            a.pb(op(p, pre[p], -p + pre[p], -1));
            shape[s[p]].erase(it);
            s[p] = x;
            shape[s[p]].insert(p);
            it = shape[s[p]].lower_bound(p);
            if (next(it) != shape[s[p]].end()) {
                int t = *next(it);
                a.pb(op(t, pre[t], -t + pre[t], -1));
                pre[t] = p;
                a.pb(op(t, p, t - p, -1));
            }
            if (it != shape[s[p]].begin())
                pre[p] = *prev(it), a.pb(op(p, pre[p], p - pre[p], -1));
            else
                pre[p] = p;
            ans[i] = -1;
        } else {
            int l, r;
            cin >> l >> r;
            a.pb(op(l, r, l, r, i, 1));
            ans[i] = 0;
        }
    }
    CDQ(a);
    for (int i = 0; i < q; i++)
        if (ans[i] != -1)
            cout << ans[i] << '\n';
}

signed main() {
    ios::sync_with_stdio(0);
    cin.tie(0);
    solve();
}
```

## 斜率優化

可以套用斜率優化的題目通常可以將 DP 式化簡成 \\( dp_i = \min/\max_{j < i} \\{A_iX_j + B_iY_j + C_i\\} \\)。常見可以用動態凸包、李超線段樹等資結去做優化，而我們一樣可以用 CDQ 分治來取代掉資結。

在使用動態凸包時，我們會去二分搜凸包斜率最接近 \\( -\frac{A_i}{B_i} \\) 的邊來取得最佳答案，之後再把 \\( (X_i, Y_i) \\) 丟上動態凸包，而 CDQ 的想法也是類似的。將更新凸包和詢問的操作存在一個數列中並依照時間序去分成左右兩個區間，對於在相同區間的操作及詢問去分治解決，而對於操作在左區間，詢問在右區間的情況，由於左區間操作的時間序都比右區間的早，所以這些操作可以視為靜態的，這樣一來就將動態凸包轉為靜態版本了。

具體來說，定義 \\( solve(l, r) \\) 的功能為對於每個 \\( l \leq i \leq r \\)，利用 \\( l \leq j < i \\) 去更新 \\( dp_i \\) 的答案。在分治前，先將數列依照斜率 \\( -\frac{A_i}{B_i} \\) 排序，其作法為：

+ 將時間序 \\( \in [l, \frac{l + r}{2}] \\) 的分到左區間，\\( [\frac{l + r}{2} + 1, r] \\) 的分到右區間
+ 透過 \\( solve(l, \frac{l + r}{2}) \\) 去計算左區間的 dp 值並依照 \\( X_i \\) 做排序
+ 由於左區間操作的位置 \\( X_i \\) 具單調性，可以 \\( O(n) \\) 將左區間的點 \\( (X_i, Y_i) \\) 建成凸包
+ 又由於一開始依照斜率去排序，因此右區間斜率具單調性，可以用雙指標在 \\( O(n) \\) 時間內把左區間的最佳轉移點更新到右區間的點上
+ 透過 \\( solve(\frac{l + r}{2} + 1, r) \\) 把右區間的 \\( dp \\) 值算完
+ 將整個區間用 merge sort 依照 \\( X_i \\) 排序好

注意到這邊是先算完左區間對右區間的貢獻後才去分治計算右區間。這是因為在計算某個點的 dp 值前，需要保證其轉移點的 dp 值也都被計算完了，所以才要在計算右區間內的操作及詢問前，先將左區間的貢獻都計算到右區間上。

以 [Monster Game II](https://cses.fi/problemset/task/2085/) 來說，dp 式為 \\( dp_i = \min_{j < i}\\{dp_j + s_if_j\\} \\)，程式碼為 :

```cpp
#include <bits/stdc++.h>
#define int long long

using namespace std;

const int N = 2e5 + 10;
const int INF = 1e18 + 10;

struct point {
    int id, s, f;
};

int n;
int dp[N];
point a[N];

int cross(point m, point x, point y) {
    return (x.f - m.f) * (dp[y.id] - dp[m.id]) -
           (dp[x.id] - dp[m.id]) * (y.f - m.f);
}

bool slopeCmp(point p1, point p2, int sl) {
    return sl * (p2.f - p1.f) >= dp[p2.id] - dp[p1.id];
}

void calc(int l, int mid, int r) {
    vector<point> ans;
    int m = 0;
    auto addPoint = [&](const point p) {
        while (m > 1 && cross(ans[m - 2], ans[m - 1], p) <= 0)
            ans.pop_back(), m--;
        ans.push_back(p);
        m++;
    };
    for (int i = l; i <= mid; i++)
        addPoint(a[i]);
    if (ans.size() == 1) {
        for (int i = mid + 1; i <= r; i++)
            dp[a[i].id] = min(dp[a[i].id], dp[ans[0].id] + a[i].s * ans[0].f);
        return;
    }
    int p = 1;
    for (int i = mid + 1; i <= r; i++) {
        while (p < ans.size() && slopeCmp(ans[p - 1], ans[p], -a[i].s))
            p++;
        dp[a[i].id] =
            min(dp[a[i].id], dp[ans[p - 1].id] + a[i].s * ans[p - 1].f);
    }
}

void CDQ(int l, int r) {
    if (l == r)
        return;
    int mid = (l + r) >> 1;
    vector<point> tmp;
    for (int i = l; i <= r; i++)
        tmp.push_back(a[i]);
    int pl = l, pr = mid + 1;
    for (auto &i : tmp) {
        if (i.id <= mid)
            a[pl++] = i;
        else
            a[pr++] = i;
    }
    CDQ(l, mid);
    calc(l, mid, r);
    CDQ(mid + 1, r);
    tmp.clear();
    auto cmp = [](const point x, const point y) {
        if (x.f == y.f)
            return dp[x.id] < dp[y.id];
        return x.f < y.f;
    };
    for (pl = l, pr = mid + 1; pr <= r; pr++) {
        while (pl <= mid && cmp(a[pl], a[pr]))
            tmp.push_back(a[pl++]);
        tmp.push_back(a[pr]);
    }
    while (pl <= mid)
        tmp.push_back(a[pl++]);
    for (int i = l; i <= r; i++)
        a[i] = tmp[i - l];
}

void solve() {
    cin >> n >> a[0].f;
    dp[0] = a[0].s = a[0].id = 0;
    for (int i = 1; i <= n; i++)
        cin >> a[i].s;
    for (int i = 1; i <= n; i++) {
        cin >> a[i].f;
        a[i].id = i;
        dp[i] = INF;
    }
    sort(a, a + 1 + n, [](point x, point y) { return x.s > y.s; });
    CDQ(0, n);
    cout << dp[n] << '\n';
}

signed main() {
    ios::sync_with_stdio(0);
    cin.tie(0);
    solve();
}
```

## 其他例題

> [Luogu P3157](https://www.luogu.com.cn/problem/P3157)
>
> 給定一個長度為 \\( n \\) 數列 \\( a \\)，以及 \\( m \\) 筆刪除其中一個元素的操作。
>
> 輸出每次刪除操作前的逆序數對數量
>
> \\( 1 \leq n \leq 10^5 ,\ 1 \leq m \leq \min(n, 50000) \\)

<details><summary>Solution</summary>

就這題而言，我們可以反過來想「每次刪除的操作後少了幾個逆序數對」

記 \\( t_i \\) 為 \\( i \\) 這個位置被刪掉的時間，如果沒被刪除則為 \\( inf \\)。若將 \\( i \\) 這個位置刪除，則減少的逆序數對 \\( (i, j) \\) 皆符合以下兩種條件其中之一：

+ \\( (i < j) \land (a_i > a_j) \land (t_i < t_j) \\)
+ \\( (i > j) \land (a_i < a_j) \land (t_i < t_j) \\)

會發現這其實就是三維偏序問題。

因此我們能先算出一開始有多少逆序數對，並透過兩次 CDQ 去計算出對於每個時間 \\( t_i \\) 有幾組符合偏序關係的數對，也就是這次刪除後少了幾個逆序數對。

</details>

> [Codeforces 1093E](https://codeforces.com/contest/1093/problem/E)
>
> 給定兩個長度為 \\( n \\) 的 Permutation \\( a, b \\) 以及 \\( q \\) 筆操作，操作有以下兩種：
>
> + 詢問有幾個數同時出現在 \\( a \\) 中 \\( [l_a, r_a] \\) 的區間，以及 \\( b \\) 中 \\( [l_b, r_b] \\) 的區間
> + swap \\( (b_x, b_y) \\)
>
> \\( n, q \leq 2 \times 10^5 \\)

<details><summary>Solution</summary>

記 \\( x \\) 在 \\( a \\) 當中出現的位置為 \\( pa_x \\)，在 \\( b \\) 當中出現的位置為 \\( pb_x \\)。

可以發現若對於所有 \\( 1 \leq x \leq n \\)，我們將 \\( (pa_x, pb_x) \\) 視為二維平面上的一個點，那麼第一個操作就相當於詢問以 \\( (l_a, l_b) \\) 為左下角，\\( (r_a, r_b) \\) 為右上角的矩形中有多少個點。

而對於第二項操作，其實也就是刪除以及新增平面上的點而已，因此可以用前面例題一樣的方式去做。

</details>

## 練習題

>[Luogu P4090](https://www.luogu.com.cn/problem/P4690)
>
> 給定一個長度為 \\( n \\) 的數列 \\( a \\) 以及 \\( q \\) 筆操作，有兩種操作：
>
> + 將某個區間的值全部修改成一個值
> + 詢問某個區間中有多少不同的值
>
> \\( 1 \leq n, q \leq 10^5,\ 1 \leq a_i \leq 10^9 \\)

<details><summary>Hint</summary>

一樣朝著前面例題 \\( pre_i \\) 的方向去想

</details>

>[CEOI2017 day2 building bridges](https://csacademy.com/contest/archive/task/building-bridges/)
>
>一共有 \\( n \\) 個柱子，每個柱子的高度為 \\( h_i \\)，要選擇某些柱子去在上面建立橋樑，其中一定要包含第 \\( 1 \\) 個及第 \\( n \\) 個柱子
>
>在 \\( (i, j) \\) 兩個柱子建立橋樑的費用為 \\( (h_i - h_j)^2 \\)，並且對於那些沒選到的柱子，還需要額外花費 \\( w_i \\) 的費用來拆除，問最小花費
>
>+ \\( 2 \leq n \leq 10^5 \\)
>+ \\( 0 \leq h_i \leq 10^9 \\)
>+ \\( 0 \leq \lvert w_i \rvert \leq 10^9 \\)

<details><summary>Hint</summary>

考慮 dp，令 \\( f(i) \\) 為在前 \\( i \\) 個柱子中選擇了第 \\( i \\) 個柱子的最小花費。

</details>

## Reference

+ [https://oi-wiki.org/misc/cdq-divide/](https://oi-wiki.org/misc/cdq-divide/)
+ [https://www.cs.princeton.edu/~danqic/papers/divide-and-conquer.pdf](https://www.cs.princeton.edu/~danqic/papers/divide-and-conquer.pdf)
+ [https://zhuanlan.zhihu.com/p/332996578](https://zhuanlan.zhihu.com/p/332996578)
+ [https://robert1003.github.io/2020/01/31/cdq-divide-and-conquer.html](https://robert1003.github.io/2020/01/31/cdq-divide-and-conquer.html)
