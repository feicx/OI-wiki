> SAT 是适定性（ Satisfiability ）问题的简称 。一般形式为 k - 适定性问题，简称 k-SAT 。而当 $k>2$ 时该问题为 NP 完全的。所以我们之研究 $k=2$ 的情况。

## 定义

2-SAT ，简单的说就是给出 $n$ 个集合，每个集合有两个元素，已知若干个 $<a,b>$ ，表示 $a$ 与 $b$ 矛盾（其中 $a$ 与 $b$ 属于不同的集合）。然后从每个集合选择一个元素，判断能否一共选 $n$ 个两两不矛盾的元素。显然可能有多种选择方案，一般题中只需要求出一种即可。

## 现实意义

比如邀请人来吃喜酒，夫妻二人必须去一个，然而某些人之间有矛盾（比如 A 先生与 B 女士有矛盾，C 女士不想和 D 先生在一起），那么我们要确定能否避免来人之间没有矛盾，有时需要方案。这是一类生活中常见的问题。

## 常用解决方法

### Tarjan [SCC 缩点](/graph/scc)

算法考究在建图这点，我们举个例子来讲：

假设有 ${a1,a2}$ 和 ${b1,b2}$ 两对，已知 $a1$ 和 $b2$ 间有矛盾，于是为了方案自洽，由于两者中必须选一个，所以我们就要拉两条条有向边 $(a1,b1)$ 和 $(b2,a2)$ 表示选了 $a1$ 则必须选 $b1$ ，选了 $b2$ 则必须选 $a2$ 才能够自洽。

然后通过这样子建边我们跑一遍 Tarjan SCC 判断是否有一个集合中的两个元素在同一个 SCC 中，若有则输出不可能，否则输出方案。构造方案只需要把几个不矛盾的 SCC 拼起来就好了。

显然地, 时间复杂度是 $O(n+m)$ 的，即缩点的复杂度。

### 爆搜

就是沿着图上一条路径，如果一个点被选择了，那么这条路径以后的点都将被选择，那么，出现不可行的情况就是，存在一个集合中两者都被选择了。

那么，我们只需要枚举一下就可以了，数据不大，答案总是可以出来的。

#### 爆搜模板

下方代码来自刘汝佳的白书：

```cpp
//来源：白书第323页
struct Twosat {
  int n;
  vector<int> g[maxn * 2];
  bool mark[maxn * 2];
  int s[maxn * 2], c;
  bool dfs(int x) {
    if (mark[x ^ 1]) return false;
    if (mark[x]) return true;
    mark[x] = true;
    s[c++] = x;
    for (int i = 0; i < (int)g[x].size(); i++)
      if (!dfs(g[x][i])) return false;
    return true;
  }
  void init(int n) {
    this->n = n;
    for (int i = 0; i < n * 2; i++) g[i].clear();
    memset(mark, 0, sizeof(mark));
  }
  void add_clause(int x, int y) {  //这个函数随题意变化
    g[x].push_back(y ^ 1);         //选了x就必须选y^1
    g[y].push_back(x ^ 1);
  }
  bool solve() {
    for (int i = 0; i < n * 2; i += 2)
      if (!mark[i] && !mark[i + 1]) {
        c = 0;
        if (!dfs(i)) {
          while (c > 0) mark[s[--c]] = false;
          if (!dfs(i + 1)) return false;
        }
      }
    return true;
  }
};
```

## 例题

### **HDU3062 [Party](http://acm.hdu.edu.cn/showproblem.php?pid=3062)**

**题面**

有 n 对夫妻被邀请参加一个聚会，因为场地的问题，每对夫妻中只有 $1$ 人可以列席。在 $2n$ 个人中，某些人之间有着很大的矛盾（当然夫妻之间是没有矛盾的），有矛盾的 $2$ 个人是不会同时出现在聚会上的。有没有可能会有 $n$ 个人同时列席？

这是一道多校题，裸的 2-SAT 判断是否有方案，按照我们上面的分析，如果 $a1$ 中的丈夫和 $a2$ 中的妻子不合，我们就把 $a1$ 中的丈夫和 $a2$ 中的丈夫连边，把 $a2$ 中的妻子和 $a1$ 中的妻子连边，然后缩点染色判断即可。

