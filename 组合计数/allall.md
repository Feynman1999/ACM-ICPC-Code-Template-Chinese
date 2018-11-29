[TOC]



## Lucas定理

Lucas定理解决的问题是组合数取模：
$$
\binom n m\mod p
$$
这里$n,m$可能很大，比如达到$1e15$，而p在$1e6$以内。显然运用常规的阶乘方法无法直接求解，所以引入Lucas定理。

 

把n和m写成**p进制数**的样子（如果长度不一样把短的补成长的那个的长度）： 
$$
n=(a_0a_1\dots a_k)_p \\
m=(b_0b_1\dots b_k)_p
$$
那么： 
$$
\binom n m\equiv \prod _{i=0}^k\binom {a_i} {b_i}\mod p
$$



#### 证明

从递归的角度看Lucas定理，是这样的：
$$
设n=ap+b,m=cp+d,(b,d<p,a=\lfloor\frac{n}{p}\rfloor,c=\lfloor \frac{m}{p}\rfloor) \\
\binom n m\equiv \binom a c*\binom b d  \quad  (mod\ p)
$$
可以通过二项式定理来说明上面的式子是成立的。

首先，对于任意质数$p$，有： 
$$
(1+x)^p\equiv 1+x^p\mod p
$$
**证明(5)：**  p is prime，%p意义下，由费马小定理$(1+x)^{p-1}\equiv1$，又$1+x\equiv1+x$  所以$(1+x)^{p}\equiv 1+x$

又$x^{p-1}\equiv1$  则$x^{p}\equiv x$  即$1+x^{p}\equiv 1+x$    **所以$(1+x)^{p}\equiv 1+x^p$    (mod p)



则%p意义下，二项式展开
$$
\begin{aligned}
(1+x)^n&=(1+x)^{\lfloor \frac{n}{p}\rfloor *p}(1+x)^b \\
&=(1+x^p)^{\lfloor \frac{n}{p}\rfloor}(1+x)^b \\
&=\sum _{i=0}\binom {\lfloor \frac{n}{p}\rfloor} ix^{pi}\sum _{j=0}\binom b jx^j
\end{aligned}
$$
考察等式左右两边$x^m$的系数，可以发现： 
$$
\begin{aligned}
左边&=\binom n m \\
右边&=\binom {\lfloor \frac{n}{p}\rfloor} i\binom b j,(pi+j=m,j<p) \\
&=\binom {\lfloor \frac{n}{p}\rfloor} {\lfloor \frac{m}{p}\rfloor} \binom b {m\%p}\\
&=\binom {a} {c} \binom b {d}
\end{aligned}
$$
证毕。

算法时间复杂度为$O(log_p^n)$



```c++
#include<bits/stdc++.h>
using namespace std;
typedef long long ll;
const int mod=10007;

ll inv[mod+10];
ll Fac[mod+10];
ll Fac_inv[mod+10];

void init()
{
    inv[1]=1;
    Fac[0]=1;
    Fac_inv[0]=1;
    for(int i=1;i<mod;++i){
        if(i!=1) inv[i]=((mod-mod/i)*(inv[mod%i]))%mod;
        Fac[i]=Fac[i-1]*i%mod;
        Fac_inv[i]=Fac_inv[i-1]*inv[i]%mod;
    }
}

ll C(int n,int m)
{
    if(m>n) return 0;
     return ((Fac[n]*Fac_inv[n-m]%mod)*Fac_inv[m])%mod;
}

ll lucas(int n,int m)
{
    if(m>n) return 0;
    ll ans=1;
    for(;m;n/=mod,m/=mod) ans=(ans*C(n%mod,m%mod))%mod;
    //cout<<ans<<endl;
    return ans;
}
```





## 扩展Lucas定理

若$p$不为质数

把$p$质因数分解，由于是多个不同素数的幂，所以可以直接运用**中国剩余定理**将答案合并。所以现在问题就转化成，$p$为素数，求： 
$$
\binom n m\mod p^q
$$
由于$\binom n m$=$\frac{n!}{m!(n-m)!}$ ，于是问题转为求解
$$
n! \% p^q        \quad  p \ is\  prime
$$

**值得注意的是这里  $p^q\approx 1e6 \quad n\approx 1e15$**

如何求解呢？

不妨带入具体例子体验一下，如$n=20,p=3,q=2$

$n!=1*2*3*4*5*6*7*8*9*10*11*12*13*14*15*16*17*18*19*20$

我们将其中$p=3$的倍数单独拿出来，并将3这个公因子提出可得：

$n!=3^6*(1*2*3*4*5*6)*(1*2*4*5*7*8*10*11*13*14*16*17*19*20)$

①这时观察前面一部分，又出现了新的阶乘，即$\lfloor \frac{n}{p}\rfloor!=\lfloor \frac{20}{3}\rfloor!=6!=(1*2*3*4*5*6)$ ，其前公因子$p=3$的指数为$\lfloor \frac{n}{p}\rfloor=6$

而对于新的阶乘，是当前问题的子问题，所以可以递归进行下去。

②观察后面一部分，即$(1*2*4*5*7*8*10*11*13*14*16*17*19*20)$

$1\equiv10(mod \ 3^2)$ 

$2\equiv11(mod \ 3^2)$

$4\equiv13(mod \ 3^2)$

............

$8\equiv17(mod \ 3^2)$

即相差了$p^q=3^2$ ，所以对于每个长度为$p^q-\frac{p^q}{p}$ 的连乘（**减去是因为有些数是$p$的倍数**），它们$\%p^q$同余，所以**只用计算其中一组即可**。而一组的数量最坏是$1e6$级别的，可行。

**对于最后没构成一组的**， 数量最坏也是$1e6$级别的，单独处理。如上面例子的$19*20$



问题：$m!$、$(n-m)!$与$p^q$不一定互质  所以可能没有逆元

这里的处理方法是 由于$\frac{n!}{m!(n-m)!}$一定能除尽

> 小注：计算$n!$的质因子p的指数$x$的公式为 

$$
f(n)=f(\lfloor{n\over p}\rfloor)+\lfloor{n\over p}\rfloor
$$

> 证明即为上面的①





#### 模板题

计算$C(n,m)\%p$               $m<n<1e18$                 $1< m <= 1e6$  

http://codeforces.com/gym/100633/problem/J

```c++
#include<bits/stdc++.h>
using namespace std;
typedef long long ll;

ll n,m,mod;// cal C(n,m)%mod

const int maxn=20;//同余方程个数
//在拓展卢卡斯中  模数一般1e6
//质因子数目很少

ll a[maxn];//每个方程的右值
ll M[maxn];//每个方程模的数
ll p[maxn];//底数
ll q[maxn];//质数
int num_equ;//方程数 即m的质因子数

//快速幂
ll fast_pow(ll a,ll b,ll mod)
{
    ll res=1;
    while(b)
    {
        if(b&1) res=a*res%mod;
        b>>=1;
        a=a*a%mod;
    }
    return res;
}

//拓展欧几里得
ll e_gcd(ll a,ll b,ll &x,ll &y)
{
    if(b==0){x=1;y=0;return a;}
    ll ans=e_gcd(b,a%b,x,y);
    ll temp=x; x=y; y=temp-a/b*y;
    return ans;
}

//%mod逆元
ll inv(ll a,ll mod)
{
    ll ans,tmp;
    e_gcd(a,mod,ans,tmp);
    return (ans+mod)%mod;
}

//x同余a  mod m
//a为方程右值 m为方程的模 n为方程数
//下标从0开始
ll CRT(ll a[],ll m[],int n){
    ll M=1;
    for(int i=0;i<n;++i) M*=m[i];
    ll ret=0;
    for(int i=0;i<n;++i){
        ll x,y;
        ll tm=M/m[i];
        ll tmp=e_gcd(tm,m[i],x,y);//tm 和 m[i]互质  只有一个解
        //cout<<x<<endl;
        ret=(ret+tm*x*a[i])%M;//这里x可能为负值  但不影响结果
    }
    return (ret+M)%M;
}

//质因子分解m
void solve_fac()
{
    int tp=mod;
    num_equ=0;
    for(int i=2;i<(int)(sqrt(tp)+1.0);++i){
        if(tp%i==0){
            //i是一个质因子
            p[num_equ]=i;
            int num=0;
            while(tp%i==0){
                num++;
                tp/=i;
            }
            q[num_equ++]=num;
        }
    }
    if(tp!=1){
        p[num_equ]=tp;
        q[num_equ++]=1;
    }
    //num_equ;方程数 即m的质因子数
    for(int i=0;i<num_equ;++i){
        M[i]=1; for(int j=0;j<q[i];++j) M[i]*=p[i];
    }
}

//求解n! % p^q
ll solve(ll n,int id)//id对应第几个质因子
{
    //为了保证和p^q互质 从而有逆元 我们提出来的公因子p是不考虑的
    if(n==0) return 1LL;
    ll res=1;
    ll num=n/M[id];//同余的组数
    if(num>0){//有完整的一组
        for(int i=1;i<=M[id];++i){
            if(i%p[id]) res=res*i%M[id];
        }
        res=fast_pow(res,num,M[id]);//num组
    }
    for(ll i=1;i<=n%M[id];++i){//不成组的单独处理
        if(i%p[id]) res=res*i%M[id];
    }
    //子问题递归即可
    return res*solve(n/p[id],id)%M[id];
}


ll C(ll n,ll m,int id)//id表示哪个质因子
{
    if(m>n) return 0;
    ll a=solve(n,id),b=solve(m,id),c=solve(n-m,id);
    ll k=0;
    //k为p的指数 即solve中没有考虑的部分
    for (ll i=n;i;i/=p[id]) k+=i/p[id];
    for (ll i=m;i;i/=p[id]) k-=i/p[id];
    for (ll i=n-m;i;i/=p[id]) k-=i/p[id];
    //cout<<"p M a b c k:   "<<p[id]<<" "<<M[id]<<" "<<a<<" "<<b<<" "<<c<<" "<<k<<endl;
    //k>=0是一定的  最后再乘上p^k
    return a*inv(b,M[id])%M[id]*inv(c,M[id])%M[id]*fast_pow(p[id],k,M[id])%M[id];
}

int main()
{
#ifndef ONLINE_JUDGE
    freopen("in.txt","r",stdin);
#endif
    ios::sync_with_stdio(false);
    cin>>n>>m>>mod;
    solve_fac();
    for(int i=0;i<num_equ;++i){
        a[i]=C(n,m,i);
    }
    cout<<CRT(a,M,num_equ)<<endl;
    return 0;
}
```



## 一些组合式的化简

