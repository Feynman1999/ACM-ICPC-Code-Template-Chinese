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



