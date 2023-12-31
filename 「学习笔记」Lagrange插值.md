# Lagrange 插值

有 $\texttt{Lagrange}$ 公式：

$$f(x)=\sum_{i=1}^n y_i \prod_{i\neq j} \frac{x-x_j}{x_i-x_j}$$

如果给定了点值直接逆做就行了

貌似和 $\rm IDFT$ 有类似的地方，但是也显然是不一样的（点值的位置是不同的，$FFT$ 利用了单位根的性质）

好像可以快速插值然后单点求值

时间复杂度 $O(n^2)$ 

## 前后缀优化 Lagrange 插值

如果点值连续，可以预处理要求的 $f(k)$ 的 $\prod k-i$

那么求一个 $\prod \frac{k-x_i}{x_i-x_j}$ 就可以阶乘逆元做了

这个是最常见的做法，貌似下面的例题均用此做法来进行了时间复杂度的优化

## 重心 Lagrange 插值

推一下式子考虑增量即可

应用是自然数幂前缀和，因为伯努利数原理，那么用第二个插值方法就行了 


# 例题

## 「JLOI2016」 成绩比较

先钦定哪些人比他高，最后给答案乘 $\binom {n-1}k$

考虑名次的限制，这里会计漏或者重复，那么容斥，也就是 

$$\sum_{i=0}^{n-k} (-1)^{n-k-i}\binom {n-k} i \prod_{k=1}^m \binom {i}{R_i} $$

最后是分数的部分，列式展开发现需要对自然数求幂和，插值即可

$$\prod_{i=1}^m\sum_{x=1}^{U_i} (U-x)^{R_i-1}x^{n-R_i}$$

## 「集训队互测2012」calc

朴素的 $dp$ 是 $trivial$ 的：$f_{i,j}=f_{i-1,j-1}\times j+f_{i-1,j} $

假设 $f_{i,j}$ 为 $g(i)$ 次多项式，正向可以证明 $g(i)=2i$ 

所以求出来一些点值之后插 $k$ 处的就行了

也可以使用生成函数科技：

考虑答案显然为 $\prod_{i=1}^k  (1+ix)$

按照套路，乘法取 $\ln$ 转加法

得到的式子为：

$$exp(\sum_{i=1}^{\infty}\frac{(-1)^{i+1}\sum i^k}{i!}x^i)$$

换换顺序得到 $Tyler$ 展开的式子，同时还可以等比数列求和，那么需要几次 $exp$ 就好了

## 「Codeforces 995F」Cowmpany Cowmpensation 

树上的上面一题，值得指出的是，这个多项式是 $n$ 次的

貌似没有生成函数做法

## 「NOI2019」机器人

本题并不属于简单题

考虑一个非常 $trivial$ 的 $dp$：设 $f_{i,j,k}$ 表示 $[i,j]$ 的最大值为 $k$ 的时候的答案，那么答案为 $f_{1,n,mx}$

转移考虑找区间里面满足 $|(x-i)-(x-j)|\le 2$ 的 $x$ 

其实不难观察得到 $x\in \{mid,mid+1,mid-1\}$（分区间长度讨论即可）

所以发现这个 $f_{l,r}$ 的状态大概在 $kn\log n$ 左右，跑一个记忆化搜索发现是 $2000$ 到 $3000$

因为是插值找的这题目，所以考虑 $f_{l,r,k}$ 会不会是关于 $k$ 的多项式，按套路，可能是 $r-l+2$ 次的

考虑对于 $l=r$ ，不为 $0$ 的 $k$ 是常数，那么按照原来 $dp$ 式子的转移做两边乘法，所以次数增加

（以上证明显然并不严谨，具体证明留坑待填）

所以考虑按照值域分段来进行 $dp$ 的转移，对于当前值域 $[v[i],v[i+1]-1]$ 可以仅取区间长度个点值来转移，然后就可以插出来 $dp_{l,r,v[i+1]-1}$

以上思考是 $trivial$ 的，具体实现的时候，$dp$ 暴力进行即可，使用记忆化搜索，每次清空 $n+2$ 位

以上做法是相对好写（好猜）但是速度非常之慢，本地跑极限数据要 $6s$ 左右（但是交上去卡了好久过了），根据赛时反馈也确实只能拿 95 分

貌似 $\texttt{mayaohua2003}$ 的游记里面提到了一个下降幂的做法，因为我完全不熟悉下降幂的科技，所以等学习了之后再补

不可否认，这目前是（估计也将会是）这篇博客里面最高明的题目
