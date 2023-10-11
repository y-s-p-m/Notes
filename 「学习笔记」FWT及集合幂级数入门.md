# FWT

相比与 $\rm NTT/FFT$ 中的加法卷积，这里支持了位运算卷积，原理基于集合幂级数研究

vfk 的论文阅读没有任何门槛，所以就不抄一遍了

<details>
<summary>Code Display</summary>

```cpp
inline void And(int *f,int lim,int opt){
    for(int p=2;p<=lim;p<<=1){
        int len=p>>1; for(int k=0;k<lim;k+=p){
            for(int l=k;l<k+len;++l){
                if(opt==1) f[l]=add(f[l],f[l+len]); 
                else f[l]=del(f[l],f[l+len]);
            }
        }
    }return ;
}
inline void Or(int *f,int lim,int opt){
    for(int p=2;p<=lim;p<<=1){
        int len=p>>1; for(int k=0;k<lim;k+=p){
            for(int l=k;l<k+len;++l){
                if(opt==1) f[l+len]=add(f[l+len],f[l]); 
                else f[l+len]=del(f[l+len],f[l]);
            }
        } 
    }return ;
}
inline void Xor(int *f,int lim,int opt){
    for(int p=2;p<=lim;p<<=1){
        int len=p>>1; for(int k=0;k<lim;k+=p){
            for(int l=k;l<k+len;++l){
                f[l]=add(f[l],f[l+len]); 
                f[l+len]=del(f[l],add(f[l+len],f[l+len])); 
                if(opt==-1){
                    f[l]=mul(f[l],i2);
                    f[l+len]=mul(i2,f[l+len]);    
                }
            }
        }
    }return ;
}
```

</details>
	

# 子集卷积

这个和普通的 $\rm FWT$ 的不同之处是中间要对集合的大小相加

那么进行 $\rm FWT$ 之后中间对位数做加法卷积再 $\rm IFWT$ 即可

<details>
<summary>Code Display</summary>

```cpp
struct node{
    int a[25];
    void operator+=(const node &p){
        for(int i=0;i<=n;++i) a[i]=add(a[i],p.a[i]); 
        return ;
    }
    void operator-=(const node &p){
        for(int i=0;i<=n;++i) a[i]=del(a[i],p.a[i]); 
        return ;
    }
    node operator*(const node &p)const{
        node ans; for(int i=0;i<25;++i) ans.a[i]=0; 
        for(int i=0;i<=n;++i) 
            for(int j=0;i+j<=n;++j)
                ans.a[i+j]=add(ans.a[i+j],mul(a[i],p.a[j])); 
        return ans;
    }
}F[N],G[N];
inline void FWT(node *f,int lim,int opt){
    for(int p=2;p<=lim;p<<=1){
        int len=p>>1; 
        for(int k=0;k<lim;k+=p) 
            for(int l=k;l<k+len;++l) 
                if(opt==1) f[len+l]+=f[l]; 
                else f[l+len]-=f[l];
    } return ;
}
signed main(){
    n=read(); int lim=(1<<n); 
    for(int i=1;i<lim;++i) bit[i]=bit[i>>1]+(i&1); 
    for(int i=0;i<lim;++i) F[i].a[bit[i]]=read();
    for(int i=0;i<lim;++i) G[i].a[bit[i]]=read();
    FWT(F,lim,1); FWT(G,lim,1); 
    for(int i=0;i<lim;++i) F[i]=F[i]*G[i]; FWT(F,lim,-1);
    for(int i=0;i<lim;++i) printf("%lld ",F[i].a[bit[i]]); puts("");
    return 0;
}
```

</details>

# 集合幂级数

将形式幂级数中的 $x$ 换成了 $n$ 维 $0/1$ 向量 $\rm x$ ，具体建议阅读 2015 年 vfk 的论文

根据 FWT/FMT 的原理该运算支持除法操作，只要系数没有 $0$ 且存在逆元即可，同时 FWT 本身也需要在满足运算存在对 2 逆元的局面下进行

而当不存在逆元的特殊情况可以看看 AGC034F 的做法，博主不清楚是不是有扩展性

---

求解 $\ln,\exp$ 的手段比较抽象，先像子集卷积把元素分类，之后做 $\rm FMT$

接下来变成了若干形式幂级数所以直接求，最后 $\rm IFMT$ 回去

因为不懂怎么分开 $\rm FMT$ 所以就鸽掉了

# 例题

## THUPC2019 找树

**其实 FWT 本质就是个求点值，转插值**

