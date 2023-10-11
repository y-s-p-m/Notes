# 介绍

由于$spfa$容易被卡，实际上我们在$O(nlog \space n)$ 的算法只有堆优化的$Dijkstra$

由于先天问题，$Dijkstra$无法处理在负权图上的问题

所以“$Johnson$全源最短路”算法就应运而生了

# 算法流程

我们针对$Dijkstra$无法处理负权图进行优化

我们考虑如何把每条边的权值转化成正数

这里引入**“势能”**的概念

势能需要一个起始点：建立一个虚拟源点，向每一个点连一条权值为$0$的有向边

跑一遍$spfa$，记录每个点到虚拟源点的最短路，记为 $res$，因为是

（这里问显然$dis=0$的同学，请注意这可能是一个负权图）

然后我们把每一条边的边权加上两端势能： `res[e[num].from]-res[e[num].to];`

然后我们跑$n$次$Dijkstra$，最后将势能减去，即 `ans[from][to]-=res[from]-res[to];`

复杂度$O(n^2 \space log \space n+nm)$

```cpp
const int N=4e3+10;
struct node{int to,dis,nxt;}e[N<<3];
int head[N],n,m,cnt,app[N],res[N],ans,tmp[N]; bool vis[N];
inline void add(int u,int v,int w){
    e[++cnt].dis=w; e[cnt].nxt=head[u]; e[cnt].to=v; 
    return head[u]=cnt,void();
}
inline bool spfa(int s){
    queue<int> q; memset(res,0x3f,sizeof(res));
    q.push(s); vis[s]=1; res[s]=0;
    while(!q.empty()){
        int fr=q.front(); q.pop(); vis[fr]=0;
        for(int i=head[fr];i;i=e[i].nxt){
            int t=e[i].to,dist=e[i].dis+res[fr];
            if(res[t]>dist){
                res[t]=dist;
                if(!vis[t]){
                    // Negative Circle
                    if(++app[t]>=n) return 0;
                    vis[t]=1; q.push(t);
                }
            } 
        }
    }
    return 1;
}
#define mp make_pair
inline void dij(int s){
    priority_queue<pair<int,int> > q;
    q.push(mp(0,s)); memset(vis,0,sizeof(vis)); 
    for(int i=1;i<=n;++i) tmp[i]=1e9; tmp[s]=0; ans=0;
    while(!q.empty()){
        int fr=q.top().second; q.pop(); 
        if(vis[fr]) continue; vis[fr]=1;
        for(int i=head[fr];i;i=e[i].nxt){
            int t=e[i].to,dist=e[i].dis+tmp[fr];
            if(tmp[t]>dist){
                tmp[t]=dist;
                if(!vis[t]) q.push(mp(-dist,t));
            }
        }
    }
    for(int i=1;i<=n;++i){
        if(tmp[i]==1e9) ans+=tmp[i]*i;
        else ans+=i*(tmp[i]+res[i]-res[s]);
    } 
    return printf("%lld\n",ans),void();
}
signed main(){
    n=read(); m=read(); for(int i=1,u,v,w;i<=m;++i) u=read(),v=read(),w=read(),add(u,v,w);
    for(int i=1;i<=n;++i) add(n+1,i,0); if(!spfa(n+1)) return puts("-1"),0;
    for(int i=1;i<=n;++i) for(int j=head[i];j;j=e[j].nxt) e[j].dis+=res[i]-res[e[j].to];
    for(int i=1;i<=n;++i) dij(i);
    return 0;
}
```

# 应用前景

这个破玩意学它有什么用呢？

求解最小费用最大流就是不断求解最短路,然后通过最短路增广的过程

由于走反向边费用要取负,无法使用 $Dijkstra$ 增广,只能通过 $Bellman\text{-}Ford$

而有了 $Johnson$ 算法以后,就可以先做一轮 $Bellman\text{-}Ford$,

然后不断通过 $Dijkstra$ 增广，运行更稳定,效率更高