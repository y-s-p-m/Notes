貌似看懂板子了，那么来更一更

# min25筛

求积性函数前缀和，复杂度 $O(\frac{n^{0.75}}{\log n})$，常数比较小

不支持多点求，同时需要满足 $f(prime)$ 是关于 $prime$ 的低次多项式（很低次）

这个低次比如 $\varphi (prime)=prime-1$，其实这个东西是个能快速求的多项式就行了

先把 $\sum f(i)$ 转成质数的和加合数的和，那么这个算法主要是分成两个部分：第一个 $\rm{DP}$ 和第二个 $\rm{DP}$

## Part I：DP

考虑如下一个 $\rm{DP}:$

$$\rm{g_k(n,j)=\sum_{i=1}^n[i\in {}\{prime\}||minp(i)> P_{j}]i^k}$$

表示 $n$ 以内质数或者所有最小质因子大于 $\rm{Prime_j}$ 的合数的 $k$ 次方和

转移就是删掉最小质因子为 $j$ 的数，具体如下：

$$g_k(n,j) =\begin{cases}g_k(n,j-1)-p_j^k\times(g_k(\frac{n}{p_j},j-1)-g_k(p_j-1,j-1))[p_j^2\le n]\\g_k(n,j-1)\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \  \ \ \ \  [p_j^2>n]\end{cases}$$

第一种转移乘的 $p_j^k$ 表示贡献，考虑 $i^k$ 是完全积性函数，所以直接除不用管互质

最后减去 $g(p_j-1,j-1)$ 表示删去质数在 $g(n,k)$ 中的贡献

关注到转移的时候对于 $[1,n]$ 的每个数只关注其 $\frac n{\rm{Prime}}$ 的值，那么先 $\sqrt n$ 处理出来所有 $\lfloor\frac ni\rfloor$ 的结果

看起来这样子计算的时候只关注了每个整除区间右端点的 dp 值，如果随机访问怎么办？

不难发现所求是 $g_k(n,n)$，根据转移方程求解的时候只需要 $\lfloor\frac{N}{x}\rfloor$，所以预处理 $\lfloor \frac ni\rfloor$ 的结果即可

这部分的复杂度分析大可直接计算循环了多少次，这个复杂度甚至和杜教筛的分析完全一样，只是由于只扫描预处理了的质因子，那么要除一个 $\log$

## Part II: DP

接下来设 $S(n,x)$ 表示 $n$ 以内所有最小质因子大于 $Prime_x$ 的数的 **函数值** 之和，$g(n)$ 表示 $\le n$ 范围内所有质数函数值之和，$sum_x$ 表示前 $x$ 个质数的函数值之和

根据定义：序号 $\le x$ 的质数并不计算在 $S(*,x)$ 中。所以答案是 $S(n,0)$

转移如下：

$$S(n,x)=g(n)-sum_x+\sum_{p_k^c\le n,k> x} f(p_k^c)\times (S(\frac{n}{p_k^c},k)+[c\neq 1])$$

含义其实是比较显然的，注意后面的求和又一次使用了积性函数的性质，加 $[c\neq 1]$ 的含义是如果此时不为 $1$ 还需要把 $p_c$ 的函数值加入答案，非常巧妙

直接递归统计贡献即可，也不需要记忆化

## 实现

先预处理每个质数处的前缀和和每个 $g_k(x,0)$，根据具体含义，这些东西都是可以通过次方前缀和来做出来的

不难发现对于高次自然数幂前缀和，需要伯努利数或者插值，这也就是处理高次的时候会复杂的原因

处理 $g_k(x,0)$ 时只处理每个整除区间右端点的函数前缀和，然后做第一部分的 $\rm{DP}$（直接对着式子板抄即可）

对于第二部分中的 $g(n)-sum_p$ 这个直接对着多项式做减法即可，剩下的都是裸的实现，无需赘述

一些细节：

- 整除分块的数组要开 $2$ 倍，原因是 $\lfloor \frac{n}x\rfloor$ 的结果要分成 $\le \sqrt n$ 和 $\ge \sqrt n$ 的两部分

- 处理 $1$ 的贡献通常的办法是都删掉，最后加上一个 $f(1)$ 

代码是模板题 LOJ6053