```cpp
//作者：小黑AWM
#include <algorithm>
#include <cstdio>
#include <cstring>
#include <iostream>
#define maxn 2018
#define maxm 4000400
using namespace std;
int Index, instack[maxn], DFN[maxn], LOW[maxn];
int tot, color[maxn];
int numedge, head[maxn];
struct Edge {
  int nxt, to;
} edge[maxm];
int sta[maxn], top;
int n, m;
void add(int x, int y) {
  edge[++numedge].to = y;
  edge[numedge].nxt = head[x];
  head[x] = numedge;
}
void tarjan(int x) {  //缩点看不懂请移步强连通分量上面有一个链接可以点。
  sta[++top] = x;
  instack[x] = 1;
  DFN[x] = LOW[x] = ++Index;
  for (int i = head[x]; i; i = edge[i].nxt) {
    int v = edge[i].to;
    if (!DFN[v]) {
      tarjan(v);
      LOW[x] = min(LOW[x], LOW[v]);
    } else if (instack[v])
      LOW[x] = min(LOW[x], DFN[v]);
  }
  if (DFN[x] == LOW[x]) {
    tot++;
    do {
      color[sta[top]] = tot;  //染色
      instack[sta[top]] = 0;
    } while (sta[top--] != x);
  }
}
bool solve() {
  for (int i = 0; i < 2 * n; i++)
    if (!DFN[i]) tarjan(i);
  for (int i = 0; i < 2 * n; i += 2)
    if (color[i] == color[i + 1]) return 0;
  return 1;
}
void init() {
  top = 0;
  tot = 0;
  Index = 0;
  numedge = 0;
  memset(sta, 0, sizeof(sta));
  memset(DFN, 0, sizeof(DFN));
  memset(instack, 0, sizeof(instack));
  memset(LOW, 0, sizeof(LOW));
  memset(color, 0, sizeof(color));
  memset(head, 0, sizeof(head));
}
int main() {
  while (~scanf("%d%d", &n, &m)) {
    init();
    for (int i = 1; i <= m; i++) {
      int a1, a2, c1, c2;
      scanf("%d%d%d%d", &a1, &a2, &c1, &c2);  //自己做的时候别用cin会被卡
      add(2 * a1 + c1,
          2 * a2 + 1 - c2);  //我们将2i+1表示为第i对中的，2i表示为妻子。
      add(2 * a2 + c2, 2 * a1 + 1 - c1);
    }
    if (solve())
      printf("YES\n");
    else
      printf("NO\n");
  }
  return 0;
}
```
### NOI2017 Day2 T1 游戏

**题面**

小 L 计划进行$n$场游戏，每场游戏使用一张地图，小 L 会选择一辆车在该地图上完成游戏。

小 L 的赛车有三辆，分别用大写字母A、B、C表示。地图一共有四种，分别用小写字母x、a、b、c表示。其中，赛车A不适合在地图a上使用，赛车B不适合在地图b上使用，赛车C不适合在地图c上使用，而地图x则适合所有赛车参加。适合所有赛车参加的地图并不多见，最多只会有d张。

$n$场游戏的地图可以用一个小写字母组成的字符串描述。例如：S=xaabxcbc表示小 L 计划进行8场游戏，其中第1场和第5场的地图类型是x，适合所有赛车，第2场和第3场的地图是a，不适合赛车A，第4场和第7场的地图是b，不适合赛车B，第6场和第8场的地图是c，不适合赛车C。

小 L 对游戏有一些特殊的要求，这些要求可以用四元组 $(i,h_i,j,h_j)$来描述，表示若在第iii场使用型号为$h_i$的车子，则第$j$场游戏要使用型号为$h_j$的车子。

你能帮小 L 选择每场游戏使用的赛车吗？如果有多种方案，输出任意一种方案。如果无解，输出 "-1"（不含双引号）。

**思路**

很显然a,b,c场地分别对应两种车，符合2−SAT的要求

这个是按A,BA,BA,B必须同时存在建边的，比如说$(i,h_i,j,h_j)$为$(1,c,5,a)$，则将1场地的$c$状态−>5场地的$a$状态

但是x场地对应三种车怎么办呢？

发现x场地的个数很少，最大才8个

所以考虑枚举x场地的状态，比如这次可以让x场地对应AB车（就成了c场地），下次对应BC车（就成了a场地），再下次对应AC车（就成了b场地）

这样复杂度是$O(3^{8}*(n+m))$，仍然过不了这道题

发现x对应AC车的情况在前面两个枚举中都出现了，也就是说前面两个枚举把所有情况都包括了

所以只用枚举前两个就行了，复杂度$O(2^{8}*(n+m))$

部分代码
``` cpp
bool look()//枚举x状态进行判断是否可行
{
    memset(h,0,sizeof(h));
    memset(dfn,0,sizeof(dfn));
    memset(low,0,sizeof(low));
    num=0,Col=0;
    for(int i=1;i<=m;++i)
      {
        if(S[s[i].i]==s[i].hi+32) continue;//S是场地字符串，s是给定4元组
        int tt=s[i].hi+32==pos[0][s[i].i]?s[i].i:s[i].i+n;//pos为每个点代表的两个状态
        if(S[s[i].j]==s[i].hj+32) add(tt,re[tt]);//re为对立点
        else
        {
            int ttt=s[i].hj+32==pos[0][s[i].j]?s[i].j:s[i].j+n;
            add(tt,ttt),add(re[ttt],re[tt]);
        }
      }
    for(int i=1;i<=2*n;++i)
      if(!dfn[i]) tarjan(i);
    for(int i=1;i<=n;++i)
      if(col[i]==col[i+n]) return 0;
    return print(),1;
}
```

注意：中间那里是判断$j$处是否能被选（这里能被选的意思是没有条件限制的$j$处场地适合开这个车）如果能被选，就按A,BA,BA,B必须同时存在连边；否则说明不能选这个四元组，就按AAA不能存在连边

## 练习题

HDU1814 [和平委员会](http://acm.hdu.edu.cn/showproblem.php?pid=1814)

POJ3683 [牧师忙碌日](http://poj.org/problem?id=3683)
