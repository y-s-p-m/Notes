# WQS 二分

考虑有一类问题是 $n$ 选 $k$ 然后求最优价值

如果发现这个 $k$ 和代价 $f(k)$ 构成的函数是凸的，那么可以考虑 $WQS$ 二分

考虑用一个直线去切原函数，而能得到合法的截距的最值也就是问题的最优解

那么考虑移项得到 $b=f(x)-kx$ 

那么我们对于所有的物品增加一个附加权值 $x=-mid$，同时记录选择物品的个数 $cnt$

最后对于一个特定的 $k=mid$，判断是否合法来更新答案即可

既然是 $\rm{DP}$ 优化的手段，那么二分里面常配合 $\rm{DP}$

# 例题

## HEOI2018 林克卡特树

本题二分 $mid$ 之后，合法与否即维护出来链的数量和 $K$ 是否相同

如果二分的时候出现了有多个决策点的情况，那么记录下来 $x_{max}$  最后带入求值

此时问题转化为求若干个不相交的链和最大值，只需不交，不考虑个数

设 $f_{i,0/1/2}$ 表示当前点和当前链的关系

$0$ 是不在链上，$1$ 表示在链的一端，$2$ 表示在链的中间

根据含义容易得到朴素的转移式子

<details>
<summary>Code Display</summary>

```cpp
const int N = 3e5 + 10;
struct nd {int to, nxt, dis;} e[N << 1];
int head[N], tot, n, m, k;
inline void add(int u, int v, int w) {
    e[++tot].to = v; e[tot].nxt = head[u]; e[tot].dis = w; 
    return head[u] = tot, void();
}
struct node {
    int dp, cnt;
    node() {};
    node(int x, int y) {dp = x; cnt = y; return;}
    bool operator<(const node &a)const {
        if (dp ^ a.dp) return dp < a.dp;
        return cnt < a.cnt;
    }
    node operator+(const node &a)const {return node(dp + a.dp, cnt + a.cnt);}
} f[N][3];
inline node max(node a, node b) {return a < b ? b : a;}
inline int max(int x, int y) {return x < y ? y : x;}
inline int calc(int x) {return max(f[x][1].dp, max(f[x][0].dp, f[x][2].dp));}
inline void dfs(int x, int fa, int now) {
    f[x][0] = node(0, 0);
    f[x][1] = node(-1e13, 0);
    f[x][2] = node(-now, 1);
    for (reg int i = head[x]; i; i = e[i].nxt) {
        int t = e[i].to; if (t == fa) continue;
        dfs(t, x, now);
        node mxt = max(f[t][0], max(f[t][1], f[t][2]));
        f[x][2] = max(f[x][2] + mxt, f[x][1] + max(node(f[t][0].dp + e[i].dis, f[t][0].cnt),
                      node(f[t][1].dp + e[i].dis + now, f[t][1].cnt - 1)));
        f[x][1] = max(f[x][1] + mxt, f[x][0] + max(node(f[t][0].dp + e[i].dis - now, f[t][0].cnt + 1),
                      node(f[t][1].dp + e[i].dis, f[t][1].cnt)));
        f[x][0] = f[x][0] + mxt;
    } return ;
}
signed main() {
    n = read(); k = read()+1;
    for (reg int i = 1, u, v, w; i < n; ++i) u = read(), v = read(), w = read(), add(u, v, w), add(v, u, w);
    int l = -1e13 - 10, ans = 0, r = 1e13 + 10;
    while (l <= r) {
        int mid = (l + r) >> 1;
        dfs(1, 0, mid);
        node mx = max(f[1][0], max(f[1][1], f[1][2]));
        if (mx.cnt < k) r = mid - 1;
        else l = mid + 1, ans = mid;
    }
    dfs(1, 0, ans); printf("%lld\n", calc(1) + k * ans);
    return 0;
}
```
</detail>