```cpp
const int N=2e5+10,mod=1e9+7,inv2=5e8+4;
int fl[N],pri[N],num,tot,bl,n,id1[N],id2[N],g2[N],g1[N],s1[N],R[N]; 
inline int get(int x){return x<=bl?id1[x]:id2[n/x];}
inline int calc(int x,int y){
    if(x<=1||pri[y]>x) return 0;
    int pos=get(x),ans=del(del(g2[pos],g1[pos]),(s1[y-1]+1-y+mod)%mod);
    if(y==1) ans=add(ans,2); 
    for(int i=y;i<=num&&pri[i]*pri[i]<=x;++i){
        int prod=pri[i];
        for(int j=1;prod*pri[i]<=x;prod*=pri[i],++j){
            ans=add(ans,add(mul(calc(x/prod,i+1),pri[i]^j),pri[i]^(j+1)));
        } 
    } return ans;
}
signed main(){
    n=read(); bl=sqrt(n);  
    for(int i=2;i<=bl;++i){
        if(!fl[i]) pri[++num]=i,s1[num]=add(s1[num-1],i);
        for(int j=1;j<=num&&pri[j]*i<=bl;++j){
            fl[i*pri[j]]=1; if(i%pri[j]==0) break;
        } 
    }
    for(int l=1,r;l<=n;l=r+1){
        r=(n/(n/l)); R[++tot]=n/l;
        // R[i] 记录区间右端点
        g1[tot]=del(R[tot]%mod,1); 
        g2[tot]=mul(add(R[tot]%mod,2),mul(del(R[tot]%mod,1),inv2));
        if(n/l<=bl) id1[n/l]=tot; else id2[r]=tot; 
    } 
    for(int i=1;i<=num;++i){
        for(int j=1;pri[i]*pri[i]<=R[j];++j){
            int k=get(R[j]/pri[i]);
            // 因为 (n/a)/b) = (n/(ab)) 那么可以直接找 id
            g1[j]=del(g1[j],del(g1[k],i-1));
            g2[j]=del(g2[j],mul(pri[i],del(g2[k],s1[i-1])));
        } 
    } printf("%lld\n",add(1,calc(n,1))); 
    return 0;
}
```

# 例题

## LOJ6625

当头一棒

这题目告诉我们 $min25$ 筛不一定要写第二部分，（那么这题应该放到 $DP$ 杂题……）

考虑合数处的取值是 $\rm{minp(i)-2}$，那么使用第一部分的 $DP$ 能统计出来每个质数作为多少质数的 $\rm{minp}$

这种题对于博主这种一根筋的废物还是很有用的！

## LOJ3069

现场居然能过掉 $4$ 个人，候选队实力恐怖如斯

面向数据范围不难考虑对 $f(x)$ 是否是积性函数进行思考，而根据定义我们发现圆上的整点是平均分布在四个象限里面的，那么对 $f(i)/4$ 打表

好耶，是积性函数！但是如何证明呢？

首先证明两个结论：

- 奇质数 $x$ 可以被分解成 $a^2+b^2$ 当且仅当 $x\equiv 1\ (\bmod\  4)$

	根据二次剩余的一些理论，得到此时存在一个 $a^2 \equiv -1 (\bmod \  x)$

	那么构造 $as_1+t_1\equiv as_2+t_2\mod p$ 满足 $s_1,s_2,t_1,t_2\le \sqrt p$

	此时满足条件的的 $x=|s_1-s_2|,y=|t_1-t_2|$，证明带入即可，那么发现 $x^2+y^2 \equiv 0\mod p,x^2+y^2\le p$

定义高斯整数 $x+yi[x,y\in\mathrm{Interger}]$ 和其范数 $N(x+y_i)=x^2+y^2$

同时定义高斯质数为不能表示为若干个高斯质数乘积的数，这里使用上面的结论，发现高斯质数就是 $p \equiv 3\mod 4$ 的质数

定义两个不同的高斯整数是相互伴随的当且仅当一个通过旋转 $90,180,270$ 可以成为另一个

考虑这样一个性质：对于高斯整数，也存在唯一分解定律，这里分解出来的结果每个数可以替换为其伴随

这个性质考虑归奶证明，使用范数的乘积可以快速得到

那么这个函数是积性函数就完成了证明，同时不难发现质数处取值：

$$g(p^k)=\begin{cases}(2p+1)^k[p\equiv 1\mod 4]\\1 \ \ \ \ \ \ \ \ \ \ \ \ \ \ [p\equiv 3\mod 4]\end{cases}$$

剩下的部分不难使用 $min25$ 筛进行求解，但是一个需要分开讨论

在进行 $min25$ 筛的第一个 $\rm{DP}$ 的过程中，使用 $3\times 3\equiv 1$ 的优美性质来进行 $p\equiv 3\mod 4$ 的部分的转移

具体实现的时候分 $g_1(n,i),g_2(n,i)$ 两个部分进行转移，一开始转移的都是个数，并不计算具体点值

剩下的都交给第二个 $\rm{DP}$ 啦，细节并不多，复杂度 $\Theta(\frac{n^{0.75}}{\log n})$