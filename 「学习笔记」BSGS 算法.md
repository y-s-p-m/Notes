# BSGS

求：满足一个最小的$x$，满足：

$$A^x\equiv B \mod \ p$$ 

其中 $A,B$ 给定，$p$ 是质数

## Solution

首先一个结论：我们取出$A^0\rightarrow A^{p-1}$ 必然有一个 $A^q\equiv B \mod p$

但是我们发现并不能这样，因为时间不允许，但可以进行如下操作

任意给个 $S$ 求出来：$B \times A^1\rightarrow B \times A^{S-1} \mod \ p $ 的值，放到一个 $hash$ 表中 

然后我们再求：$A^{0\times S} \rightarrow A^{\lfloor\frac{p}{S}\rfloor\times S}$

对于每个 $A^S$ 的次幂，我们移项，转化成下式：

$$A^{k \times S}\equiv {B \times {A^ p}}$$

然后我们把它扔到我们原来的 $hash$ 表里面找

如果找到了，答案就是 $k \times S-p$

复杂度是显然的 $\Theta(\sqrt {\bmod})$

实现用 `std::map` 实现会多 $\log$ 但是非常舒服 

```cpp
mp.clear(); int s=ceil(sqrt(p)),t=z%p; mp[t]=0;
for(int i=1;i<=s;++i) t=t*y%p,mp[t]=i;
int base=ksm(y,s,p),now=1; bool fl=0;
for(int i=1;i<=s;i++){
	now*=base; now%=p;
	if(!mp.count(now)) continue;
	int t=i*s-mp[now]; 
	printf("%lld\n",(t%p+p)%p); fl=1; break;
} 
```

# exBSGS

求：满足一个最小的$x$，满足：

$$A^x\equiv B \mod p$$ 

其中 $A,B$ 给定，不保证模数为质数

## Solution

可能在乘法的过程中把质因子凑全了，所以朴素的 BSGS 就施展不了拳脚了

特判 $B=0$，可以暴力：

枚举答案 $x$ 同时在每一步除掉 $gcd(A,p)$

如果什么时候 $p=1$，$x$ 为最小解，同时如果 $gcd(A,p)=1$ 那么无解

--- 

对于 $B\neq 0$ 的情况

$$A^{x-1}\times A \equiv B (mod\ p)$$

$$A^{x-1}\times A-kp=B$$

$k$ 是任意的一个整数

由裴蜀定理：如果 $B \neq 0 \mod gcd(A,p)$ 无解

否则我们把等式两边 除 $gcd(A,p)$ 则有：

$$\frac{A}{gcd(A,p)} \times A^{x-1} \equiv \frac B {gcd(A,p)} \mod \frac{p}{gcd(A,p)}$$

如果 $\frac{p}{gcd(A,p)}$ 不整除就继续除 $gcd$，整除之后就可以拿普通的 BSGS 做了

```cpp
inline void work(){
	y%=p; z%=p;
	if(z==1) return puts("0"),void();
	if(!y&&!z) return puts("1"),void();
	if(!y) return puts("No Solution"),void();
	if(!z){
		int res=0,d;
		while((d=__gcd(y,p))!=1){
			++res; p/=d;
			if(p==1) return printf("%lld\n",res),void();
		}return puts("No Solution"),void();
	}
	int c=1,res=0,d;
	while((d=__gcd(y,p))!=1){
		if(z%d) return puts("No Solution"),void();
		p/=d; z/=d; res++; (c*=y/d)%=p;
		if(c==z) return printf("%lld\n",res),void();
	}
	mp.clear(); int t=z%p,s=ceil(sqrt(p)); mp[t]=0;
	for(int i=1;i<=s;++i) (t*=y)%=p,mp[t]=i;
	int base=ksm(y,s,p),now=c;
	for(int i=1;i<=s;++i){
		(now*=base)%=p;
		if(!mp.count(now)) continue;
		return printf("%lld\n",i*s+res-mp[now]),void();
	} puts("No Solution"); 
	return ;
}
```

# 广义BSGS

形式：$f(f(f(f(x))))=k(mod\ p)$

狭义的普通 BSGS 对应就是 $f(x)=x\times A$

如果我们把 $f(x)$ 看成矩阵什么的也一样能做

我们发现其实在 BSGS 中存在一部求逆函数的操作，其实就是逆元

然后我们对于要求的$f(x)$，找到相应的逆函数 （例如逆矩阵，向量的逆） 即可

应用是求广义斐波那契数列的循环节

例题：BZOJ 4128

矩阵求逆之后上 广义BSGS 即可，会被卡常

然而我们可以直接偷懒换式子：

$$A^{kS}=B\times A^t(mod\ p)$$

所以我们把右边的所有结果存一存，左边就上矩阵乘就好了，并不用求逆

对于判定矩阵相等在哈希的射程范围之内