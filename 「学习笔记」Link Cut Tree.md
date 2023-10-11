# 定义

起源是有些树要动态加边或者删边，所以这时候用到了 $lct$

我们把 $splay$ 放到外层作为**外层树**，每个点被包含在一个单独的 $splay$ 中

其实这里主要是运用了 $splay$ 可以区间反转的功能（当然可以 $fhq$ 但是复杂度多一个 $\log$）

同时在 $splay$ 里面**中序遍历得到的点的深度是递增的**，也就是说不一定在当前 $splay$ 的根就是这个子树里面深度最浅的点

因此我们把边就得拆成实边和虚边，这里在同一个 $splay$ 里面的边都是实边，而不在里面的边则是虚边

这里注意：父子连边不变，不是说当前 $splay$ 的根连向下个 $splay$ 的根来表示连边

那么就有一些定义

$fa[x]:$ $x$ 在 $splay$ 里面的父亲，不是原树

$ls[x],rs[x]$ 在 $splay$ 里面的左右儿子，真树里面的连边都是靠 $fa[son]=x$ 来的
 
# 操作

## access

联通根到当前点的链

这个每个 $splay$ 跟着做就行了，每次把当前点干到 $splay$ 的根，然后改儿子

这里是把 $rs[fat]=now$ ，根据深度的原则不难得到

所以简单的代码如下：

```cpp
inline void access(int x){
	for(reg int y=0;x;x=fa[y=x]) splay(x),rs[x]=y,push_up(x);
    return ;
}
```

其实这里是换了个更的方式：把更新儿子的步骤放到了上面，这里被覆盖的儿子的父亲没变，但是父子关系变成了虚边 

## makeroot

指定根

很好说，直接打通链时候翻上去就行了，但是很坑的是这样的话没有深度保证

所以要翻转整个当前 $splay$，打标记即可，和普通 $splay$ 没有区别

```cpp
inline void pushroot(int x){
	swap(ls[x],rs[x]); fl[x]^=1; 
    return ;
}
inline void makeroot(int x){
	access(x); splay(x); pushroot(x);
    return ;
}
```

## findroot

找到原树上的根

换到根之就找左儿子，也就是：

```cpp
inline int findroot(int x){
    access(x); splay(x); 
    while(ls[x]) x=ls[x]; 
    return splay(x),x;//多splay来保证复杂度……
}
```

当然这样写是非常慢的，那么**特定场景**下可以用并查集来进行替换和卡常

## split

打通一条链，随便钦定一个点为根然后连上就行了

```cpp
inline void split(int x,int y){
    makeroot(x); access(y); splay(y); 
    return ;
}
```

然后直接查询 $s[y]$ 就是这个路径上面的信息，需要理解一下为什么这样就能维护出来单独的路径

## link

连接两个点之间的边

```cpp
inline void link(int x,int y){
	make_root(x); if(findroot(y)!=x) fa[x]=y; 
    return ;
}
```

## cut

断开边，先把一个点转到必然是另一个的父亲然后判断合法性

也就是说：

```cpp
inline void cut(int x,int y){
	make_root(x); 
    if(findroot(y)!=x||fa[y]!=x||ls[y]) return ;
    //第一个是不在一个树上，findroot(y) 之后 x 是这个splay的根
    //如果y的父亲不是x那么必然没有连边，考虑findroot中的更改
    //如果有ls[y] 那么就有越级父亲
    rs[x]=0; fa[y]=0; push_up(x);
    return ;
}
```

这里写 $lct$ 的 $splay$ 和一般的不太一样，具体如下：

$(1)$ 维护一个是不是当前splay的根的函数：

```cpp
inline bool isroot(int x){return ls[fa[x]]!=x&&rs[fa[x]]!=x;}
```

$(2)$ $rotate$ 和 $splay$ 的时候记得判断 $z=fa[fa[x]]$ 的情况，不能找到另一个splay上面

$(3)$ 下方 $makeroot$ 标记的时候要注意从上往下，原因可以手玩一下

这样的话板子就随便打了

# 功能

## 维护链上信息

其实本质上就是 $split$ 一下，然后有各种打标记的方式

### Luogu 1501

裸题，直接打标记就行了

### Luogu 4332

显然改变一个 $[n+1,3n]$ 的点的本质会修改一条链上的答案

肯定是临界的会改，问题转化成了维护链上最深的 $cnt[x]$ 不是 $1/2$ 的点的位置

打通一条链的操作就是 $access$ ，然后在平衡树上二分

具体而言就是 $push\_up$ 维护几个子树面是不是都是 $1/2$ 即可

时间复杂度 $O(n\log^2 n)$

貌似有一个少 $\log$ 的写法，就是记录子树里面的最深 $val\neq 1/2$  的点

如果修改值的话需要交换

这题写的原则就是多 $push\_up $

## 维护双联通分量/联通性

### Luogu2542 

逆序之后考虑如何维护必经边

这个必经边容易让人想到点双，那么考虑缩点

每次如果 $link$ 失败了就删掉环上的点，缩成一个新点，所有连的点都指向这个点

但是需要更改的是 $access$ 的时候是要跳 $find(fa[x])$ 的

## 维护生成树或者树边信息

边权很难维护，如果按照一些树题放到儿子上的话一变父子关系就废掉了

所以考虑拆点，把边权放到一个新的点上，因为比较优秀的编号方式，每个新点的 $id>n$ 

所以更改或者一些其他操作就好说了

```cpp
link(e[i].id,e[i].x); link(e[i].id,e[i].y);
cur(e[i].id,e[i].x); cut(e[i].id,e[i].y);
```

### NOI2014 魔法森林

看到是多维的先想降维，所以 $sort$ 掉 $a/b$

那么然后的问题是如何维护一个用 $a$ 最小的生成树

那么每次加边如果成环就断掉环上最大的 $a$ 然后更新答案

为啥原来那么菜还要去抄题解呀

## 维护虚子树信息

记 $s_i$ 表示虚子树信息，那么不一样的地方首先是 $access$

```cpp
for(reg int y=x;x;x=fa[y=x]){
    splay(x); 
    s[x]-=s[rs[x]],si[x]+=rs[x];
	s[x]+=s[y],si[x]-=s[y];
    rs[x]=y;
}
```

然后 $link$ 的时候要先把 $y$ 整到根上面，防止祖先的信息被记漏

```cpp
inline void link(int x,int y){
    make_root(x); if(findroot(y)==x) return ;
    fa[x]=y; access(y); splay(y); si[y]+=s[x]; 
    push_up(y); //真的别忘记了多push_up
    return ;
}
```

### BJOI2014 大融合

貌似线段树合并随便做？

如果 $lct$ 的话考虑如果 $split(x,y)$ 之后那么有剩下的边就是虚的了

那么维护上 $s[x],si[x]$ 查询的时候回答 $s[x]\times(s[y]-s[x])$

## 维护树上染色联通块

### SP16549

新建一个根作为 $1$ 的父亲，对于修改颜色操作，就把原来父亲节点的边断开，连上新的

需要用 $lct$ 维护虚子树的大小