![img](https://s1.ax2x.com/2018/08/23/5DhlTy.png)

## 卡特兰数

​	在一个平面直角坐标系中，只能往右或往上走一个单位长度，问有多少种不同的路径可以从左下角$(1,1)$走到右上角$(n,n)$，并且要求路径不能经过直线$y=x$上方的点，下图中的路径都是合法的。 

![450px-Catalan_number_4x4_grid_example.svg](http://blog.miskcoo.com/wp-content/uploads/2015/07/450px-Catalan_number_4x4_grid_example.svg_.png)



​	如果没有限制条件，那么从左下角走到右上角一共有 $2n $步，有$n$步是往右，另外$n$步是往上，那么路径方案数就是$2n$步中选择$n$步往右，一共有$C_{2n}^n$种方案。

​	考虑其中哪些方案是**不合法**的。

​	对于每一种不合法的方案，它的路径一定与$y=x+1$有交。我们找到它与$y=x+1$的第一个交点，然后将这个点后面部分的路径关于$y=x+1$做一个对称。那么这些路径原先到达的是$(n,n)$，现在到达$(n-1,n+1)$。且可以确定是，对称后的路径和对称前的路径是一一对应的，即方案数相同。而从$(1,1)$到$(n-1,n+1)$的路径有$C_{n-1+n+1}^{n-1}$种，即不合法的方案数，则**合法的方案数**为
$$
CATLAN_n=C_{2n}^n-C_{2n}^{n-1}=\frac{1}{n+1}C_{2n}^n
$$

这个式子是计算中比较常用的。



**注意卡特兰数是比较大的数** 


$cat[35]=\binom{70}{35}\div36$                           $3116285494907301262≈3.116285e+18  $

$cat[36]=\binom{72}{36}\div37$                           $11959798385860453492≈1.195980e+19  $

即n取36时，即超出long long int 的上界





## 一些卡特兰数的模型

#### 括号序列计数

有多少种不同的长度为$n$的括号序列？ 

一个括号序列是指$ (), ()(), (())() $这样的由括号组成的序列，并且没有左右括号无法匹配的情况。

可以将长度为$2n$的括号序列映射成上面的路径：首先**对于左括号，那么就向右走一个单位长度，对于右括号，那么就向上走一个单位长度**，由于括号序列合法，那么每次向右走的次数不会少于向上的次数，也就是这条路径不会在 $y=x$之上。再考虑每一条这样的路径，也能够对应到一种合法的括号序列，因此，长度为$2n$的括号序列的方案数就是$Catlan_{n}$



#### 出栈顺序

有$n$个元素和一个栈，每次可以将一个元素入栈，或者将栈顶元素弹出，问**有多少种可能的操作序列**，这可以将问题对应成括号序列，入栈为左括号，出栈为右括号，因此方案数也是$Catlan_{n}$



#### 排队问题

有 $2n$ 个人，他们身高互不相同，他们要成两排，每一排有 $n$ 个人，并且满足**每一排必须是从矮到高**，且后一排的人要比前一排对应的人要高，问有多少种排法?

我们考虑先把这些人从矮到高排成一排，现在来分配哪个人在前，哪个人在后，例如有 $6$ 个人，身高是 1, 2, 3, 4, 5, 6

我们用 1 表示这个人应该在后排，0 表示这个人应该在前排，例如说 100110 表示两排分别是 2, 3, 6 和 1, 4, 5 这是不合法的

由于后排要比前排对应的人高，也就是说 0 的出现次数在每一个地方都不应该小于 1的次数，又是一个括号序列，因此方案数也是$Catlan_{n}$



#### 二叉树计数

有多少种不同的 $n$ 个结点的二叉树 ?

![](http://blog.miskcoo.com/wp-content/uploads/2015/07/Catalan_number_binary_tree_example.png)



3个结点的二叉树共有5种，如上图（浅绿色为NULL）

朴素的想法是由于二叉树是递归定义的，可以考虑使用递推方法

用$ f_n$ 表示有 $n$ 个结点的二叉树的方案数（空树算一种，即 $f_0=1$），那么枚举子树大小可以得到方程
$$
f_n = \sum_{i = 0}^{n - 1} f_if_{n - i - 1}
$$
直接计算时间复杂度是$O(n^2)$

现在换个角度来想，对这棵二叉树进行遍历，并且考虑一个括号序列，当第一次遇到这个结点的时候，在括号序列末尾添加一个左括号，在从左子树回到这个结点的时候，在括号序列中添加一个右括号，这样，**将每一种不同的二叉树都对应到了一种不同的括号序列**，同样对于每一种不同的括号序列都可以找到对应的一种不同的二叉树，因此，有 n 个结点的二叉树的种类也是$Catlan_{n}$





## 用生成函数求解卡特兰数通项公式

观察上面的递推式
$$
f_n = \sum_{i = 0}^{n - 1} f_if_{n - i - 1}
$$
它的结果是$Catlan_{n}$，可以借助**生成函数**来证明。

定义序列 $Cat_n$ 的生成函数为 $C(z)$，再根据上面的递推式，可以列出方程 
$$
C(z) = zC^2(z) + 1
$$
解方程得
$$
C(z) = \frac{1 \pm \sqrt{1 - 4z}}{2z}
$$
因为$C(0)=C_0=1$，因此对上式求极限
$$
\begin{eqnarray*}
 \lim_{z \rightarrow 0^+} \frac{1 + \sqrt{1 - 4z}}{2z} &=& +\infty \\
\lim_{z \rightarrow 0^+} \frac{1 - \sqrt{1 - 4z}}{2z} &=& 1 
\end{eqnarray*}
$$
因此，取负号，$C(z) = \frac{1 - \sqrt{1 - 4z}}{2z}$

根据广义二项式定理展开$\sqrt{1-4z}$
$$
\sqrt{1-4z} = \left (1-4z \right )^{\frac{1}{2}} = \sum_{i=0}^{\infty}{\frac{1}{2} \choose i}(-4z)^i
$$
然后考虑第$n$项系数
$$
\begin{eqnarray*}
& &(-4)^n { \frac{1}{2} \choose n }\\
&=& (-4)^n \frac{\frac{1}{2}\cdot \left( \frac{1}{2} -1 \right)\cdot \left( \frac{1}{2} -2 \right)\cdots \left( \frac{1}{2} - n + 1 \right)}{1\cdot2\cdot3\cdots n}\\
&=& (-2)^n \frac{1\cdot (1-2)\cdot (1-4)\cdots (1 - 2n + 2)}{1\cdot2\cdot3\cdots n}\\
&=& (-2)^n \frac{1\cdot (1-2)\cdot (1-4)\cdots (1 - 2n + 2)}{1\cdot2\cdot3\cdots n}\\
&=& -\frac{1\cdot 3\cdot 5\cdots (2n - 3)}{1\cdot2\cdot3\cdots n}\cdot \frac{1\cdot 2\cdot 4\cdots 2n}{1\cdot 2 \cdot 3 \cdots n}\\
&=& -\frac{(2n)!}{(2n-1)(n!)^2}\\
&=& -\frac{1}{2n-1}{{2n} \choose n}
\end{eqnarray*}
$$
因此可以得到
$$
\begin{eqnarray*}
C(z) &=& \frac{1-\sqrt{1-4z}}{2z}\\
&=&\frac{1+\sum_{i=0}^{\infty}\frac{1}{2i-1}{{2i} \choose i} z^i}{2z}\\
&=&\frac{\sum_{i=1}^{\infty}\frac{1}{2i-1}{{2i} \choose i} z^{i-1}}{2}\\
&=&\sum_{i=0}^{\infty}\frac{2}{2i+1}{{2i+2} \choose {i+1}} z^i\\
&=&\sum_{i=0}^{\infty}\frac{1}{2}\cdot\frac{1}{2i+1}\cdot\frac{(2i+1)(2i+2)}{(i+1)^2}{{2i} \choose i} z^i\\
&=&\sum_{i=0}^{\infty}\frac{1}{n+1}{{2i} \choose i} z^i\\
\end{eqnarray*}
$$
证毕。同样得到了$C_n = \frac{1}{n+1}{{2n} \choose n}$ 

以上内容参考自*http://blog.miskcoo.com/2015/07/catalan-number* 





## 默慈金数

#### 圆上不相交弦

一个给定的数n的**默慈金数**是 **在一个圆上的n个点间，画出彼此不相交的弦的全部方法的总数，**

下图显示了“在一个圆上的4个点间，画出彼此不相交的弦的所有9种方法”：

![MotzkinChords4.svg](https://upload.wikimedia.org/wikipedia/commons/thumb/4/4a/MotzkinChords4.svg/300px-MotzkinChords4.svg.png)

 下图显示了“在一个圆上的5个点间，画出彼此不相交的弦的所有21种方法”： 

![MotzkinChords5.svg](https://upload.wikimedia.org/wikipedia/commons/thumb/c/ce/MotzkinChords5.svg/428px-MotzkinChords5.svg.png)





#### 路径计数(注意和卡特兰路径的区别)

在一个网格上，若限定每步只能向右移动一格（可以向右上、右下、横向向右，三种），并禁止移动到y=0以下的地方，则以这种走法用n步从（0,0）移动至（n,0）的可能形成的路径的总数为n的默慈金数。

以下为例，下例显现了从（0,0）至（4,0）照上述的走法中，九种可行的路径：

![Motzkin4.svg](https://upload.wikimedia.org/wikipedia/commons/thumb/b/b7/Motzkin4.svg/500px-Motzkin4.svg.png)

如何计数呢？

考虑向右横走对当前的高度没有影响，而去掉这种操作后，就是卡特兰路径，于是可以**枚举卡特兰路径**，即：

![{\displaystyle M_{n}=\sum _{k=0}^{\lfloor n/2\rfloor }{\binom {n}{2k}}C_{k}.}](https://wikimedia.org/api/rest_v1/media/math/render/svg/bb720a1ad038049569101610065cc75e4153f42a)  

其还有递归和递推式：

![{\displaystyle M_{n+1}=M_{n}+\sum _{i=0}^{n-1}M_{i}M_{n-1-i}={\frac {2n+3}{n+3}}M_{n}+{\frac {3n}{n+3}}M_{n-1}}](https://wikimedia.org/api/rest_v1/media/math/render/svg/67c15d15d75b84be531d300da75a75dad011179f) 

递推式的证明可参见<https://www.fq.math.ca/Scanned/40-1/woan.pdf> 

**若用卡特兰求解，**求解一个$n$的时间是$O(n)$ (需要预处理逆元)

若有多组询问 如$T=1e5$，则需要**递推式**预处理出默慈金数    

如[HDU - 5673 ](https://vjudge.net/problem/368986/origin)   若预处理则只需要几十ms即可跑完  虽然一般不会卡掉卡特兰数求解



默慈金数更多相关资料：

https://cs.uwaterloo.ca/journals/JIS/VOL3/SULANKE/sulanke.html
## 第一类斯特林数













## 第二类斯特林数





















## Berlekamp-Massey

**求解线性递推式**

* 时间复杂度$O(n^3logt)$    **n为矩阵大小  t为要求解的项数**



```c++
#include<bits/stdc++.h>
using namespace std;
#define rep(i,a,n) for (int i=a;i<n;i++)
#define per(i,a,n) for (int i=n-1;i>=a;i--)
#define pb push_back
#define mp make_pair
#define all(x) (x).begin(),(x).end()
#define fi first
#define se second
#define SZ(x) ((int)(x).size())
typedef vector<int> VI;
typedef long long ll;
typedef pair<int,int> PII;
const ll mod=1000000007;
ll powmod(ll a,ll b) {ll res=1;a%=mod; assert(b>=0); for(;b;b>>=1){if(b&1)res=res*a%mod;a=a*a%mod;}return res;}
// head

ll n;
namespace linear_seq {
    const int N=10010;
    ll res[N],base[N],_c[N],_md[N];

    vector<int> Md;
    void mul(ll *a,ll *b,int k) {
        rep(i,0,k+k) _c[i]=0;
        rep(i,0,k) if (a[i]) rep(j,0,k) _c[i+j]=(_c[i+j]+a[i]*b[j])%mod;
        for (int i=k+k-1;i>=k;i--) if (_c[i])
            rep(j,0,SZ(Md)) _c[i-k+Md[j]]=(_c[i-k+Md[j]]-_c[i]*_md[Md[j]])%mod;
        rep(i,0,k) a[i]=_c[i];
    }
    int solve(ll n,VI a,VI b) { // a 系数 b 初值 b[n+1]=a[0]*b[n]+...
//        printf("%d\n",SZ(b));
        ll ans=0,pnt=0;
        int k=SZ(a);
        assert(SZ(a)==SZ(b));
        rep(i,0,k) _md[k-1-i]=-a[i];_md[k]=1;
        Md.clear();
        rep(i,0,k) if (_md[i]!=0) Md.push_back(i);
        rep(i,0,k) res[i]=base[i]=0;
        res[0]=1;
        while ((1ll<<pnt)<=n) pnt++;
        for (int p=pnt;p>=0;p--) {
            mul(res,res,k);
            if ((n>>p)&1) {
                for (int i=k-1;i>=0;i--) res[i+1]=res[i];res[0]=0;
                rep(j,0,SZ(Md)) res[Md[j]]=(res[Md[j]]-res[k]*_md[Md[j]])%mod;
            }
        }
        rep(i,0,k) ans=(ans+res[i]*b[i])%mod;
        if (ans<0) ans+=mod;
        return ans;
    }
    VI BM(VI s) {
        VI C(1,1),B(1,1);
        int L=0,m=1,b=1;
        rep(n,0,SZ(s)) {
            ll d=0;
            rep(i,0,L+1) d=(d+(ll)C[i]*s[n-i])%mod;
            if (d==0) ++m;
            else if (2*L<=n) {
                VI T=C;
                ll c=mod-d*powmod(b,mod-2)%mod;
                while (SZ(C)<SZ(B)+m) C.pb(0);
                rep(i,0,SZ(B)) C[i+m]=(C[i+m]+c*B[i])%mod;
                L=n+1-L; B=T; b=d; m=1;
            } else {
                ll c=mod-d*powmod(b,mod-2)%mod;
                while (SZ(C)<SZ(B)+m) C.pb(0);
                rep(i,0,SZ(B)) C[i+m]=(C[i+m]+c*B[i])%mod;
                ++m;
            }
        }
        return C;
    }
    int gao(VI a,ll n) {
        VI c=BM(a);
        c.erase(c.begin());
        rep(i,0,SZ(c)) c[i]=(mod-c[i])%mod;
        return solve(n,c,VI(a.begin(),a.begin()+SZ(c)));
    }
};
vector<int> a;
int main() {
    int T;
    scanf("%d",&T);
    a.clear();
    a.pb(31),a.pb(197),a.pb(1255),a.pb(7997),a.pb(50959),a.pb(324725);
    while(T--)
    {
        scanf("%lld",&n);
        printf("%d\n",linear_seq::gao(a,n-1));
    }
    return 0;
}
```



## FFT

求解多项式乘法：
$$
C(x)=A(x)*B(x)=\sum_{k=0}^{2n-2}(\sum_{k=i+j}a_ib_j)x^k
$$
$DFT$（离散傅里叶变换）是一种对n个元素的数组的变换，根据式子直接的方法是$O(n^2)$的。但是用分治的方法是$O(nlogn)$，即$FFT$（快速傅里叶变换）。

由于$DFT$满足**cyclic convolution（循环卷积）**的性质，即

定义$h=a (*) b$  为$h_r=\sum_{x+y=r}a_xb_y$（多项式乘法）（$h$为结果多项式）

则有$DFT(a (*) b)=DFT(a)\cdot DFT(b)$ 右边为点乘（很自然）

所以所求$a (*) b=DFT^{-1}(DFT(a)\cdot DFT(b))$

即只要对$a,b$分别进行$DFT$变化之后点乘之后再逆变换就可以了



* 由于$FFT$本身算法的要求，$n$是$2$的次幂，注意补0
* $DFT$是定义在复数域上的，有与整数之间变换的要求
* 不要忘记最后除n

---

* C++ 的 STL 在头文件 `complex` 中提供一个复数的模板实现 `std::complex<T>`，其中 `T` 为实数类型，一般取 `double`，在对精度要求较高的时候可以使用 `long double` 或 `__float128`（不常用）。
* 单位根的倒数等于其共轭复数，使用`std::conj()` 取得 IDFT 所需的单位根的倒数。
* 对时间要求苛刻时，可以自己实现复数类以提高速度



**时间复杂度$O(nlogn)$  常数较大**



![img](https://s1.ax2x.com/2018/08/20/59v9Bl.png)

### FFT的递归实现

直接按照上述思路实现即可

使用C++已经封装的Complex

![img](https://s1.ax2x.com/2018/05/18/xeCfJ.png) 

```c++
#include<bits/stdc++.h>
using namespace std;
typedef long long ll;
int debug_num=0;
#define debug cout<<"debug "<<++debug_num<<" :"
#define lson l,m,rt<<1
#define rson m+1,r,rt<<1|1
#define bit(a,b) ((a>>b)&1)
const int inf = 0x3f3f3f3f;
const ll inff = 0x3f3f3f3f3f3f3f3f;
const double pi=acos(-1.0);
const int maxn=1e5;

typedef complex<double> xu;

xu a[maxn<<2],b[maxn<<2];

int inver;

void FFT(xu *s ,int n){
    if(n==1) return ;//n=1时值就是系数  因为只有w_1^0这一点
    xu a1[n>>1],a2[n>>1];
    for(int i=0;i<n;i+=2){
        a1[i>>1]=s[i];
        a2[i>>1]=s[i+1];
    }
    FFT(a1,n>>1); FFT(a2,n>>1);
    xu w=xu(cos(2*pi/n),inver*sin(2*pi/n));
    xu wn=xu(1,0);
    for(int i=0;i<(n>>1);++i,wn=wn*w){
        s[i]=a1[i]+wn*a2[i];
        s[i+(n>>1)]=a1[i]-wn*a2[i];
    }
}

int main()
{
#ifndef ONLINE_JUDGE
    freopen("in.txt","r",stdin);
#endif
    ios::sync_with_stdio(false);
    int n,m;
    cin>>n>>m;//多项式的最高次数
    int tp;
    n++;
    m++;
    for(int i=0;i<n;++i){
        cin>>tp;
        a[i]=xu(tp,0);
    }
    for(int i=0;i<m;++i){
        cin>>tp;
        b[i]=xu(tp,0);
    }
    int N=1; while(N<n+m-1) N<<=1;
    inver=1;
    FFT(a,N); FFT(b,N);
    for(int i=0;i<N;++i) a[i]=a[i]*b[i];
    //现在a中的值是上面的傅里叶变化的结果啦 wn0,wn1,wn2,...,wn n-1
    //下面作为逆变化的系数
    inver=-1;
    FFT(a,N);
    for(int i=0;i<n+m-1;++i) cout<<ll(a[i].real()/N+0.5)<<' ';
    return 0;
}
```





### FFT的迭代实现

递归实现的 FFT 效率不高，因为有栈的调用和参数的传递，实际中一般采用**迭代实现**

**二进制位翻转**

对分治规律的总结

```c
//以8项为例
000 001 010 011 100 101 110 111
0   1   2   3   4   5   6   7   //初始要求
0   2   4   6 / 1   3   5   7   //分治后的两个多项式的系数 对应于原多项式
0   4 / 2   6 / 1   5 / 3   7   
0 / 4 / 2 / 6 / 1 / 5 / 3 / 7
000 100 010 110 001 101 011 111 //发现正好是翻转
```

 这为迭代实现提供了理论基础



**蝴蝶操作**

考虑合并两个子问题的过程，假设$A_1(w_{\frac{n}{2}}^k)$ 和$A_2(w_{\frac{n}{2}}^k)$ 分别存放在$a[k]$和$a[\frac{n}{2}+k]$中，$A(w_n^k)$和$A(w_n^{k+\frac{n}{2}})$ 将要存放在$b[k]$和$b[\frac{n}{2}+k]$中，合并的操作可以表示为：
$$
b[k]\leftarrow  a[k]+w_n^k*a[\frac{n}{2}+k]  \\
b[k+\frac{n}{2}] \leftarrow a[k]-w_n^k*a[\frac{n}{2}+k]
$$
考虑加入一个临时变量t,使得这个过程可以在原地完成，而不需要数组b，	
$$
t \leftarrow w_n^k*a[\frac{n}{2}+k]   \\
a[k+\frac{n}{2}] \leftarrow a[k]-t  \\
a[k] \leftarrow a[k]+t
$$
由于$k$和$k+\frac{n}{2}$ 是对应的，所以不同的k之间不会相互影响

这一过程被称为**蝴蝶操作** 



**使用C++已经封装的Complex的迭代实现**  

![img](https://s1.ax2x.com/2018/05/18/xUnsp.png) 

```c++
#include<bits/stdc++.h>
using namespace std;
typedef long long ll;
int debug_num=0;
#define debug cout<<"debug "<<++debug_num<<" :"
#define lson l,m,rt<<1
#define rson m+1,r,rt<<1|1
#define bit(a,b) ((a>>b)&1)
const int inf = 0x3f3f3f3f;
const ll inff = 0x3f3f3f3f3f3f3f3f;
const double pi=acos(-1.0);
const int maxn=1e5;

typedef complex<double> xu;

xu omega[maxn<<2],omegaInverse[maxn<<2];//辅助

void init(const int n){
    for(int i=0;i<n;++i){
        omega[i]=xu(cos(2*pi/n*i),sin(2*pi/n*i));
        omegaInverse[i]=conj(omega[i]);
    }
}

void trans(xu *a,const int n,const xu *omega){//a是系数  n是2的次幂  omega是要带入的点
    int k=0;
    while((1<<k)<n) k++;
    //看N是2的多少次幂
    for(int i=0;i<n;++i){
        int t=0;
        for(int j=0;j<k;++j) if(i&(1<<j)) t|=(1<<(k-j-1));
        if(i<t) swap(a[i],a[t]);//原地翻转
    }
    for(int l=2;l<=n;l*=2){//l=1不用求 就是系数
        int m=l/2;
        for(xu *p=a;p!=a+n;p+=l){//l为一段要求 由两段l/2合并得到
            //当前处理[p,p+l)
            for(int i=0;i<m;++i){
                //蝴蝶操作  omega_{l}^{i}=omega_{N}^{N/l*i}
                xu t=omega[n/l*i]*p[i+m];
                p[i+m]=p[i]-t;
                p[i]+=t;
            }
        }
    }
}

void dft(xu *a,const int n){
    trans(a,n,omega);
}

void idft(xu *a,const int n){
    trans(a,n,omegaInverse);
    for(int i=0;i<n;++i) a[i]/=n;
}


xu a[maxn<<2];
xu b[maxn<<2];


int main()
{
#ifndef ONLINE_JUDGE
    freopen("in.txt","r",stdin);
#endif
    ios::sync_with_stdio(false);
    int n,m;
    cin>>n>>m;//多项式的最高次数
    n++;
    m++;
    int tp;
    for(int i=0;i<n;++i){
        cin>>tp;
        a[i]=xu(tp,0);
    }
    for(int i=0;i<m;++i){
        cin>>tp;
        b[i]=xu(tp,0);
    }
    int N=1;
    while(N<m+n-1) N<<=1;
    init(N);
    dft(a,N);
    dft(b,N);
    for(int i=0;i<N;++i) a[i]*=b[i];
    idft(a,N);
    for(int i=0;i<n+m-1;++i){
        cout<<(ll)(a[i].real()+0.5)<<" ";
    }
    return 0;
}
```





## 快速沃尔什变换(FWT)

wmr

```c++
#include<bits/stdc++.h>
using namespace std;
const int maxn=1024,mod=1e9+7,rev=(mod+1)>>1;
int a[maxn+50],b[maxn+50];
long long fc[maxn*maxn];
int T,n,m;
void fwt(int *a,int n)
{
    for(int d=1;d<n;d<<=1)
        for(int m=d<<1,i=0;i<n;i+=m)
            for(int j=0;j<d;j++)
            {
                int x=a[i+j],y=a[i+j+d];
                a[i+j]=(x+y)%mod,a[i+j+d]=(x-y+mod)%mod;
                //xor:a[i+j]=x+y,a[i+j+d]=(x-y+mod)%mod;
                //and:a[i+j]=x+y;
                //or:a[i+j+d]=x+y;
            }
}
void ufwt(int *a,int n)
{
    for(int d=1;d<n;d<<=1)
        for(int m=d<<1,i=0;i<n;i+=m)
            for(int j=0;j<d;j++)
            {
                int x=a[i+j],y=a[i+j+d];
                a[i+j]=1LL*(x+y)*rev%mod,a[i+j+d]=(1LL*(x-y)*rev%mod+mod)%mod;
                //xor:a[i+j]=(x+y)/2,a[i+j+d]=(x-y)/2;
                //and:a[i+j]=x-y;
                //or:a[i+j+d]=y-x;
            }
}
void solve(int *a,int *b,int n)//下标0..n-1的数组a和b求异或卷积，O(nlogn)，返回值在a中
{
    fwt(a,n);
    fwt(b,n);
    for(int i=0;i<n;++i) a[i]=1LL*a[i]*b[i]%mod;
    ufwt(a,n);
}
int main()
{
    fc[0]=1;
    for(int i=1;i<=1000000;++i) fc[i]=fc[i-1]*i%mod;
    scanf("%d",&T);
    while(T--)
    {
        scanf("%d%d",&n,&m);
        memset(a,0,sizeof(a));
        memset(b,0,sizeof(b));
        for(int i=1;i<=n;++i) a[i]=1;
        for(int i=1;i<=m;++i) b[i]=1;
        solve(a,b,maxn);
        long long ans=1;
        for(int i=0;i<maxn;++i) ans=ans*fc[a[i]]%mod;
        printf("%lld\n",ans);
    }
    return 0;
}
```




## 快速数论变换(NTT)

wmr

```c++
#include<bits/stdc++.h>
using namespace std;
typedef long long ll;
const int maxn=262144*4;
//const long long P=50000000001507329LL; // 190734863287 * 2 ^ 18 + 1
//const int P=1004535809; // 479 * 2 ^ 21 + 1
const ll mod=998244353; // 119 * 2 ^ 23 + 1
const ll G=3;

ll len=0;
ll pw[maxn+5],pwinv[maxn+5];
ll A[maxn+5],B[maxn+5];
ll f[maxn+5];
int n,m;

ll Pow(ll a,ll b,ll mod)
{
    ll ans=1;
    while(b)
    {
        if(b&1) ans=ans*a%mod;
        a=a*a%mod;
        b>>=1;
    }
    return ans;
}
ll Inv(ll x)
{
    return Pow(x,mod-2,mod);
}
void Init()
{
    ll inv=Inv(G);
    for (int i=1;i<=maxn;i<<=1)
    {
        pw[i]=Pow(G,(mod-1)/i,mod);
        pwinv[i]=Pow(inv,(mod-1)/i,mod);
    }
}
void rader(ll *a)
{
    for(int i=0,j=0;i<len;i++)
    {
        if(i>j) swap(a[i],a[j]);
        int k=len;
        do{k>>=1;j^=k;}while(j<k);
    }
}
void ntt(ll *a,int f)
{
    rader(a);
    for(int i=2;i<=len;i<<=1)
    {
        int m=i>>1;
        for(int j=0;j<len;j+=i)
        {
            ll w=1,wn=pw[i];
            if(f==-1) wn=pwinv[i];
            for(int k=0;k<m;k++)
            {
                ll x=a[j+k+m]*w%mod;
                a[j+k+m]=(a[j+k]-x+mod)%mod;
                a[j+k]=(a[j+k]+x)%mod;
                w=w*wn%mod;
            }
        }
    }
    if(f==-1)
    {
        ll inv=Inv(len);
        for(int i=0;i<len;i++) a[i]=a[i]*inv%mod;
    }
}
void con(ll *A,int n,ll *B,int m)
{
    /*A[0..n-1]与B[0..m-1]卷积*/
    for(len=1;len<max(n,m);len<<=1);
    len<<=1;
    for(int i=n;i<len;++i) A[i]=0;
    for(int i=m;i<len;++i) B[i]=0;
    ntt(A,1);
    ntt(B,1);
    for(int i=0;i<len;++i) A[i]=A[i]*B[i]%mod;
    ntt(A,-1);
}
int main()
{
    Init();
    con(A,n,B,m);
    return 0;
}
```





## 分数类

完成分数的**加减乘除**运算

分子分母最好不要超过int     这样相乘不会超过long long

```c++
#include<bits/stdc++.h>
using namespace std;

typedef long long ll;

ll gcd(ll a,ll b){
    if(b==0) return a;
    return gcd(b,a%b);
}

struct Fraction{
    ll num;
    ll den;
    Fraction (ll num=0,ll den=1){
        if(den<0){
            num=-num;
            den=-den;
        }
        assert(den != 0);
        ll g=gcd(abs(num),den);
        this->num=num/g;
        this->den=den/g;
    }
    Fraction operator + (const Fraction &o) const{
        return Fraction(num*o.den+den*o.num,den*o.den);
    }
    Fraction operator - (const Fraction &o) const{
        return Fraction(num*o.den-den*o.num,den*o.den);
    }
    Fraction operator * (const Fraction &o) const{
        return Fraction(num*o.num,den*o.den);
    }
    Fraction operator / (const Fraction &o) const{
        return Fraction(num*o.den,den*o.num);
    }
    bool operator < (const Fraction &o) const{
        return num*o.den<den*o.num;
    }
    bool operator == (const Fraction &o) const{
        return num*o.den==den*o.num;
    }
};

int main()
{
    Fraction f1(33455,-3243235);
    Fraction f2(3533245,345323);
    cout<<f1.num<<"/"<<f1.den<<endl;
    cout<<f2.num<<"/"<<f2.den<<endl;
    Fraction f3=f2-f1;
    cout<<f3.num<<"/"<<f3.den<<endl;
    return 0;
}

```

## 康拓展开

**用排列求数时**

把排列看成一个多进制数。第i位（1~n位）的进制权值是$(N-i)! $

设$a[i]=x$ ，$a[i+1, n]$ 中有$k$个比$x$小。那么这一位应该用$k*(N-i)!$ 来作为权值，最后加上所有位的权值即可。



**用数求排列时**(上一过程的逆过程)

从高位到低位一位一位确定。用这一位的权值除以这一位对应的阶乘，若为k，那么从当前还没用过的数（用过的一定在它前面了）中找出第k+1小（即后面有k个比它小）的即可。



```c++
#include<bits/stdc++.h>
using namespace std;

const int maxn=100;

typedef long long ll;

ll factorial[maxn];

int n;
ll x;

//O(n^2) a[] 给定的排列   x散列值(从0开始)
ll array_to_int(int a[maxn])
{
    int i,j;
    ll ans=0,temp;
    for(i=1;i<=n;++i){
        temp=0;
        for(j=i+1;j<=n;++j) if(a[j]<a[i]) ++temp;
        ans+=factorial[n-i]*temp;
    }
    return ans;
}

//O(n^2) x散列值(从0开始) a[] 散列值对应的排列
void int_To_Array(ll x,int a[maxn])
{
    bool used[maxn];
    int i,j;
    ll temp;
    for(i=1;i<=n;++i) used[i]=false;
    for(i=1;i<=n;++i){
        temp=x/factorial[n-i];
        for(j=1;j<=n;++j) if(!used[j]){
            if(temp==0) break;
            --temp;
        }
        a[i]=j;
        used[j]=true;
        x%=factorial[n-i];
    }
}

void init()
{
    factorial[0]=factorial[1]=1;
    for(int i=2;i<=19;++i){
        factorial[i]=factorial[i-1]*i;
    }
}

int ans[maxn];

int main()
{
    init();
    cin>>n>>x;
    //序号x从0开始
    //排列为1~n
    int_To_Array(x,ans);
    for(int i=1;i<=n;++i) cout<<ans[i]<<' ';

    cout<<endl;

    cout<<array_to_int(ans)<<endl;


    return 0;
}

```

## 数值积分

给定函数$f(x)$ ，用数值方法求积分$\int_{a}^{b}f(x)dx$ 



这里给出两种方法$Simpson$和$Romberg$ 

$Simpson$方法比较简单，是以二次曲线逼近的方式取代矩形或梯形积分公式，以求得定积分的数值近似解

$Romberg$稍微复杂，这里不做介绍，**同等的计算复杂度下，精度比$Simpson$高**



```c++
//Simpson和Romberg
#include<bits/stdc++.h>
using namespace std;
typedef long long ll;
int debug_num=0;
#define debug cout<<"debug "<<++debug_num<<" :"
const double pi=acos(-1.0);

//f 为被积函数
//a b 为积分上下限
//n为划分份数
//求积分值
inline double f(double x)//被积函数
{
    return sin(x);
}

double simpson(double a,double b,int n){//分成n块再相加 n一般取300 特殊情况特殊考虑
    const double h=(b-a)/n;
    double ans=f(a)+f(b);
    for(int i=1;i<n;i+=2) ans+=4*f(a+i*h);
    for(int i=2;i<n;i+=2) ans+=2*f(a+i*h);
    return ans*h/3;
}

double romberg(double a,double b,double eps=1e-8){//eps为允许误差
    vector<double > t;
    double h=b-a,last,curr;
    int k=1,i=1;
    t.push_back(h*(f(a)+f(b))/2); //梯形
    do{
        last=t.back();
        curr=0;
        double x=a+h/2;
        for(int j=0;j<k;++j){
            curr+=f(x);
            x+=h;
        }
        curr=(t[0]+h*curr)/2;
        double k1=4.0/3.0,k2=1.0/3.0;
        for(int j=0;j<i;j++){
            double temp=k1*curr-k2*t[j];
            t[j]=curr;
            curr=temp;
            k2/=4*k1-k2;
            k1=k2+1;
        }
        t.push_back(curr);
        k*=2;
        h/=2;
        i++;
    }while(fabs(last-curr)>eps);
    return t.back();
}

int main()
{
    //freopen("in.txt","r",stdin);
    ios::sync_with_stdio(false);
    cout<<"simpson:"<<simpson(0,pi,10)<<"  romberg:"<<romberg(0,pi,1e-1)<<endl;
    cout<<"simpson:"<<simpson(0,pi,100)<<"  romberg:"<<romberg(0,pi,1e-4)<<endl;
    cout<<"simpson:"<<simpson(0,pi,200)<<"  romberg:"<<romberg(0,pi,1e-6)<<endl;
    cout<<"simpson:"<<simpson(0,pi,300)<<"  romberg:"<<romberg(0,pi)<<endl;//默认1e-8
    return 0;
}
```





## 自适应辛普森算法

Adaptive Simpson's method。

对Simpson方法的优化

具体的方法是，改**每隔Δx取一个区段**为**智能地**取区段。 如何让程序智能起来？容易脑补，如果某段区间容易拟合上，那就少取几段；如果难以拟合，就多取几段。

如何衡量是否“容易拟合”？上面的辛普森公式给了很好的方案。 令Sim(le,rt)为辛普森公式所求出的答案。那么：

1. 如果$Sim(a,mid)+Sim(mid,b)−Sim(a,b)<γ$

   则这一块**可以**很好地拟合上，返回$Sim(a,mid)+Sim(mid,b)$

2. 如果$Sim(a,mid)+Sim(mid,b)−Sim(a,b)≥γ$
   则这一块**不能**很好地拟合上，**递归处理**$(a,mid)和(mid,b)$

上面的方法很容易理解。如果(a,b)可以很好地被拟合，那么Sim(a,mid)+Sim(mid,b)应该和Sim(a,b)相差无几。

如果函数难以拟合，那么我们就需要多次拟合，来计算出**尽可能**准确的结果。所以，这个算法是“自适应”的。

精度比之前的普通辛普森算法好得多。





## 拉格朗日插值法求积分

拉格朗日插值法求积分的原理是，**以多项式来拟合被积函数，然后积多项式**。

拉格朗日插值法：给定n个点，拟合出一条多项式函数，过这些点。

由于我们可以选择很多个点，因此多项式可以做得与原函数**相当接近**。同时，多项式函数很容易求导，所以也容易求积分，精度有保障。

**时间复杂度**：找到n个点用时$O(n)$，插值$O(n^2)$，求导$O(n)$，求积分$O(n)$。 如果懒得写拉格朗日插值法，高斯消元(n个方程的方程组)也可以，但是插值的复杂度就变成$O(n^3)$。





## 泰勒展开求积分

如果我们遇到的函数**可以泰勒展开**（例如$e^x$，$sin⁡(x)$之类的函数），那么我们可以将它展开成一个多项式，然后直接用多项式的积分方法来求解。

例如，如果我们要求$f(x)=sin(x)$的积分，我们先把它在$O(n)$内展开成$f(x)=x−x^3/3!+x^5/5!−x^7/7!+⋯$然后在$O(n)$内求出对应的原函数F(x)，然后在$O(n)$内用**霍纳法**则求出F(l)和F(r)的值。

**霍纳法**（中文名为**秦九韶算法**） 

名字高大上，实际上就是从低次幂向高次幂计算，这样x值可以重复利用，时间复杂度$O(n)$





**泰勒展开**

如果函数$f(x)$在含有$x_0$的某个开区间(a,b)内具有(n+1)阶的导数，那么对于任一$x∈(a,b)$，有
$$
f(x) = f(x_0) + f'(x_0)(x-x_0) + \frac{f''(x_0)}{2!}(x-x_0)^2 + \cdots + \frac{f^{(n)}(x_0)}{n!}(x-x_0)^n + R_n(x)
$$
其中
$$
R_n(x) = \frac{f^{(n+1)}(\epsilon)}{(n+1)!}(x-x_0)^{(n+1)}
$$


这里的ϵ是介于$x$与$x0$之间的某个值，$R_n$被称为**拉格朗日余项**。

一般常取在 $x=0$ 处的展开（也称作**麦克劳林展开**）。



题外话：关于[泰勒公式的形象解释](https://www.zhihu.com/question/21149770)




## 全排列中的格雷码序列

给定一个二进制的位数n,**求出一个0到$2^n-1$的排列**，使得相邻两项（**包括头尾相邻**）的**二进制表达**中只有恰好一位不同。

Gray 序列的第$i$位为$i \ xor (i>>1)$ 

* 时间复杂度$O(2^n)$  

```c++
//O(2^n)
#include<bits/stdc++.h>
using namespace std;

//输出n位的格雷码序列
vector<int> Gray_Create(int n){
    vector<int> res;
    res.clear();
    for(int i=0;i<(1<<n);++i){
        res.push_back(i^(i>>1));
    }
    return res;
}

vector<int> ans;

int main()
{
    int n;//二进制的位数
    cin>>n;
    ans=Gray_Create(n);
    for(int i=0;i<ans.size();++i) cout<<ans[i]<<' ';
    return 0;
}

```







## 进制转换

把一个x进制数转换成y进制



```c++
//先把x进制转为十进制(不超过long long)  再将其不断取模再倒序 转换成y进制数
//复杂度O(length)
#include<bits/stdc++.h>
using namespace std;

//2<=x,y<=36
string trans_form(int x,int y,string s){
    string res="";
    long long sum=0;
    for(int i=0;i<s.length();++i){
        if(s[i]=='-') continue;
        if(s[i]>='0'&&s[i]<='9'){
            sum=sum*x+s[i]-'0';
        }
        else{
            sum=sum*x+s[i]-'A'+10;
        }
    }
    while(sum){
        char tmp=sum%y;
        sum/=y;
        if(tmp<=9){
            tmp+='0';
        }
        else{
            tmp=tmp-10+'A';
        }
        res=tmp+res;
    }
    if(res.length()==0) res="0";
    if(s[0]=='-') res='-'+res;
    return res;
}

int main()
{
    int x,y;
    string tp;
    while(cin>>tp>>x>>y) cout<<trans_form(x,y,tp)<<endl;
    return 0;
}

```

## 整数高精度

$+ \ - * \ / < \ ==  \ \% $        

```c++
//高精度(压四位)
#include<bits/stdc++.h>
using namespace std;

const int ten[4]={1,10,100,1000};//辅助压位 压四位 这样乘法不会超int
const int maxl=1000; //最大位数

struct BigNumber{
    int d[maxl];//每4位压位 d[0]为压位后的位数 d[1]为原先的最低四位
    BigNumber (string s){
        int len=s.size();
        d[0]=(len-1)/4+1;
        int i,j,k;
        for(i=1;i<maxl;++i) d[i]=0;
        for(i=len-1;i>=0;--i){
            j=(len-i-1)/4+1;
            k=(len-i-1)%4;
            d[j]+=ten[k]*(s[i]-'0');
        }
        while(d[0]>1 && d[d[0]]==0) --d[0];//去除前导零
    }
    BigNumber(){
        *this=BigNumber(string("0"));
    }
    string toString(){
        string s("");
        int i,j,temp;
        //先处理高1位
        for(i=3;i>=1;--i) if(d[d[0]]>=ten[i]) break;
        temp=d[d[0]];
        for(j=i;j>=0;--j){
            s=s+(char)(temp/ten[j]+'0');
            temp%=ten[j];
        }
        //再处理剩下的位
        for(i=d[0]-1;i>0;--i){
            temp=d[i];
            for(j=3;j>=0;--j){
                s=s+(char)(temp/ten[j]+'0');
                temp%=ten[j];
            }
        }
        return s;
    }
}zero("0"),d,mid1[15];//d为作除法时的余数（可看做是高精度模高精度） mid1辅助作除法


bool operator <(const BigNumber &a,const BigNumber &b){
    if(a.d[0]!=b.d[0]) return a.d[0]<b.d[0];
    int i;
    for(i=a.d[0];i>0;--i) if(a.d[i]!=b.d[i]) return a.d[i] < b.d[i];
    return false;
}

//一定没有前导零
BigNumber operator +(const BigNumber &a,const BigNumber &b){
    BigNumber c;
    c.d[0]=max(a.d[0],b.d[0]);
    int i,x=0;
    for(i=1;i<=c.d[0];++i){
        x=a.d[i]+b.d[i]+x;
        c.d[i]=x%10000;
        x/=10000;
    }
    while(x!=0)
    {
        c.d[++c.d[0]]=x%10000;
        x/=10000;
    }
    return c;
}

BigNumber operator - (const BigNumber &a,const BigNumber &b){
    BigNumber c;
    c.d[0]=a.d[0];
    int i,x=0;
    for(i=1;i<=c.d[0];++i){
        x=10000+a.d[i]-b.d[i]+x;
        c.d[i]=x%10000;
        x=x/10000-1;
    }
    while((c.d[0]>1)&&(c.d[c.d[0]]==0)) --c.d[0];
    return c;
}

BigNumber operator *(const BigNumber &a,const int &k){//乘以较小数
    BigNumber c;
    c.d[0]=a.d[0];
    int i,x=0;
    for(i=1;i<=a.d[0];++i){
        x=a.d[i]*k+x;
        c.d[i]=x%10000;
        x/=10000;
    }
    while(x>0)
    {
        c.d[++c.d[0]]=x%10000;
        x/=10000;
    }
    while((c.d[0]>1) && (c.d[c.d[0]]==0) ) --c.d[0];
    return c;
}


BigNumber operator *(const BigNumber &a,const BigNumber &b){
    BigNumber c;
    c.d[0]=a.d[0]+b.d[0];
    int i,j,x;
    for(i=1;i<=a.d[0];++i){
        x=0;
        for(j=1;j<=b.d[0];++j){
            x=a.d[i]*b.d[j]+x+c.d[i+j-1];
            c.d[i+j-1]=x%10000;
            x/=10000;
        }
        c.d[i+b.d[0]]=x;//保留进位 可能产生前导零
    }
    while((c.d[0]>1)&&(c.d[c.d[0]]==0)) --c.d[0];
    return c;
}

//辅助除法
bool smaller(const BigNumber & a,const BigNumber &b ,int delta){
    if(a.d[0]+delta!=b.d[0]) return a.d[0]+delta<b.d[0];
    //a为试减的数 b为余数
    //如果b的位数和a+delta是相等的
    int i;
    //从最高位开始
    for(i=a.d[0];i>0;--i) if(a.d[i]!=b.d[i+delta]) return a.d[i]<b.d[i+delta];//i+delta为b高位对应a的数
    return true;//都相等返回true
}

//辅助除法
void Minus(BigNumber &a,const BigNumber &b,int delta){
    //引用a 即余数   b为要减去的数 即除数的倍数
    int i,x=0;
    //a最高的a.d[0]-delta位减去b
    for(i=1;i<=a.d[0]-delta;++i){
        x=10000+a.d[i+delta]-b.d[i]+x;
        a.d[i+delta]=x%10000;
        x=x/10000-1;
    }
    while((a.d[0]>1) && (a.d[a.d[0]]==0)) --a.d[0];
}

BigNumber operator / (const BigNumber &a,const BigNumber &b){
    BigNumber c;
    d=a;//余数
    int i,j,temp;
    mid1[0]=b;
    for(i=1;i<=13;++i){
        mid1[i]=mid1[i-1]*2;
    }
    for(i=a.d[0]-b.d[0];i>=0;--i){
        temp=8192;//2^13   倍增思想
        for(j=13;j>=0;--j){ //mid1[j] 为 除数b*2^j
            if(smaller(mid1[j],d,i)){
                Minus(d,mid1[j],i);
                c.d[i+1]+=temp;//i+1是商的位权 因为i是delta
            }
            temp/=2;
        }
    }
    c.d[0]=max(1,a.d[0]-b.d[0]+1);
    while((c.d[0]>1) && (c.d[c.d[0]]==0)) --c.d[0];
    return c;
}


int main()
{
    string tp1,tp2;
    cin>>tp1>>tp2;
    BigNumber a=(tp1);
    BigNumber b=(tp2);
    BigNumber c=a-b;
    BigNumber d=a+b;
    BigNumber e=a*b;
    BigNumber f=a/b;
    cout<<c.toString()<<' '<<d.toString()<<' '<<e.toString()<<' '<<f.toString()<<endl;
    return 0;
}




```



## 大整数除较小数

```c++
#include<iostream>
#include<algorithm>
using namespace std;

typedef long long ll;

string div(string a,ll b,ll &d)
{
    string r,ans;
    d=0;
    if(a=="0") return a;
    for(int i=0;i<a.length();i++){
            r+=(d*10+a[i]-'0')/b+'0';//求出商
            d=(d*10+(a[i]-'0'))%b;//求出余数
    }
    int p=0;
    for(int i=0;i<r.length();i++)
    if(r[i]!='0') {p=i;break;}
    return r.substr(p);//从p截取到末尾 去除前导零
}

int main()
{
    ios::sync_with_stdio(false);
    string a;
    ll b,d;
    while(cin>>a>>b)
    {
        string ans=div(a,b,d);
        cout<<"商: "<<ans<<"     "<<"余数: "<<d<<endl;
    }
    return 0;
}

```



## 大整数模较小数

```c++
long long BignumMod(char *s,int mod)//大整数取模
{
    long long ans=0;
    for(int i=0;s[i]!='\0';++i){
        ans=(ans*10+s[i]‐'0')%mod;
    }
    return ans;
}
```





## 大整数幂

```c++

```





## 大整数开根

大整数位数n

* **牛顿迭代法**  $ O(n^2logn)$

其中迭代次数为$log(n)$  每一次迭代为$O(2n^2+2n)$  



牛顿迭代法的核心思想：利用一个根的猜测值$x_0$作为近似值，使用函数f(x)在$x_0$处的**泰勒级数展式的前两项**作为函数f(x)的近似表达式...迭代



* **二分法**$O(n^3)$  

其中二分次数为$O(3.32*n)$   每一次二分为$O(2n^2+2n)$  

```c
//类的定义为前面高精度
BigNumber tp;
string s;
BigNumber f(BigNumber x)
{
    return x*x-tp;
}

BigNumber deri(BigNumber x)
{
    return x*2;
}

BigNumber newton_sqrt(BigNumber a)//牛顿迭代法
{
    int num=a.d[0]*4;
    string tmp="1";
    for(int i=0;i<num/2+1;++i){
        tmp+="0";
    }//开始的a大约是原位数的一半
    a=BigNumber(tmp);
    num=log(num)+20;
    while(num--)
    {
        a=a-f(a)/deri(a);
        //cout<<num<<"******"<<a.toString()<<endl;
    }
    while(tp<a*a){
        a=a-BigNumber("1");
    }
    return a;
}

BigNumber bina_sqrt(BigNumber a)//二分法
{
    BigNumber l("0");
    BigNumber r(s);
    BigNumber cp("1");
    while(cp<r-l)
    {
        BigNumber mid=(l+r)/BigNumber("2");
        //cout<<"******"<<mid.toString()<<endl;
        if(mid*mid<tp) l=mid;
        else r=mid;
    }
    while(tp<r*r){
        r=r-BigNumber("1");
    }
    return r;
}

int main()
{
    freopen("in.txt","r",stdin);
    ios::sync_with_stdio(false);
    cin>>s;
    BigNumber a(s);
    tp=a;
    BigNumber ans=newton_sqrt(a);
    cout<<ans.toString()<<endl;
    cout<<clock()<<endl;
    cout<<endl<<"**********************************"<<endl;

    BigNumber bns=bina_sqrt(a);
    cout<<bns.toString()<<endl;
    cout<<clock()<<endl;

    return 0;
}
```









## 高阶代数方程求根

给定方程$a_nx^n+a_{n-1}x^{n-1}+...+a_1x+a_0=0$ ，求出该方程的所有实数解



首先对其求导，求出其导函数的所有零点，那么**在导函数两个相邻的零点之间，该n次方程一定是单调的**，并且最多只有一个零点，利用这个性质，我们可以二分求出这个零点。

而求出导函数的零点可以**递归**地做下去，直到n=1时，可以直接返回答案。



**时间复杂度分析**

对于n次方程，假设现在已经求出其所有极值点（最大n-1个），最多需要二分n个区间（包括两端），每一次二分最坏是100*n的复杂度。由于是递归的，所以上述过程要做n次，所以总时间为

**这里100是每一次二分循环次数 ，每一次二分要求n阶函数的值，是$O(n)$的，调用二分的次数大概是$n^2$级别的。**

* **总时间为$O(n^3*100)$** 

再考虑到每一次递归时常数较大

**n>100时需谨慎**  



```c++
//TIMUS1503  模板题
#include<bits/stdc++.h>
using namespace std;
typedef long long ll;
int debug_num=0;
#define debug cout<<"debug "<<++debug_num<<" :"

const double eps=1e-12;
const double inf=1e12;

inline int sign(double x){
    return x<-eps ? -1 : x>eps ;
}

//coef 系数  coef[i]=a[i]
inline double get(const vector<double> &coef,double x){//将x带入方程的值
    double e=1,s=0;//e=x^i (i=0,1,2,...,n)   coef.size():n+1
    for(int i=0;i<coef.size();++i) s+=coef[i]*e,e*=x;
    return s;
}

double Find(const vector <double> &coef,int n,double lo,double hi){
    double sign_lo,sign_hi;
    if((sign_lo=sign(get(coef,lo)))==0) return lo;//lo就是零点
    if((sign_hi=sign(get(coef,hi)))==0) return hi;//hi就是零点
    if(sign_hi*sign_lo>0) return inf;//当前区间没有零点
    for(int step=0;step<100 && hi-lo>eps;++step){//标准二分
        double m=(lo+hi)*.5;
        int sign_mid=sign(get(coef,m));
        if(sign_mid==0) return m;
        if(sign_lo*sign_mid<0) hi=m;
        else lo=m;
    }
    return (lo+hi)*.5;
}

vector<double> solve(vector<double> coef,int n){
    vector<double> ret;
    if(n==1){
        if(sign(coef[1])) ret.push_back(-coef[0]/coef[1]);
        return ret;//直接求解
    }
    vector<double> dcoef(n);//求导 大小只有n(原函数是n+1)
    for(int i=0;i<n;++i) dcoef[i]=coef[i+1]*(i+1);
    vector<double> droot=solve(dcoef,n-1);
    droot.insert(droot.begin(),-inf);
    droot.push_back(inf);//开头结尾插上-inf 和inf 方便二分  关键（和最高次系数的正负有关？）
    for(int i=0;i+1<droot.size();++i){
        double tmp=Find(coef,n,droot[i],droot[i+1]);
        if(fabs(tmp)<inf) ret.push_back(tmp);//是有效根
    }
    return ret;
}

vector <double > ans;

vector <double > coef;

int main()
{
    //freopen("in.txt","r",stdin);
    //ios::sync_with_stdio(false);
    int n;
    scanf("%d",&n);
    ans.clear();
    coef.clear();
    for(int i=0;i<=n;++i){//n+1个系数
        double tp;
        scanf("%lf",&tp);
        coef.push_back(tp);
    }
    reverse(coef.begin(),coef.end());//注意顺序看要不要逆序   coef[0]放的是x^0对应的系数

    ans=solve(coef,n);
    sort(ans.begin(),ans.end());
    for(int i=0;i<ans.size();++i){
        printf("%.6f\n",ans[i]);
    }
    return 0;
}


```

## 矩阵类

```c++
//矩阵类的定义 模数需要时加上
const int maxn=101;
const int maxm=101;

struct Matrix{
    int n,m;//行 列
    int a[maxn][maxm];
    void clear(){
        n=m=0;
        memset(a,0,sizeof(a));
    }

    Matrix operator +(const Matrix &b) const{
        Matrix tmp;
        tmp.n=n; tmp.m=m;
        for(int i=0;i<n;++i)
            for(int j=0;j<m;++j)
                tmp.a[i][j]=a[i][j]+b.a[i][j];
        return tmp;
    }

    Matrix operator -(const Matrix &b) const{
        Matrix tmp;
        tmp.n=n; tmp.m=m;
        for(int i=0;i<n;++i)
            for(int j=0;j<m;++j)
                tmp.a[i][j]=a[i][j]-b.a[i][j];
        return tmp;
    }

    Matrix operator *(const Matrix &b) const{
        Matrix tmp;
        tmp.clear();
        tmp.n=n; tmp.m=b.m;
        for(int i=0;i<n;++i)
            for(int j=0;j<b.m;++j)
                for(int k=0;k<m;++k)
                    tmp.a[i][j]+=a[i][k]*b.a[k][j];
        return tmp;
    }
};
```





## 矩阵的逆

时间复杂度  $O(n^3)$ 

```c++
//矩阵的逆 将原矩阵A和一个单位矩阵E作成大矩阵(A,E)
//用初等行变换将大矩阵中的A变为E，则会得到(E,A-1)的形式
//O(n^3)
#include<bits/stdc++.h>
using namespace std;
const double eps=1e-8;
const int maxn=101;

inline vector<double> operator * (vector<double> a,double b){
    int N=a.size();
    vector<double> res(N,0);//大小为N  初值为0的动态数组
    for(int i=0;i<N;++i)
        res[i]=a[i]*b;
    return res;
}

inline  vector<double> operator - (vector<double> a,vector<double> b){
    int N=a.size();
    vector<double> res(N,0);
    for(int i=0;i<N;++i)
        res[i]=a[i]-b[i];
    return res;
}

inline void inverse(vector<double> A[],vector<double> C[],int N){
    for(int i=0;i<N;++i)
        C[i]=vector<double> (N,0);
    for(int i=0;i<N;++i)
        C[i][i]=1;//初始化 N*N单位阵
    //在对角线上都要非零
    for(int i=0;i<N;++i){//枚举列

        for(int j=i;j<N;++j)//枚举行
            if(fabs(A[j][i])>eps){
                swap(A[i],A[j]);
                swap(C[i],C[j]);
                break;//找到就break
            }
        C[i]=C[i]*(1/A[i][i]);
        A[i]=A[i]*(1/A[i][i]);//对角线消为1

        for(int j=0;j<N;++j)//枚举行
            if(j!=i && fabs(A[j][i])>eps){
            C[j]=C[j]-C[i]*A[j][i];
            A[j]=A[j]-A[i]*A[j][i];
            }
    }
}

vector<double> A[101];//要求矩阵
vector<double> C[101];//逆矩阵

int main()
{
    ios::sync_with_stdio(false);
//    freopen("read.txt","r",stdin);
    int n;//矩阵大小
    cin>>n;
    for(int i=0;i<n;++i){
        for(int j=0;j<n;++j){
            double tmp;
            cin>>tmp;
            A[i].push_back(tmp);
        }
    }
    inverse(A,C,n);
    //输出逆矩阵
    for(int i=0;i<n;++i){
        for(int j=0;j<n;++j){
            cout<<C[i][j]<<' ';
        }
        cout<<endl;
    }
    return 0;
}

```

## 矩阵的简单应用

矩阵经常被应用到**递推、动态规划**的转移当中，有一个二维状态数组$dp[n][m]$，有如下转移方程$dp[i][j]=a_{j1}dp[i-1][1]+a_{j2}dp[i-1][2]+...+a_{jm}dp[i-1][m] $。

> 实际上我们正好可以用一个矩阵乘法来表示这个过程，A矩阵是一个$m*m$的矩阵，我们一般叫做**状态转移矩阵**

$$
\begin{bmatrix}dp[i][1]\\ dp[i][2]\\ ......\\ dp[i][m]\end{bmatrix}=\begin{bmatrix}a[1][1] &a[1][2]  & ...... &a[1][m] \\  a[2][1]&a[2][2]  &...... & a[2][m]\\  ......& ...... &......  &...... \\  a[m][1]& a[m][2] & ...... &a[m][m] \end{bmatrix}\begin{bmatrix}dp[i-1][1]\\ dp[i-1][2]\\ ......\\ dp[i-1][m]\end{bmatrix}
$$

第二维都是$1,2,...,m$，于是可以简记成$dp[i]=A·dp[i-1]$
通过递推可以得到$dp[n]=A^{n-1}dp[1]$，即我们通过矩阵的乘法表示了dp的转移。
而对于乘法可以采用**二分快速幂**的方法，求得$A^{n-1} $,从而计算$dp[n] $。





## 矩阵快速幂

已知$f_x=a_0f_{x-1}+a_1f_{x-2}+...+a_{n-1}f_{x-n}$和$f_0,f_1,...,f_{n-1}$，给定t，求$f_t$ 
$$
A=\begin{bmatrix}

0& 1 & 0&...&0\\
0& 0 &1 &...&0\\ 
..  & ..  &..&...&..\\
0& 0&0 &... &1\\
a_{n-1} & a_{n-2} &a_{n-3}&...&a_{0} 

\end{bmatrix},B=\begin{bmatrix}
f_{x-n}\\
f_{x-n+1}\\
..\\
..\\
f_{x-2}\\
f_{x-1}
\end{bmatrix}
$$
**时间复杂度    $O(n^3logt)$**



```c
int solve(int a[],int b[],int m,int t){
    //a是常系数数组 b是初值数组 m是数组大小 t是要求解的项数 从第0项开始 所以共有t+1项
    //m为已知递推式的阶数
    //输出函数在第t项的值f(t)
    //调用矩阵类
    Matrix M,F,res;//M是辅助常数矩阵 F是转移矩阵
    M.clear(),F.clear(),res.clear();
    M.n=M.m=m;
    res.n=res.m=m;
    F.n=m,F.m=1;
    for(int i=0;i<m-1;++i)
        M.a[i][i+1]=1;
    for(int i=0;i<m;++i){
        M.a[m-1][i]=a[i];//a[i]先存小项的系数 即递推式最靠末尾的系数
        F.a[i][0]=b[i];//b[i]先存小项的值 即f0(通常)
        res.a[i][i]=1;//初始化为单位矩阵
    }
    if(t<m) return F.a[t][0];
    for(t-=m-1;t;t/=2){//t-=m-1为还要转移的次数
        if(t&1) res=M*res;
        M=M*M;
    }
    F=res*F;
    return F.a[m-1][0];
}
```







## 例题

**用 fib(n) 表示斐波那契数列的第 n 项，现在要求你求 fib(n)mod m，fib(1)=1，fib(2)=1。**



斐波那契的递推式是$f[n]=f[n-1]+f[n-2](n>2)$
我们构建转移矩阵A                    $A=\begin{bmatrix}0 &1 \\  1&1 \end{bmatrix}$

于是有$\begin{bmatrix}f[i-1]\\ f[i]\end{bmatrix}=A\cdot \begin{bmatrix}f[i-2]\\ f[i-1]\end{bmatrix}=\begin{bmatrix}0 &1 \\1&1 \end{bmatrix}\begin{bmatrix}f[i-2]\\ f[i-1]\end{bmatrix}$

当然矩阵的数的位置可以调换（不唯一），编码时应注意

> 这样就能将复杂度优化为$O(2^3lgn)$



```c
#include<iostream>
#include<string.h>
using namespace std;

const int maxn=101;
const int maxm=101;
int mod;//需要时使用

struct Matrix{
    int n,m;//行 列
    int a[maxn][maxm];//注意函数内数组大小不能超过500000int
    void Clear(){
        n=m=0;
        memset(a,0,sizeof(a));
    }

    Matrix operator +(const Matrix &b) const{
        Matrix tmp;
        tmp.n=n; tmp.m=m;
        for(int i=0;i<n;++i)
            for(int j=0;j<m;++j)
                tmp.a[i][j]=a[i][j]+b.a[i][j];
        return tmp;
    }

    Matrix operator -(const Matrix &b) const{
        Matrix tmp;
        tmp.n=n; tmp.m=m;
        for(int i=0;i<n;++i)
            for(int j=0;j<m;++j)
                tmp.a[i][j]=a[i][j]-b.a[i][j];
        return tmp;
    }

    Matrix operator *(const Matrix &b) const{
        Matrix tmp;
        tmp.Clear();
        tmp.n=n; tmp.m=b.m;
        for(int i=0;i<n;++i)
            for(int j=0;j<b.m;++j)
                for(int k=0;k<m;++k){
                    tmp.a[i][j]+=a[i][k]*b.a[k][j]%mod;
                    tmp.a[i][j]%=mod;
                }
        return tmp;
    }
};

int solve(int a[],int b[],int m,int t){
    //a是常系数数组 b是初值数组 m是数组大小 t是要求解的项数 从第0项开始 所以共有t+1项
    //m为已知递推式的阶数
    //输出函数在第t项的值f(t)
    //调用矩阵类
    Matrix M,F,res;//M是辅助常数矩阵 F是转移矩阵
    M.Clear(),F.Clear(),res.Clear();
    M.n=M.m=m;
    res.n=res.m=m;
    F.n=m,F.m=1;
    for(int i=0;i<m-1;++i)
        M.a[i][i+1]=1;
    for(int i=0;i<m;++i){
        M.a[m-1][i]=a[i];//a[i]先存小项的系数 即递推式最靠末尾的系数
        F.a[i][0]=b[i];//b[i]先存小项的值 即f0(通常)
        res.a[i][i]=1;//初始化为单位矩阵
    }
    if(t<m) return F.a[t][0];
    for(t-=m-1;t;t/=2){//t-=m-1为还要转移的次数
        if(t&1) res=M*res;
        M=M*M;
    }
    F=res*F;//res为最后的常数矩阵
    return F.a[m-1][0];
}


int main()
{
    mod=1000000007;
    int t;
    int a[2],b[2];
    a[0]=a[1]=1;
    b[0]=0;
    b[1]=1;
    while(cin>>t&&t!=-1)
    {
        cout<<solve(a,b,2,t)<<endl;
    }
    return 0;
}

```



## 线性基



### 高斯消元_浮点型

时间复杂度   $O(n^3)$ 

```c++
//高斯消元 时间复杂度O(n^3) 使用浮点数计算
#include<bits/stdc++.h>
using namespace std;
const double eps=1e-8;
const int maxn=101;
double det;//行列式的值

//也可以避免实数运算 使用辗转相减的方法 多一个log的时间复杂度
//特别适合于行列式求值 取模操作 (除法要求逆元)
//但这里不支持
inline int solve(double a[][maxn],bool l[],double ans[],const int &n,const int &m){
    //a为方程组对应的矩阵
    //l,ans存储解  l[]表示是否为自由变元 1表示不是 0表示是
    //n为未知数的个数 m为方程的个数
    //如果无解返回-1 否则返回自由变元数
    int res=0,r=0;//r为第几行 res为自由变元数
    for(int i=0;i<n;++i) l[i]=false;//开始都是自由变元

    for(int i=0;i<n;++i){//枚举列

        for(int j=r;j<m;++j)//枚举行
            if(fabs(a[j][i])>eps){
                //找到当前列下从第r行开始第一个不为零的元素并交换到第r行
                //如果一直为0 则下面会continue res++ 即自由变元+1
                //如果有不为0的 把它调上来（交换行）
                for(int k=i;k<=n;++k)//第j行和第r行交换 因为a[j][i]!=0
                    swap(a[j][k],a[r][k]);
                break;
            }

        //从a[r][i]这一个元素开始 往下的每个元素都是0了 所以这个元素xi是自由变元 i+1跳过即可
        if(fabs(a[r][i])<eps){
            ++res;
            continue;
        }

        for(int j=0;j<m;++j)//j是行数 从第一行（j=0）开始  让上三角更简洁 这样后面求ans就没有必要从下往上了
            if(j!=r && fabs(a[j][i])>eps){
                double tmp=a[j][i]/a[r][i];
                for(int k=i;k<=n;++k)//这里从i开始就行了 ; 若用最小公倍数法，这里要注意从0开始
                    a[j][k]-=tmp*a[r][k];
            }
        l[i]=true,++r;
    }

    det=1;
    for(int i=0;i<n;++i)
        det*=a[i][i];

    //检查是否无解
    for(int i=n-res;i<m;++i){
        if(fabs(a[i][n])>eps) return -1;
    }

    //输出阶梯型矩阵
//    for(int j=0;j<m;++j){
//        for(int k=0;k<=n;++k){
//            cout<<a[j][k]<<' ';
//        }
//        cout<<endl;
//    }


    //下面求结果
    for(int i=0;i<n;++i){
        if(l[i])//不是自由变元
            for(int j=0;j<n;++j)
                if(fabs(a[j][i])>eps)//最后更新的是ans 因为到该数下面全为0了 
                        ans[i]=a[j][n]/a[j][i];
    }
    return res;//返回自由变元数
}

int main()
{
    ios::sync_with_stdio(false);
    double a[maxn][maxn];
    bool l[maxn];
    double ans[maxn];
    int n,m,res;
    int t;
    cin>>t;
    while(t--)
    {
        memset(a,0,sizeof(a));
        cin>>n>>m;
        for(int j=0;j<m;++j)
            for(int i=0;i<=n;++i)
                cin>>a[j][i];

        res=solve(a,l,ans,n,m);
        if(res==-1){
            cout<<"无解"<<endl;
            cout<<"系数矩阵对应的行列式值为"<<det<<endl;
        }
        else if(res){
            cout<<"自由变元为:";
            for(int i=0;i<n;++i)
                if(!l[i]) cout<<'x'<<i<<"  ";
            cout<<"系数矩阵对应的行列式值为"<<det<<endl;
        }
        else{
            cout<<"解为";
            for(int i=0;i<n;++i)
                if(l[i]) cout<<'x'<<i<<':'<<ans[i]<<"  ";
            cout<<"系数矩阵对应的行列式值为"<<det<<endl;
        }
    }
    return 0;
}


```





### 高斯消元_整型

时间复杂度  $O(n^3)$ 

```c++
//高斯消元 时间复杂度O(n^3) 使用整型计算
//公倍数法 适用于只需要整数解、取模的情况
#include<iostream>
#include<math.h>
#include<set>
#include<cstdio>
#include<stdlib.h>
#include<string.h>
using namespace std;
const int maxn=101;
long long det;//行列式的值
int mod;

//特别适合于行列式求值 取模操作 (除法要求逆元)
//不化成对角而是上三角 因为涉及到取模 要保留一些信息
inline int solve(int a[][maxn],bool l[],int ans[],const int &n,const int &m){
    //a为方程组对应的矩阵
    //l,ans存储解  l[]表示是否为自由变元 1表示不是 0表示是
    //n为未知数的个数 m为方程的个数
    //如果无解返回-1 否则返回自由变元数
    int res=0,r=0;//r为第几行 res为自由变元数
    for(int i=0;i<n;++i) l[i]=false;//开始都是自由变元

    for(int i=0;i<n;++i){//枚举列
        for(int j=r;j<m;++j)//枚举行
            if(a[j][i]){
                //找到当前列下从第r行开始第一个不为零的元素并交换到第r行
                //如果一直为0 则下面会continue res++ 即自由变元+1
                //如果有不为0的 把它调上来（交换行）
                for(int k=i;k<=n;++k)//第j行和第r行交换 因为a[j][i]!=0
                    swap(a[j][k],a[r][k]);
                break;
            }

        //从a[r][i]这一个元素开始 往下的每个元素都是0了 所以这个元素xi是自由变元 i+1跳过即可
        if(a[r][i]==0){
            ++res;
            continue;
        }

        for(int j=0;j<m;++j)//j是行数 从0开始 是对角 不是上三角 这样不要从下往上回代
            //j=r+1 则下面回代
            if(j!=r&&a[j][i]){
                int ta=a[r][i];
                int tb=a[j][i];
                for(int k=0;k<=n;++k){//公倍数法这里要从0开始
                    a[j][k]=((a[j][k]*ta-a[r][k]*tb)%mod+mod)%mod;
                }
            }
        l[i]=true,++r;
    }

    //输出阶梯型矩阵
//    for(int j=0;j<m;++j){
//        for(int k=0;k<=n;++k){
//            cout<<a[j][k]<<' ';
//        }
//        cout<<endl;
//    }

    det=1;
    for(int i=0;i<n;++i)
        det=(det*a[i][i])%mod;

    //检查是否无解
    for(int i=n-res;i<m;++i){
        if(a[i][n]) return -1;
    }

    //对角法 不用回代
    for(int i=0;i<n;++i){
        if(l[i]){
            for(int j=0;j<n;++j){
                if(a[j][i]>0){//最后更新的是ans 因为到该数下面全为0了
                    //枚举求值 因为题目要求最短的序列 即x尽量小 所以有满足的直接break就行
                    //这里可能出现多解的情况 一般情况是解同余式ax同余c(mod m)
                    for(ans[i]=0;ans[i]<mod;++ans[i]) if((ans[i]*a[j][i]+mod)%mod==a[j][n]) break;
                }
            }
        }
    }
    //下面求结果 不考虑自由变元    回代法
//    for(int i=n-1;i>=0;--i){
//        int temp=a[i][n];
//        for(int j=i+1;j<n;++j){
//            if(a[i][j]){
//                temp-=(a[i][j]*ans[j])%mod;
//                temp=(temp%mod+mod)%mod;
//            }
//        }
//        //枚举求值 因为题目要求最短的序列 即x尽量小 所以有满足的直接break就行
//        //这里可能出现多解的情况 一般情况是解同余式ax同余c(mod m)
//        for(ans[i]=0;ans[i]<mod;++ans[i])
//            if((ans[i]*a[i][i]+mod)%mod==temp) break;
//    }
    return res;//返回自由变元数
}

int main()
{
    mod=400;
    ios::sync_with_stdio(false);
    bool l[maxn];
    int ans[maxn];
    int a[maxn][maxn];
    int n,m,res;
    while(cin>>n>>m)
    {
        for(int j=0;j<m;++j)
            for(int i=0;i<=n;++i)
                cin>>a[j][i];

        res=solve(a,l,ans,n,m);

        if(res==-1){
            cout<<"无解"<<endl;
            cout<<"系数矩阵对应的行列式值为"<<det<<endl;
        }
        else if(res){
            cout<<"自由变元为:";
            for(int i=0;i<n;++i)
                if(!l[i]) cout<<'x'<<i<<"  ";
            cout<<"系数矩阵对应的行列式值为"<<det<<endl;
        }
        else{
            cout<<"解为";
            for(int i=0;i<n;++i)
                if(l[i]) cout<<'x'<<i<<':'<<ans[i]<<"  ";
            cout<<"系数矩阵对应的行列式值为"<<det<<endl;
        }
    }
    return 0;
}


```





## 拉格朗日插值法

#### 推荐阅读    

https://www.zhihu.com/question/58333118/answer/262507694

http://ruanxingzhi.coding.me/File/%E6%96%87%E6%A1%A3/%E6%8B%89%E6%A0%BC%E6%9C%97%E6%97%A5%E6%8F%92%E5%80%BC%E6%B3%95.pdf

https://blog.csdn.net/qq_35649707/article/details/78018944



#### 定义

对某个多项式，给定$k+1$个取值点：
$$
(x_0,y_0),...,(x_k,y_k)
$$
其中$x$互不相同，则唯一确定一个多项式

由**拉格朗日插值法**得到该多项式为
$$
L(x)=\sum_{j=0}^ky_jl_j(x)
$$
其中每个$l_j(x)$为**拉格朗日基本多项式**（或称**插值基函数**），其表达式为
$$
l_j(x)=\prod_{i=0,i≠j}^k\frac{x-x_i}{x_j-x_i}
$$
拉格朗日基本多项式$l_j(x)$的特点是在$x_j$上取值为1，在其它的点$x_i,\ i≠j$  上取值为0

**时间复杂度**    求原函数的复杂度为$O(n^2)$，单次求值的复杂度为$O(n^2)$。 





#### 优点与缺点

​	拉格朗日插值法的公式结构整齐紧凑，在理论分析中十分方便，然而在计算中， 当插值点增加或减少一个时，所对应的基本多项式就需要全部重新计算，于是整个公式都会变化，非常繁琐。这时可以用重心拉格朗日插值法或牛顿插值法来代替。

​	此外，当插值点比较多的时候，拉格朗日插值多项式的次数可能会很高，因 此具有数值不稳定的特点，也就是说尽管在已知的几个点取到给定的数值，但在 附近却会和“实际上”的值之间有很大的偏差（**过拟合现象**）。这类现象也被称为龙格现象，解决的办法是分段用较低次数的插值多项式。 





#### 一些题目

- [bzoj4559:成绩比较](https://blog.csdn.net/qq_35649707/article/details/78018944#bzoj4559%E6%88%90%E7%BB%A9%E6%AF%94%E8%BE%83)
- [bzoj2655: calc](https://blog.csdn.net/qq_35649707/article/details/78018944#bzoj2655-calc)
- [bzoj3453:XLkxc](https://blog.csdn.net/qq_35649707/article/details/78018944#bzoj3453xlkxc)



















#### 用母函数求斐波那契的通项

https://blog.csdn.net/IcePrincess_1968/article/details/78955510





#### 用母函数求卡特兰数的通项

http://blog.miskcoo.com/2015/07/catalan-number#i-6

观察递推式
$$
f_n = \sum_{i = 0}^{n - 1} f_if_{n - i - 1}
$$
它的结果是$Catlan_{n}$，可以借助**生成函数**来证明。

定义序列 $Cat_n$ 的生成函数为 $C(z)$，再根据上面的递推式，可以列出方程 
$$
C(z) = zC^2(z) + 1
$$
解方程得
$$
C(z) = \frac{1 \pm \sqrt{1 - 4z}}{2z}
$$
因为$C(0)=C_0=1$，因此对上式求极限
$$
\begin{eqnarray*}
 \lim_{z \rightarrow 0^+} \frac{1 + \sqrt{1 - 4z}}{2z} &=& +\infty \\
\lim_{z \rightarrow 0^+} \frac{1 - \sqrt{1 - 4z}}{2z} &=& 1 
\end{eqnarray*}
$$
因此，取负号，$C(z) = \frac{1 - \sqrt{1 - 4z}}{2z}$

根据广义二项式定理展开$\sqrt{1-4z}$
$$
\sqrt{1-4z} = \left (1-4z \right )^{\frac{1}{2}} = \sum_{i=0}^{\infty}{\frac{1}{2} \choose i}(-4z)^i
$$
然后考虑第$n$项系数
$$
\begin{eqnarray*}
& &(-4)^n { \frac{1}{2} \choose n }\\
&=& (-4)^n \frac{\frac{1}{2}\cdot \left( \frac{1}{2} -1 \right)\cdot \left( \frac{1}{2} -2 \right)\cdots \left( \frac{1}{2} - n + 1 \right)}{1\cdot2\cdot3\cdots n}\\
&=& (-2)^n \frac{1\cdot (1-2)\cdot (1-4)\cdots (1 - 2n + 2)}{1\cdot2\cdot3\cdots n}\\
&=& (-2)^n \frac{1\cdot (1-2)\cdot (1-4)\cdots (1 - 2n + 2)}{1\cdot2\cdot3\cdots n}\\
&=& -\frac{1\cdot 3\cdot 5\cdots (2n - 3)}{1\cdot2\cdot3\cdots n}\cdot \frac{1\cdot 2\cdot 4\cdots 2n}{1\cdot 2 \cdot 3 \cdots n}\\
&=& -\frac{(2n)!}{(2n-1)(n!)^2}\\
&=& -\frac{1}{2n-1}{{2n} \choose n}
\end{eqnarray*}
$$
因此可以得到
$$
\begin{eqnarray*}
C(z) &=& \frac{1-\sqrt{1-4z}}{2z}\\
&=&\frac{1+\sum_{i=0}^{\infty}\frac{1}{2i-1}{{2i} \choose i} z^i}{2z}\\
&=&\frac{\sum_{i=1}^{\infty}\frac{1}{2i-1}{{2i} \choose i} z^{i-1}}{2}\\
&=&\sum_{i=0}^{\infty}\frac{2}{2i+1}{{2i+2} \choose {i+1}} z^i\\
&=&\sum_{i=0}^{\infty}\frac{1}{2}\cdot\frac{1}{2i+1}\cdot\frac{(2i+1)(2i+2)}{(i+1)^2}{{2i} \choose i} z^i\\
&=&\sum_{i=0}^{\infty}\frac{1}{n+1}{{2i} \choose i} z^i\\
\end{eqnarray*}
$$
证毕。同样得到了$C_n = \frac{1}{n+1}{{2n} \choose n}$ 特殊多项式在整点上的线性插值方法

http://blog.miskcoo.com/2015/08/special-polynomial-linear-interpolation## 自然幂和

![img](https://s1.ax2x.com/2018/07/09/oegmN.png) 



<补上原理>



```c++
#include<bits/stdc++.h>
using namespace std;
const int maxn=1e6+10;
const int mod=1e9+7;

typedef long long ll;
ll p[maxn];
ll fac[maxn];
ll inv[maxn];
ll fac_inv[maxn];

void init()
{
    fac[0]=1;
    inv[1]=1;
    fac_inv[0]=1;
    for(int i=1;i<maxn;++i){
        if(i!=1){
            inv[i]=(mod-mod/i)*(inv[mod%i])%mod;
        }
        fac[i]=fac[i-1]*i%mod;
        fac_inv[i]=fac_inv[i-1]*inv[i]%mod;
    }
}

ll fast_pow(ll a,ll b){
    ll res=1;
    while(b)
    {
        if(b&1) res=res*a%mod;
        b>>=1;
        a=a*a%mod;
    }
    return res;
}

ll solve_inv(ll x)
{
    return fast_pow(x,mod-2);
}

int main()
{
#ifndef ONLINE_JUDGE
    freopen("in.txt","r",stdin);
#endif
    ios::sync_with_stdio(false);
    ll n,k;
    init();

    cin>>n>>k;
    p[0]=0;
    for(int i=1;i<=k+2;++i){//k+2个点插k+1次多项式
        p[i]=(p[i-1]+fast_pow(i,k))%mod;//可以用积性函数优化到O(n)
    }
    if(n<=k+2){
        cout<<p[n]<<endl;
    }
    else{
        ll help=1;
        for(ll i=n-1;i>=(n-k-2);--i) help=help*i%mod; //help是分子
        ll ans=0;
        for(int i=1;i<=k+2;++i){
            ll res=1;
            res=(res*help%mod)*solve_inv(n-i)%mod;
//            cout<<"inv(n-i): "<<solve_inv(n-i)<<endl;
//            cout<<"res: "<<res<<endl;
            res=res*fac_inv[i-1]%mod;
            //cout<<"fac_inv[i-1]: "<<fac_inv[i-1]<<endl;
            res=res*fac_inv[k+2-i]%mod;
            //cout<<"fac_inv[k+2-i]: "<<fac_inv[k+2-i]<<endl;
            //判断符号
            if((k+2-i)&1) res=(-res+mod)%mod;
            res=res*p[i]%mod;
            ans=(ans+res)%mod;
        }
        cout<<ans<<endl;
    }
    return 0;
}
```

![img](https://s1.ax2x.com/2018/08/15/599brX.png) 



![img](https://s1.ax2x.com/2018/08/16/59zZUz.md.png) 





---



![img](https://s1.ax2x.com/2018/08/19/59Zueh.md.png) 

$m^{n-1} * gcd(A_i,m)  $



---

