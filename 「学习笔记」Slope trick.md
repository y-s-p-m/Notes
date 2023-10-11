$\rm{Slope\ trick}$ 并不是一个特别的 $\rm{algorithm}$，只是一个朴素维护折线的方式

一类题目中要维护一类特殊的分段函数，满足函数连续，每段都是一次函数，斜率为整数

从一道ABC题目开始

# ABC217H

设 $dp_{i,j}$ 表示经过前 $i$ 次攻击后当前处于位置 $j$ 的最小代价，转移设 $t_i-t_{i-1}=dst$

那么转移只可能从时间段里面能走到的部分走过来，方程即：

$$dp_{i,j}=\min\limits_{t\in [j-dst,j+dst]}\{dp_{i-1,t}\}+G_i(j)$$

其中 $G_i(x)$ 表示第 $i$ 次攻击站到 $x$ 时付出的代价

不难发现无论 $d_i$ 取值如何，$G_i(x)$ 都是一个斜率单调递增的函数，即只会是 $y=-x+b\to y=b$ 或 $y=b\to y=x+b$ 中之一

转移方程可以看成将若干个凸壳对位加入，其斜率仍然单调递增，而加上取 $\min$ 操作还是个凸壳，区别就在将折线的拐点平移了

不难发现其实我们最后需要的答案一定是在斜率为 $0$ 的地方得到的，所以用两个堆分别记录斜率 $>0$ 和 $<0$ 的折线

区间取 $\min$ 对应将凸包拐点平移 $\pm dst$，原问题中给一个后缀的斜率加 $1$，前缀斜率减 $1$ 是新添加转移点

最后注意堆中的一个节点表示给斜率变化为 $1$，若出现一个点和其相邻的点的斜率差不为 $1$ 时，要存多个当前点

当然也可以开 $(x,num)$ 来处理

具体实现看代码，转移的处理仍需读者自行思考

```cpp
struct Heap{
    multiset<int> st;
    int tag;
    inline void insert(int x){return st.insert(x-tag),void();}
    inline void erase(int x){st.erase(st.find(x-tag)); return ;}
    inline void push(int x){tag+=x; return ;}
    inline int least(){return tag+(*st.begin());}
    inline int most(){return tag+(*--st.end());}
}sl,sr;
int n,ans,lst;
signed main(){
    n=read(); rep(i,1,(n<<2|1)) sl.insert(0),sr.insert(0); rep(i,1,n){
        int t=read(),d=read(),x=read(); int dis=t-lst; lst=t; 
        sl.push(-dis); sr.push(dis);
        if(d==1){
            if(sl.most()<=x){sr.insert(x);continue;}
            sl.insert(x); int tmp=sl.most(); sr.insert(tmp); sl.erase(tmp);
            ans+=sr.least()-x;
        }else{
            if(sr.least()>=x){sl.insert(x);continue;}
            sr.insert(x); int tmp=sr.least(); sr.erase(tmp); sl.insert(tmp);
            ans+=x-sl.most();
        }
    } print(ans); return 0;
}
```

# CF713C

本题将绝对值函数的两部分都需要加入代价统计

每次加入拐点的时候给左边斜率减一，右边加一，那么造成两边的斜率差为 $2$，所以需要多往堆里面放一个点

另外不难发现转移函数是一个前缀 $\min$ 的形式，那么只需要维护左半边即可

每次加入一个小于当前堆顶的 $x$ 会产生附加的代价，将凸壳话出来就能发现这个值是 堆顶和 $x$ 的差

# CF1534G

将曼哈顿距离转化成切比学夫距离有 $(x+y,x-y)$，此时向右和向上的移动变成走到 $(x+1,y+1)/(x+1,y-1)$

至此有 $\Theta(nM)$ 的做法即 $dp_{i,j}$ 表示更改坐标系之后走到了 $(i,j)$，解决了 $x+y\le i$ 的布置的最小代价，这时候每次的代价仍然是绝对值的形式

有没有非常熟悉？就还是 ABC 题的转移方程，只是这时候两边函数全了而已

仍然是双堆维护即可

# APIO2016 烟火表演

$dp_{x,i}$ 表示在 $x$ 的子树里面用 $i$ 时刻点完的最小代价，这里的时刻是相对时刻

合并子树的时候还是熟悉的斜率相加，这时候不能再维护两边了，因为转移式子比较繁琐，需要维护整个凸包

但是拐点的个数是合法的，那么可以使用左偏树来维护，注意我们并不关注斜率大于 $1$ 的部分，而能造成 $k\ge 1$ 的有且只有儿子个数个，那么记得上来先弹掉

最后答案的统计比较巧，先计算 $f(0)$ 即边权和，再逐一减掉那些小于 $0$ 的斜率的贡献