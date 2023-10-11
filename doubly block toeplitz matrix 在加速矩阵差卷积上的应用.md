[文档链接](https://genn-team.github.io/posts/sw_blog_toeplitz.html)，原文 $16\times 9$ 的矩阵有一处写错了。

CNN 的卷积是执行了 $w'_ {i,j}=\sum\limits_{x,y}w_{i+x,j+y}\times C_{x,y}$，有人认为每次平移卷积核，运算量很大，又是乘法又是加法。

现在我们把 $w_{x,y}$ 展开形成一个 $[n\times m,1]$ 的向量 $V$，然后构造一个大小为 $[(n+1)\times (m+1),n\times m]$ 矩阵 $P$ 使得 $X=PV$ 得到的 $X$ 压缩成 $[n+1,m+1]$ 矩阵之后取右下部分能得到正确的 $w'$

~~虽然我感觉变成矩阵乘向量之后运算更多了啊。~~

这个矩阵 $P$ 可以通过把 teoplitz 矩阵拼起来得到（或者说叫做 doubly block matrix，那么先给出一个 teoplitz 矩阵的示例：

$$\left(
    \matrix{ 
        a_{0} & a_{-1} & a_{-2} & \cdots  & \cdots & \cdots & a_{-(N-1)} \cr
        a_{1} & a_{0} & a_{-1} & a_{-2} &  & & \vdots \cr
        a_{2} & a_{1} & a_{0} & a_{-1} &  & & \vdots \cr
        \vdots & \ddots & \ddots & \ddots & \ddots & \ddots & & \vdots \cr
        \vdots & & & \ddots  & a_{0} & a_{-1} & a_{-2} \cr
        \vdots & & &  & a_{1} & a_{0} & a_{-1} \cr
        a_{M-1} & \cdots  & \cdots & \cdots & a_{2} & a_{1} & a_{0} }
    \right)
$$

不难发现：$a_{i,j}=V_{i-j}$，所以说该矩阵也可以被压缩成一个向量。上面提到了，doubly block teoplitz 矩阵是由若干块 teoplitz 矩阵拼起来的，即现有若干大小相同的 teoplitz matrix $A_{-(M-1)},\dots A_{N-1}$，那么一个 doubly block teoplitz 矩阵可以如下：

$$\left(
    \matrix{ 
        A_{0} & A_{-1} & A_{-2} & \cdots  & \cdots & \cdots & A_{-(N-1)} \cr
        A_{1} & A_{0} & A_{-1} & A_{-2} &  & & \vdots \cr
        A_{2} & A_{1} & A_{0} & A_{-1} &  & & \vdots \cr
        \vdots & \ddots & \ddots & \ddots & \ddots & \ddots & & \vdots \cr
        \vdots & & & \ddots  & A_{0} & A_{-1} & A_{-2} \cr
        \vdots & & &  & A_{1} & A_{0} & A_{-1} \cr
        A_{M-1} & \cdots  & \cdots & \cdots & A_{2} & A_{1} & A_{0} }
    \right) .
$$

注意无论是 teoplitz 矩阵还是拼起来的新矩阵，我们都不要求它是方阵，或者说，只要满足 $a_{i,j}=V_{i-j}$ 即可

记卷积核为$\left(\begin{matrix}k_{1,1}&k_{1,2}\\k_{2,1} &k_{2,2}\end{matrix}\right)$ ，我们将卷积核展开成一个大小 $[n+1,m+1]$ 的矩阵 $K=\left(\begin{matrix}k_{2,2}&k_{2,1}&0&0\\k_{1,2}&k_{1,1}&0&0\\0&0&0&0\\0&0&0&0\end{matrix}\right)$，左上角把卷积核中心对称了，剩下的全是 $0$

取出它的所有列向量，展开成一个 teoplitz 矩阵：

$$F_0=\left(\matrix{ k_{22} & 0 & 0 \cr
        k_{21} & k_{22} & 0 \cr
        0 & k_{21} & k_{22} \cr
        0 & 0 & k_{21}}
    \right)
    \quad
	F_1=
    \left(
    \matrix{ 
        k_{12} & 0 &  0 \cr
        k_{11} & k_{12} & 0 \cr
        0 &  k_{11} & k_{12} \cr
        0 &  0 &  k_{11}}
    \right)\quad
F_2=F_3=
    \left(
    \matrix{ 
        0 & 0  & 0 \cr
        0 & 0 & 0 \cr
        0  & 0 & 0 \cr
        0  & 0  & 0}
    \right)
    $$

然后把这些碎片拼成：

$$F=\left(
    \matrix{ 
        F_0 & F_3 & F_2 \cr
        F_1 & F_0 & F_3 \cr
        F_2 & F_1 & F_0 \cr
        F_3 & F_2 & F_1
    }
    \right)$$
第一列是依次 $F_0\dots F_3$，右上角你可以理解成把矩阵接到下面再让对角线延申得到的。

不妨假设你现在的二位图片大小为 $3\times 3$，把它展开成一个大小为 $[9,1]$ 的向量，于是我们通过

$$\left(\matrix{
k_{22} & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 \cr
k_{21} & k_{22} & 0 & 0 & 0 & 0 & 0 & 0 & 0 \cr
0 & k_{21} & k_{22} & 0 & 0 & 0 & 0 & 0 & 0 \cr
0 & 0 & k_{21} & 0 & 0 & 0 & 0 & 0 & 0 \cr
k_{12} & 0 & 0 & k_{22} & 0 & 0 & 0 & 0 & 0 \cr
k_{11} & k_{12} & 0 & k_{21} & k_{22} & 0 & 0 & 0 & 0 \cr
0 & k_{11} & k_{12} & 0 & k_{21} & k_{22} & 0 & 0 & 0 \cr
0 & 0 & k_{11} & 0 & 0 & k_{21} & 0 & 0 & 0 \cr
0 & 0 & 0 & k_{12} & 0 & 0 & k_{22} & 0 & 0 \cr
0 & 0 & 0 & k_{11} & k_{12} & 0 & k_{21} & k_{22} & 0 \cr
0 & 0 & 0 & 0 & k_{11} & k_{12} & 0 & k_{21} & k_{22} \cr
0 & 0 & 0 & 0 & 0 & k_{11} & 0 & 0 & k_{21} \cr
0 & 0 & 0 & 0 & 0 & 0 & k_{12} & 0 & 0 \cr
0 & 0 & 0 & 0 & 0 & 0 & k_{11} & k_{12} & 0 \cr
0 & 0 & 0 & 0 & 0 & 0 & 0 & k_{11} & k_{12} \cr
0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & k_{11} }\right)
\cdot
\left(\matrix{
i_{11} \cr
i_{12} \cr
i_{13} \cr
i_{21} \cr
i_{22} \cr
i_{23} \cr
i_{31} \cr
i_{32} \cr
i_{33}} 
\right)$$

得到卷积结果，新向量大小是 $[16,1]$，理论上是取右下角。