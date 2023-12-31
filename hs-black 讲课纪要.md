hs-black 好哥哥讲课的时候是分成很多类别分享的，但是作为蒟蒻的我就没那么有条理了，乱放一汽就算完事

多数为口胡，但是细节也处理得比较明白了

# HDU6566

设 $f_{x,j,0/1}$ 表示在当前节点 $x$ 的子树种选择了容量为 $j$ 的物品，当前点选/不选的最大收益以及方案数，但是直接书上背包复杂度 $\Theta(nm^2)$ 不能接受

这个 $\rm DP$ 挪到 $\rm dfs$ 序上面做，那么需要将这个 $\rm DP$ 添加维度来表示独立集，不过乍一看需要记录根链上所有点的是否选择的信息，因为从 $\rm dfs$ 序上第 $i$ 个点跳到第 $i+1$ 个点似乎是没有什么规律的

将重链剖分的过程改一改，**先遍历轻儿子，再遍历重儿子**，那么 $\rm dfs$ 序上相邻且没有父子关系的两个点的 $\rm LCA$ 一定是一个重链顶的父亲，所以把根链上所有重链顶的父亲是否出现过压到状态里即可

根据 $2^{\log 2n}=n$ 可以发现这维度降到了 $\Theta(n)$ ，再进行背包复杂度就是 $\Theta(n^2m)$ ，实现的时候按照深度给需要记录状态的节点分配二进制位就没什么障碍了

# CS Academy Expected Tree Degrees

题意简述：$n(n\le 10^6)$ 个点的树，$[2,n]$ 的父亲从编号小于其的点种随便选，求每个点度数平方和期望

平方和直接拆成选两个边，有公共端点期望，同边贡献为 $2(n-1)$，剩下的对于 $j<i$ 的点对 $(i,j)$，$i$ 的父亲是 $j$ 或者 $j$ 的父亲才有贡献

由于 $j$ 的父亲是谁带来的贡献相同，都是 $\frac{2}{i-1}$ ，而这里由于选择的是有序数对所以这部分得数要 $\times2$

总的来说：$ans=2(n-1)+2\sum_{i=3}^n\sum_{j=2}^{i-1}\frac2{i-1}$，是不是要把 $n$ 开到 $10^{14}$ 啦！

# [Unknown Source] String Problem Strikes Back

题意简述：设 $f(Str)=Str$ 的最大 $\rm Border$，给定字符串 $S$，求 $\sum_{1\le i<j\le n}f(S[i\dots j])$，其中 $S$ 为给定的纯随机 $0/1$ 字符串，$n\le 10^6$

随机所以长度 $\ge \log n$ 的串重复出现的概率就很小，所以对每个长度 $\le 3\log n$ 的 $S$ 的子串找是不是存在和其相同的串

由于 $\rm Border$ 存在嵌套关系，所以每个串 $S$ 实际贡献的是 出现次数 $\times (len_S-len_{max\rm S-Border})$

这个计算公式好像是唯一正确的，而通过研究某一 $\rm Border$ 通过挪右端点得到的计算方式是完全没有道理的

# CF1286E

考虑维护所有 $\rm Border$ 集合，新添字符之后找到集合里面和后继和该字符不一样的删掉即可，这部分可以通过记录 $i$ 在 $\rm next$ 树上第一个后继和 $s_{i+1}$ 的祖先来做到 $\Theta(n)$

使用 `std::map` 统计每种 $w$ 对应的 $\rm border$ 的数量，对于删除 $\rm border$ ，求出该位置对应长度的后缀最小值即可，这可以使用可内部二分的单调栈实现

对于当前 $w$ 带来的更新，直接在 `std::map` 里面把 $>w_i$ 的元素删去，元素个数加到当前 $w$ 上即可，这也可以支持计算当前位置为 $\rm border$ 权值之和的改变量，做前缀和得到真正答案

# Luogu6326

钦定某个点被选中就以该节点为根得到 $\rm dfs$ 序，设 $f_{i,c}$ 表示考虑了 $\rm dfs$ 序上 $[i,n]$ 的节点，总代价为 $c$ 的方案数

如果当前点不选的话，子树里面的点一个也不能选，所以需要找到 $\rm dfs$ 序在子树外的第一个位置转移；如果选择当前点就从 $f_{i+1}$ 来转移，也不用考虑 $\rm dfs$ 序上在第 $i+1$ 的位置的元素是不是在自己子树里面

多重背包需要二进制分组甚至单调队列优化