先把题目转化为是判定每个 $i$ 是否存在方案的计数问题，生成树问题启发我们使用矩阵树定理，位运算对应 $FWT$

那么对于每个加入的边，我们在 $v$ 为第三维的数组中构造基尔霍夫矩阵，然后对每个 $(x,y)$ 为第一第二维的数组做 $\rm FWT$（因为边可能有重的）

考虑 $DWT$ 就是把多项式转成点值，这里值得注意的是每个位上的运算就对应使用 $\rm And/Or-FMT$ 或者 $\rm Xor-FWT$

再反过来对每个权值求行列式的值，最后 $\rm IFWT$ 回去得到存在性

其实就是求点值和做插值  

复杂度 $\Theta(n^3 2^w)$，代码不难写

实现细节：行列式求值的时候模个大点的质数，因为我们只关心存在性

这题启发我们 $\rm FWT$ 也可以转化成点插值来理解，对于不同的运算，转成点值了也就有了独立性

同时 $\rm FWT$ 完全可以理解成多个生成函数的卷积，这样其实也可以推得模板的正确性

## SNOI2017 遗失的答案

分解 $L,G$ 质因数和次幂

一个合法的方案必然满足每个数次幂的 $\min=G_p$，最大的一个次幂是 $L_p$

观察到这个质因子个数很少，那么两位式状压：第一位表示是不是满足下界，第二位表示上界

这个状压很不套路，做前缀、后缀各一次 $dp$ 得到信息

那么每个 $L$ 的因数对其前后缀的信息做 $\rm And-FWT$ 取全集得到答案

实现的时候最后的每个因子要乘 $gcd$

## Codeforces449D

题目本质是做一个背包然后求 $f[0]$ 

这个式子是个 $And$ 卷积，只会 $\rm FWT$，当然可以得到一个每次暴力卷的做法，但是复杂度很大

考虑这种 $dp$ 的本质是某个物品如果有 $i$ 个的话那么这个点的超集就会翻番（这和 $FWT$ 的意思是一样的）

那么可以整体统计数之后 $\rm FWT$ 一次即可，在 $\rm IFWT$ 之前把每个 $A[i]$ 替换成 $2^{A_i}$

思考本质就能发现这个并不需要每次快速幂

统计答案记得减掉空集的情况，同时 $\rm FWT$ 的模数应为 $1e9+6$

## uoj310 黎明前的巧克力

朴素的 $dp$ 式子为 $f[i][j]=f[i-1][j]+2\times f[i-1][j\oplus a_i]$，使用 $\rm FWT$ 优化

但不一样的是这个题目中的系数变成了 $-1/3$

那么对于 $FWT$ 出来的数组，解一元一次方程得到 $-1$ 的个数 $k$ 和 $3$ 的个数 $n-k$（也就是必然会产生贡献）

那么用次幂替换掉之后 $\rm IFWT$ 即可

## HAOI2015 按位或

需要 $Min-Max$ 容斥的科技：

$$\min(S)=\sum\limits_{T\subset S} (-1)^{|T|+1}\max(T)$$

同时可以反过来写：

$$\max(S)=\sum\limits_{T\subset S} (-1)^{|T|+1}\min(T)$$

证明主要考虑分 $|T|$ 的奇偶性把多余的都消掉剩下 $T=\{\max(S)\}$

这题主要是考虑了应用到期望上的式子

$${E}(\max(S))=\sum\limits_{T\subset S} (-1)^{|T|+1} E(\min(T))$$

最终求的就是 $E(U)$ 那么现在没有的就是 $E(\min(T))$ 

用封闭形式等等大力推推式子，得到 $E(\min(T))=\frac{1}{1-Sum(E(U\oplus T))}$

剩下的就是子集和，$\rm FWT$ 即可

## PKUWC2018 随机游走

首先 $\min-\max$ 容斥转化为求集合最小值

这部分推导确实是新做法：设成两者相关的一次函数

$$f(S,x)=\frac{\sum\limits_{(x,y)\in E} f(y)}{d_i}+1$$

即求到了 $S$ 集合中的点的最小步数，把父亲的提出来，设 $f[t]=k_t\times f[x]+b_t$

推出来式子为 $k_x=\frac1{d_x-sumk_x},b_x=\frac{d_x+sumb_x}{d_x-sumk_x}$，均与父亲节点的信息无关

那么对于每种取值集合都 $dfs$ 一下得到 $b[rt]$ 

接下来就是 $\rm FWT$ 求子集和了


更多集合幂级数的题目参考 [https://www.cnblogs.com/lhm-/p/14287558.html](https://www.cnblogs.com/lhm-/p/14287558.html)