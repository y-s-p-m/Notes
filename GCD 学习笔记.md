今天的高代习题课讲解了非常有趣的知识，为什么 GCD 是对的呢？

我们从良序原理出发，展开这个世界的画卷：

> Well-ordering principle
>我们可以获得一个由自然数组成的集合的最小值

<details>
<summary>来看看良序定理在我们熟知的话题上是怎么应用的</summary>

----

如何使用 WOP 证明 $\sqrt 5$ 是 irrational number？设 $a^2=5b^2$，假设 $b=\min S,S=\{b|\exists a,\frac{a}b=\sqrt 5\}$，那么我们构造一个 $<b$ 的在 $S$ 中的对子即可。

已知 $a^2=5b^2\rightarrow a^2-2ab=5b^2-2ab\rightarrow \frac ab=\frac{5b-2a}{a-2b}$

$a-2b$ 明显是满足 $<b$ 的，所以产生了矛盾。

---

</details>



**良序定理的使用绝大多数要配合反证法**，比如我们熟知的第一第二类数学归纳法，就可以通过良序定理来得到其正确性。考虑第二类数学归纳法，将 $n=k$ 时命题不成立的 $k$ 取出，设 $t=\min k$，那么 $n=1\dots n=t-1$ 都是成立的，根据第二数学归纳法的定义可以得到矛盾。

通过良序定理，我们可以发现如下事实：

对于给定的正整数 $a,b$ 集合 $S=\{p|p=ax+by,p>0,x,y\in \mathbb{Z}\}$ 的最小值为 $gcd(a,b)$。

<details>
<summary>Proof</summary>

---

首先可以发现 $S$ 是非空自然数集，所以根据良序定理，它一定有最小值。假设最小值是 $c=x_0a+y_0b$。我们先尝试证明 $c|a$：

假设 $a=qc+r,0\le r<c$，如果 $r\neq 0$ ，那么我们发现  $r=a-qc=a-q(ax+by)=(1-qx_0) a- qy_0 b\in S$， 把这一串收尾拿出来 $r\in S,c=\min S,0\le r<c$，出现了矛盾。

类似的我们也可以证明在 $b$ 除以 $c$ 时余数也是 $0$。于是 $c|gcd(a,b)$

我们还需要证明 $gcd(a,b)|c$，这就自然多了，因为 $gcd(a,b)|a,gcd(a,b)|b$，$c$ 是 $a,b$ 的线性组合，当然有 $gcd(a,b)|c$

---

</details>

我们的 GCD 代码是怎么写的呢？

```cpp
inline int gcd(int n,int m){return m==0?n:gcd(m,n%m);}
```

将这个过程展开，把过程表述如下：

$$\begin{aligned}n&=q_1m+r_1,r_1< m\\m&=q_2 r_1+r_2,r_2< r1 \\r_1&=q_3 r_2+r_3,r_3<r_2\\ \dots \\r_{k-2}&=q_{k+1}r_{k-1}+r_k,r_k<r_{k-1}\\r_{k-1}&=q_{k+2} r_k+0\end{aligned}$$

在这个过程结束的时候，我们得到的结果是 $r_k=gcd(n,m)$ 

假设 $d=gcd(n,m)$，想证明 $d=r_k$，一种常见的方法是 $r_k|d,d|r_k$ 或者说 $r_k\le d,d\le r_k$

- 首先我们发现对于 $d=gcd(n,m)$ 一定满足 $d|n\leftarrow d| (q_1 m+r_1)\leftarrow d| r_1$。作类似的变换，可以推导出来 $d|n,d|m,d|r_1,d|r_2$。

	由此出发，我们可以发现 $d|r_t,d|r_{t+1}\leftarrow d| q_{t+2}r_{t+1}+r_{t+2}\leftarrow d|r_{t+2}$，最终得到 $d|r_k$
	
	想要使用良序定理也可以解决这部分，因为你可以把 $r_k$ 表示成 $Ar_{t}+Br_{t+1}=Cr_{t-1}+Dr_t$，可以理解为把 $r_{t-1}=q_{t+2}r_{t}+r_{t+1}$ 带入了，于是最后迭代到了 $r_k=pa+qb$，所以 $d|r_k$

- 注意到我们知道 $k|n,k|m\Leftrightarrow k|gcd(n,m)$ 
	
	根据上面的过程表达，我们发现 $r_k|r_{k-1},r_k|(r_{k}+q_{k+1}r_{k-1}=r_{k-2}),\dots r_k|m,r_k|n$
	
	这就引出了 $r_k|m,r_k|n$，于是 $r_k|d$，综上所述可以有 $d=r_k$

其实上面两个部分都是存在 $a|b,a|c,a|gcd(b,c)$ 的，留给读者证明这个了！其实就是使用良序定理。