对于不要求 $x$ 选中的情况可以点分治递归处理

# [Unknown Source] 字符串の小题

题意简述：将一个长度为 $n$ 的字符串分成 $k$ 段，最小化每段中字典序最大的子串的字典序。$n\le 10^6$

答案是字符串的子串，所以进行二分答案的时候可以在随机一个在当前二分范围内的字符串当做 $mid$，由于是随机，所以期望就是中位元素

此时问题本质上就是求 $mid$ 和每个前缀的 $\rm LCP$ 来判定分段，如果在贪心意义下能分成 $\le K$ 段那么就合法

所谓贪心意义就是记录下来当前区间里面最大能承受的右端点是多少，超过之则需要再分段

`check` 完之后需要维护二分范围，根据实际含义操作即可

# [Unknown Source] 创造aba

倒序做一个背包求出来 $l_{i,j}$ 表示前 $i$ 个元素得到长度为 $j$ 的字符串，后面元素任选的情况下有无可能拼成长度为 $n$ 的字符串

注意到一种暴力 $\rm DP$ 的做法是设 $f_{i,j}$ 表示前 $i$ 个串选出长度为 $j$ 的字符串的最小字典序

此时若有 $l_{i,j}=l_{i,k}=1(j<k)$ 则如果 $f_{i,j},f_{i,k}$ 都想作为最终答案的一部分，则$f_{i,j}$ 是 $f_{i,k}$ 的前缀

所以可以维护最长的 $f_{i,m}$ 和以及它的哪些前缀是可能作为 $f_{i,k}$ 出现的，显然这里需要满足对应的 $l_{i,k}=1$

在添加一个新字符串 $s$ 的时候需要让它和每个满足 $l_{i+1,k+|S|}=1$ 的 $f_{i,k}$ 拼起来和 $f_{i,m}$ 比字典序，写一个 $\rm exkmp$ 即可支持

仔细想想，如果要维护 $f_{i+1,m}$ ，新添加的 $s$ 还需要和自己的每个后缀比大小，也是 $\rm exkmp$ 射程范围之内的东西了

# SDOI2015 苹果树

将树画到平面图上，每个点的儿子节点按照遍历顺序从左往右写，再观察 $dfs$ 的过程：当遍历到某个点时，这个点的根链将树分成了两个部分

设 $f_{x,i}$ 表示正向遍历邻接表式 $\rm DFS$ 搜到了 $x$ 且 $x$ 根链上每个点最多取了 $a_{y}-1$ 个物品 的最大幸福感之和，$g_{x,i}$ 表示反向遍历邻接表式 $\rm DFS$ 搜到 $x$ 且 $x$ 的根链上每个点都不选的最大幸福感之和

根据这样的状态定义，在 **叶子节点** 选择根链免容量更新答案即可

这样的状态定义也就导致了本题转移相对复杂：

对于 $f$ 而言，在 $\rm DFS$ 的过程中先将把自己的 $a_x-1$ 个元素做多重背包，将 $\rm DP$ 数组放到每个儿子处递归

而在递归 $x$ 的祖先的其它儿子时 $x$ 是可以选满 $a_x$ 个元素的，在回溯时再用剩余的 $1$ 个元素配合 $f_{x,*}$ 更新父亲 $\rm DP$ 值即可

这样的转移方式也满足了选儿子则至少选一个父亲，因为回溯时 $\rm DP$ 数组发生了变化

对于 $g$ 而言，$\rm DFS$ 的过程则是将所有儿子递归完成后再加入 $a_x-1$ 个元素，回溯时也同样用剩下的 $1$ 个元素更新

本题必须使用单调队列优化背包，这个技术就是将容量按照模 当前物品体积 分类，窗口的左端点由选择的物品数量决定

# CFGym102341 G*

使用 $\rm SG$ 值来刻画这个过程，不难发现 `II_` 和 `_II` 是完全相同的，可以视作共同一个，但是此时加上 `III,I_I,_I_` 还是有 $4$ 种

但是由于 `_I_,I_I` 是不能继续分割的，那么分成了两个子游戏，可以将 $\rm SG$ 值异或起来

不过题目种仍然有一个上下两个不能同时为 `_I_` 的限制，所以需要将状态添加至 $f_{S,0/1,0/1}$ 三个维度，$S$ 中二进制位下 $0$ 表示 `_II` 而 $1$ 表示 `III`，后两维表示在合并过程中最上方和最下方从未变成过 `_I_` 和无所谓
  
每个询问用 `_I_,I_I` 划分开即可