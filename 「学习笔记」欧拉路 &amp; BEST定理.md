# 欧拉路

## 定义

欧拉路：图中经过所有边恰好一次的路径

欧拉回路：起点和终点相同的欧拉路

## 判定

分为有向图和无向图两种情况：

$1.$ 无向图

- 所有点的度数都是偶数，此时起点任选两个即可
 
- 恰好有两个点的度数是奇数，此时两个奇度数点为起点或终点

$2.$ 有向图

- 所有点的入度都等于出度，此时存在欧拉回路，所有点都可以做起点

- 有两个点入度不等于出度，同时满足两个点有一个入度大于出度，有一个入度小于出度，此时出度大的是起点，入度大的是终点

## Hierholzer 算法

暴力算法就是对于当前到的点，枚举所有的出边，如果没访问过就搜过去，复杂度 $O(nm)$

然后发现缺陷在于每次都从头开始，这并不优秀，加上一个类似 $dinic$ 当前弧优化的东西就行了

使用 `std::vector` 存边就更为简洁了：每次找到数组中最后一条边 `pop_back` 即可

```cpp
inline void dfs(int x){
    for(int i=head[x];i;i=e[i].nxt){
        if(vis[i]) continue;
        int t=e[i].to; vis[i]=vis[i^1]=1;
        head[x]=i; dfs(t); i=head[x];
    }
    st[++top]=x;
    return ;
}
```

复杂度 $O(n+m)$，另外也可以手写栈来减小递归带来的影响，注意栈大小开到 $\Theta(m)$ 级别

```cpp
int x,r[N*10],i,cnt,top,st[N*10];
inline void find(){
    st[++top]=1; 
    while(top){
        int x=st[top],i=head[x]; 
        while(vis[i]) i=e[i].nxt;
        if(i){
            int t=e[i].to;
            vis[i]=vis[i^1]=1; 
            head[x]=i;
        }
        else top--,r[++cnt]=x;
    }
    return ;
}
```

答案就是栈内元素的倒序

## 例题

### Hdu3018

每个无向图上的欧拉路满足的是起点和终点都是入度为 $2$ 所以

对于每个联通块考虑，直接找有多少个点的度数是奇数， $\lfloor\frac {num+1} 2\rfloor$ 即可

### 星际旅行

每条边拆成两个，拆完之后入度必然是偶数

然后删掉一些两条边使得图上有一条欧拉路即可

这就直接从度数角度考虑就行了

$1.$ 删掉两个自环

$2.$ 删掉一个自环和随便这个点连一个边

$3.$ 删掉两条这个点出发的边

### AGC018F

每个点的权值只有 $\{0,-1,1\}$ 的时候也可能满足条件，每个点是几取决于儿子的数量

不难发现每个儿子的权值都是奇数，那么如果在两棵树中儿子的数量的奇偶性不同则无解

否则建立虚拟源点，连向两棵树的根，并在将两棵树中度数为奇数的点连边，即 $(i+n,i)$，这里非根节点度数的奇偶性与其儿子数量相异

这时候每个点的度数都是偶数了，那么在图上跑欧拉回路通过边的方向就可以得到每个奇数儿子点的正负性

考虑到走入一个子树的时候所有点都为 $0$，最后从子树的根走出时满足层间的上下次数相同，那么子树中非根节点的权值和为 $0$

可能是 CF1610F 的祖宗题，但是该题标算并不是这个

# BEST 定理

有向欧拉图 $G=(V,E)$ 中以任一 $s[s\in V]$ 的 $s$ 为起始终止点的欧拉**回路**的条数可以用如下公式计算得到：

$$F(s)=deg_s!T_s(G)\prod_{x\in V,x\neq s} (deg_x-1)!$$

这里两个不同欧拉回路定义为经过边的顺序不同

这里的 $deg_x$ 表示 $x$ 的出度，由于是有向欧拉图，那么满足出度入度相等，所以统一使用 $deg_x$ 表示

$T_s(G)$ 表示以 $s$ 为根的内向树个数

务必注意这里的 $deg_x$ 为 $0$ 时不在矩阵树定理中进行行列式的计算以及 BEST 定理的使用前提是 $G=(V,E)$ 是有向欧拉图

什么？不会算内向生成树？$|K_{out}-Adj|$，外向生成树就是 $|K_{in}-Adj|$

而应用到本定理的时候由于入度出度一样所以可以任一写

## 证明

我们构造一组双射来证明两者相等：

- Part I：每棵内向树 唯一对应一条 欧拉路径

    考虑上面表达式的实际含义：对于非 $s$ 节点钦定欧拉路中每个点非最后一次走出去的出边，剩下的边按照任意顺序走，终点 $s$ 的所有出边可以任意选取，那么剩下的 $n-1$ 条边构成一棵内向树
    
    那么要完成的工作就是将一棵树加边还原成一个欧拉路径，考虑其逆过程删边：如果按照走法删掉若干条边都能保证图上仍然存在欧拉**路（不一定是回路）** 即可

    使用欧拉路存在的判定充要条件：点度数和有向边弱联通

    点度数由于是合法原图每次改变两个得到的，显然合法；非树边删除时所有边可以通过树边联通，树边删除本质上是点叶子被剥去，所以不会有待经过的边从联通块上删除

- Part II：每条欧拉路径 唯一对应 一棵内向树

    问题可以转化为：将欧拉路中每个点非最后一次走出去的出边删去，同时删去钦定的终点 $s$ 的所有出边，试证剩下的 $n-1$ 条边构成一棵内向树

    唯一的疑问点是如何证明无环：如果产生环，由于找到环上任意一个不是 $s$ 的点，那么沿着环走回到自己之后发现没有出边，而更不能作为终点，所以产生了矛盾

    由于 $s$ 一定不在环上所以这样的点一定可以被找到

## In Addition

尝试不考虑端点地计算有向欧拉图中欧拉回路数量：

根据上面的证明，每种欧拉回路由于 $s$ 的最后一个出边被砍掉后任选了，那么被记重复了 $deg_s$ 次，除掉即可，形式更为优美

$$F(s)=T_s(G)\prod_{x\in V} (deg_x-1)!$$