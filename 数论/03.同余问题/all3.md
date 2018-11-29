## N次剩余

给定$N,a,p$，求出$x^N\equiv a\ (mod \ p)$在模$p$意义下的所有解（其中$p$是素数） 



如果能找到原根$g$，则$\{1,2,..,.p-1\}$与$\{g^1,g^2,...,g^{p-1}\}$之间就建立了双射关系。

令$g^y=x,g^t=a$，$x^N\equiv a\ (mod \ p)$ ，则有
$$
g^{y*N}\equiv g^t  \quad (mod \ p)
$$
因为$p$是素数，所以方程左右都不会为0。原问题转化为：
$$
N*y\equiv t\ (mod \ (p-1))
$$
由于$N,p$已知，则上式为**解模线性方程**。

而$t$的值，由$g^t\equiv a \ (mod\ p)$，用解**离散对数**的方法求出。



**时间复杂度**  $O(\sqrt{p}log(\sqrt{p}))$  



```c++
//51Nod - 1038
//N次剩余 O(\sqrt{p}*大常数)
#include<bits/stdc++.h>
using namespace std;
typedef long long ll;

struct pli{
    ll first;
    int second;
    pli(){}
    pli (ll x_,int y_){
        first=x_;
        second=y_;
    }
    bool operator < (const pli &b) const {
        if(first==b.first) return second>b.second;//因为second更大的解更小，所以重载>
        return first<b.first;
    }
};

//ll mod_mul(ll a,ll b,ll c){//a*b %c 乘法改加法 防止超long long
//    ll res=0;
//    a=a%c;
//    assert(b>=0);
//    while(b)
//    {
//        if(b&1) res=(res+a)%c;
//        b>>=1;
//        a=(a+a)%c;
//    }
//    return res;
//}

ll fast_exp(ll a,ll b,ll c){
    ll res=1;
    a=a%c;
    assert(b>=0);
    while(b>0)
    {
        if(b&1) res=(a*res)%c;
        b=b>>1;
        a=(a*a)%c;
    }
    return res;
}

vector<ll> a;//p-1的所有质因子
bool g_test(ll g,ll p){
    for(ll i=0;i<a.size();++i)
        if(fast_exp(g,(p-1)/a[i],p)==1)//非素数时，要将p-1换为\varphi(p)
            return 0;
    return 1;
}

ll primitive_root(ll p)
{
    a.clear();
    ll tmp=p-1;//非素数时，要将p-1换为\varphi(p)
    for(ll i=2;i<=tmp/i;++i)
        if(tmp%i==0){
            a.push_back(i);
            while(tmp%i==0) tmp/=i;
        }
    if(tmp!=1) a.push_back(tmp);
    long long g=1;
    while(1)
    {
        if(g_test(g,p)) return g;
        ++g;
    }
}

ll gcd(ll a,ll b){
    if(b==0) return a;
    return gcd(b,a%b);
}

//a^x\equiv b (mod m)
ll bsgs(ll a,ll b,ll m)
{
    a%=m,b%=m;
    if(b==1) return 0;
    int cnt=0;
    ll t=1;
    for(ll g=gcd(a,m);g!=1;g=gcd(a,m)){
        if(b % g) return -1;
        m/=g , b/=g , t = t * a / g % m;//记录下要算逆元的值
        ++cnt;
        if(b==t) return cnt;
    }
    //cout<<a<<" "<<b<<" "<<m<<endl;
    //bsgs
    int M=int(sqrt(m)+1);//这里的m不等于参数中的值了
    ll base=b;
//    std::unordered_map<ll,int> hash;
//    for(int i=0;i!=M;++i){
//        hash[base]=i;//存的是大的编号
//        base=base*a%m;
//    }
//    base=fast_exp(a,M,m);//必要时要用快速乘
//    ll now=t;
//    for(int i=1;i<=M+1;++i){
//        now=now*base%m;//这里乘在左边了 相当于右边乘逆元
//        if(hash.count(now)) return i*M-hash[now]+cnt;
//    }
    pli hash[int(1e5)];
    for(int i=0;i!=M;++i){
        hash[i]=pli(base,i);
        base=base*a%m;
    }
    sort(hash,hash+M);//默认以first
    base=fast_exp(a,M,m);
    ll now=t;
    for(int i=1;i<=M+1;++i){
        now=now*base%m;
        //找大于等于now的值
        //注意下面M+10  这样可以保证解最小，若设为-1,则找不到解了
        int id=lower_bound(hash,hash+M,pli(now,M+10))-hash;//默认是first
        assert(id>=0 &&id<=M);
        if(id!=M && hash[id].first==now) return i*M-hash[id].second+cnt;//减去的编号second越大越好
    }
    return -1;
}

ll e_gcd(ll a,ll b,ll &x,ll &y)
{
    if(b==0){
        x=1;
        y=0;
        return a;
    }
    ll ans=e_gcd(b,a%b,x,y);
    ll temp=x;
    x=y;
    y=temp-a/b*y;
    return ans;
}

vector<int> residue(int p,int N,int a){
    int g=primitive_root(p);
    ll t=bsgs(g,a,p);
    vector<int> ans;
    if(a==0){
        ans.push_back(0);
        return ans;
    }
    if(t==-1) return ans;
    //解不定方程
    ll A=N,B=p-1,C=t,x,y;
    ll d=e_gcd(A,B,x,y);
    if(C % d !=0) return ans;
    x=x*(C/d)%B;
    ll delta=B/d;
    for(int i=0;i<d;++i){//一共d组解
        x=((x+delta)%B+B)%B;
        ans.push_back((int)fast_exp(g,x,p));
    }
    sort(ans.begin(),ans.end());
//unique的作用是“去掉”容器中相邻元素的重复元素（不一定要求数组有序），
//它会把重复的元素添加到容器末尾（所以数组大小并没有改变），而返回值是去重之后的尾地址
    ans.erase(unique(ans.begin(),ans.end()),ans.end());
    return ans;
}

int main()
{
    ios::sync_with_stdio(false);
    int t,p,N,A;
    //x^N=A (mod p)
    cin>>t;
    while(t--)
    {
        cin>>p>>N>>A;
        vector<int> ans=residue(p,N,A);
        if(ans.size()){
            for(int i=0;i<ans.size()-1;++i){
                cout<<ans[i]<<" ";
            }
            cout<<*ans.rbegin()<<endl;
        }
        else cout<<"No Solution"<<endl;
    }
    return 0;
}
```

## 二次互反律

**$a$模$p$的勒让德符号是**
$$
\left(\frac{a}{p}\right)=\left\{\begin{matrix}
1  \quad  若a是模p的二次剩余 \\  
-1 \quad 若a是模p的二次非剩余
\end{matrix}\right.
$$

**欧拉准则： ** 设$p$为素数，则（**直接用快速幂计算**）
$$
a^\frac{p-1}{2}\equiv  \left( \frac{a}{p} \right) \quad mod \ p
$$
**勒让德符号满足积性**
$$
\left(\frac{a}{p}\right)\left(\frac{b}{p}\right)=\left(\frac{ab}{p}\right)
$$



**定理  广义二次互反律**  设$a,b$为正奇数，则

![img](https://s1.ax2x.com/2018/08/16/59VdpO.png) 

![img](https://s1.ax2x.com/2018/08/16/59VjKq.png) 

---



使用二次互反律求解勒让德符号  （**当然也可以直接用欧拉准则，快速幂即可**）

**时间复杂度**  $O(logb)$



```c++
//poj 1808
#include<iostream>
#include<assert.h>
using namespace std;

typedef long long ll;

ll a,p;//求解勒让德符号(a/p)

ll Legendre(ll a,ll p)
{
    a=a%p;
    if(a<0) a+=p;
    int num=0;//将a的2次幂提出 保证是奇数
    while(a%2==0){
        a>>=1;
        num++;
    }
    ll ans1,ans2=1;//ans2是2的次幂的答案,默认为1
    if(a==1) ans1=1;
    else if(a==2){
        if(p%8==1||p%8==7) ans1=1;
        else ans1=-1;
    }
//    else if(a==p-1){
//        if(p%4==1) ans1=1;
//        else ans1=-1;
//    }
    else{
        if(p%4==3 && a%4==3) ans1=-Legendre(p,a);
        else ans1=Legendre(p,a);
    }
    if(num%2 &&(p%8==3||p%8==5)) ans2=-1;
    return ans1*ans2;
}

int main()
{
    int t;
    cin>>t;
    int T=0;
    while(t--)
    {
        T++;
        cin>>a>>p;
        cout<<"Scenario #"<<T<<":"<<endl<<Legendre(a,p)<<endl<<endl;
    }
    return 0;
}

```



## 二次剩余

**定理**  设$p$为一个奇素数，则恰有$\frac{p-1}{2}$个模$p$的二次剩余（即原根的偶次幂模$p$），且恰有$\frac{p-1}{2}$个模$p$的二次非剩余。



**$a$模$p$的勒让德符号是**
$$
\left(\frac{a}{p}\right)=\left\{\begin{matrix}
1  \quad  若a是模p的二次剩余 \\  
-1 \quad 若a是模p的二次非剩余
\end{matrix}\right.
$$
使用**二次互反律**可以快速计算一个数$a$是不是模$p$的二次剩余。

当然直接用**欧拉准则**来判别更方便

---

$ACM$中更常见的问题是，不仅判断是不是二次剩余，还要求解对应的底数。

给定$a,n$，其中$n$是素数，求$x^2\equiv a\ (mod \ n)$的最小整数解$x$。



**思路**  先判断是否有解，再根据剩余类特殊判断

输出满足同余式的较小解$x$  ，$-1$表示无解

取较小的辣个解  可以保证有两个解，且是对称的

即$x=[1,p-1],[2,p-2],...,[(p-1)/2,(p+1)/2]$对应的剩余是一样的



**时间复杂度**  $O(log^2n)$

(相当于求解勒让德符号的基础上，又给出了解，这里勒让德符号的判断直接使用欧拉准则)



```c++
#include<iostream>
using namespace std;

typedef long long ll;

ll fast_exp(ll a,ll b,ll c){
    ll res=1;
    a=a%c;
    while(b)
    {
        if(b&1) res=(res*a)%c;
        b>>=1;
        a=(a*a)%c;
    }
    return res;
}

//x^2 \equiv a (mod n)
ll modsqr(ll a,ll n){
    a=(a%n+n)%n;
    if(a==0) return 0;//这里考虑了解x=0的情况 一般只考虑[1,p-1]
    ll b,k,i,x;
    if(n==2) return a%n;
    if(fast_exp(a,(n-1)/2,n)!=1) return -1;
    //-----------------------------------------
    if(n%4==3) x=fast_exp(a,(n+1)/4,n);
    else{//求解n%4==1
        for(b=1;fast_exp(b,(n-1)/2,n)==1;b++);
        i=(n-1)/2;
        k=0;
        do{
            i/=2;
            k/=2;
            if((fast_exp(a,i,n)*(ll)fast_exp(b,k,n)+1)%n==0)  k+=(n-1)/2;
        }while(i%2==0);
        x=(fast_exp(a,(i+1)/2,n)*(ll)fast_exp(b,k/2,n))%n;
    }
    if(x * 2 >n) x=n-x;//取较小的辣个解  可以保证有两个解，且是对称的
    //即x=[1,p-1],[2,p-2],...,[(p-1)/2,(p+1)/2]对应的剩余是一样的
    return x;
}

int main()
{
    cout<<modsqr(0,48611)<<endl;
    return 0;
}
```





**特殊二次剩余**（比上面板子快常数倍）

当$p$模4余1时，求解$x^2\equiv -1 \ (mod \ p)$，可以直接使用二次剩余的板子，也可以使用随机算法。

即随机一个$a∈[1,p-1]$ ，求解$b\equiv a^{(p-1)/4}\ (mod\ p)$，可知$b^2\equiv (\frac{a}{p}) \ (mod\ p)$，即若选取的$a$不是$p$的二次剩余（有一半的概率），则求得的$b$即为解$x$。

由定理知有两个解$x_1,x_2$，且$x_1+x_2=p$。

```c++
//求解x^2 \equiv -1 (mod p)
ll ran(ll n){
    //要保证n%4=1
    assert(n%4==1);
    srand(time(0));
    for(;;){
        ll a=rand()*rand()%(n-1)+1;
        ll b=fast_exp(a,(n-1)/4,n);
        if(b*b%n==n-1) return b<=(n-1)/2 ? b:n-b;//若爆long long 则用快速乘
    }
}
```

## 单变元模线性方程（同余方程）

求解不定方程



```c++
#include<bits/stdc++.h>
using namespace std;

typedef long long ll;

//拓展欧几里得模板
ll e_gcd(ll a,ll b,ll &x,ll &y)
{
    if(b==0){
        x=1;
        y=0;
        return a;
    }
    ll ans=e_gcd(b,a%b,x,y);
    ll temp=x;
    x=y;
    y=temp-a/b*y;
    return ans;
}

//ax同余b(mod n) 输出[0,n) 中的解
vector<long long> line_mod_equation(long long a,long long b,long long n){
    long long x,y;
    long long d=e_gcd(a,n,x,y);
    vector<long long>ans;
    ans.clear();
    if(b%d==0){
        x=(x%n+n)%n;
        ans.push_back(x*(b/d)%(n/d));//最小正整数解
        for(long long i=1;i<d;++i)//mod n意义下有d(gcd?)个解
            ans.push_back((ans[0]+i*n/d)%n);
    }
    return ans;
}

vector<long long>ans;

int main()
{
    ios::sync_with_stdio(false);
    ll a,b,mod;
    cin>>a>>b>>mod;
    ans=line_mod_equation(a,b,mod);
    if(!ans.size()) cout<<"无解"<<endl;
    else{
        for(int i=0;i<ans.size();++i){
            cout<<ans[i]<<' ';
        }
    }
    return 0;
}

```







## 中国剩余定理（CRT） 同余方程组

求出方程组$x\equiv a_i(mod \ m_i) (0 \leqslant i <n) $ 的解x

其中$m_0,m_1,m_2,m_3...m_{n-1}$ 两两互质



**解：**

令$M_i=\prod_{j\neq i}m_j$     则有$(M_i,m_i)=1$   

故存在$p_i,q_i$ ，使得$M_i*p_i+m_i*q_i=1$ 

令  $e_i=M_ip_i$   ,    $p_i$即为$M_i$ 模$m_i$下的逆元          $(M_i,m_i)=1$   

则有
$$
e_i\equiv

\left\{\begin{matrix}
 0(mod\ m_j),j\neq i \\ 
 1(mod\ m_j),j=i
\end{matrix}\right.
$$
故$e_0a_0+e_1a_1+e_2a_2+...+e_{n-1}a_{n-1}$是方程的一个解

由**中国剩余定理**知，$[0\sim\prod_{i=0}^{n-1}m_i\}$ 中必有一解  

将上式模$\prod_{i=0}^{n-1}m_i$即可 



**时间复杂度**  $O(nlogm)$    $n$个方程

```c++
#include<bits/stdc++.h>
using namespace std;

typedef long long ll;

const int maxn=101;//方程个数

//拓展欧几里得模板
ll e_gcd(ll a,ll b,ll &x,ll &y)
{
    if(b==0){
        x=1;
        y=0;
        return a;
    }
    ll ans=e_gcd(b,a%b,x,y);
    ll temp=x;
    x=y;
    y=temp-a/b*y;
    return ans;
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

ll a[maxn];
ll m[maxn];
int n;

int main()
{
    ios::sync_with_stdio(false);
    cin>>n;
    for(int i=0;i<n;++i)
        cin>>a[i]>>m[i];
    cout<<"一个解为:"<<CRT(a,m,n)<<endl;
    return 0;
}

```




## 广义欧拉降幂

如何计算$5^{100000000000000}mod\ 12830603$?

如果12830603是素数，则直接使用费马小定理，可以将指数除以p-1，将余数作为幂计算即可。

但12830603=3571*3593，不是素数。

但但，$gcd(5,12830603)=1$，因此$5^{\varphi(12830603)}\equiv 1 (mod \ 12830603)$

计算得到$\varphi(12830603)=12823440$，因此只要把100..000除以12823440的余数作为指数。

注意这里要求5和12830603互质。

如果底数和模数不互质呢？有广义欧拉降幂公式，总结如下：
$$
\begin{align}
a^b\equiv _{mod \ p} \left\{\begin{matrix}
 a^{b\% \phi(p)}&  \quad gcd(a,p)=1 \\
a^b   \quad  & gcd(a,p)\neq 1,b<\phi(p) \\
a^{b\% \phi(p)+\phi(p)} &\quad   gcd(a,p)\neq 1,b\ge \phi(p)
\end{matrix}\right.
\end{align}
$$



**例题1**    求2的2的2的2的....次方模p ,p<1e7

我们对指数进行降幂，递归即可，可以保证在$log$步内会停止 

时间复杂度  $O(log_2p*\sqrt{p})$ 

```c++
#include<bits/stdc++.h>
using namespace std;

typedef long long ll;

//O sqrt(p) 求解欧拉函数
int phi(int x)
{
    int res=x;
    for(int i=2;i*i<=x;++i){
        if(x%i==0){
            res-=res/i;
            do{
                x/=i;
            }while(x%i==0);
        }
    }
    if(x>1) res-=res/x;
    return res;
}


ll fast_exp(ll a,ll b,ll c)
{
    ll res=1;
    a=a%c;
    assert(b>=0);
    while(b)
    {
        if(b&1){
            res=res*a%c;
        }
        a=a*a%c;
        b>>=1;
    }
    return res;
}


ll solve(ll p)//模p
{
    if(p==1) return 0;
    int ph=phi(p);
    return fast_exp(2,solve(ph)+ph,p);//广义欧拉公式 由于是无穷次幂 一定大于phi(p)
}

int main()
{
    ios::sync_with_stdio(false);
    int t;
    cin>>t;
    while(t--)
    {
        ll p;
        cin>>p;
        cout<<solve(p)<<endl;
    }
    return 0;
}
```





**例题2**   

计算$A^B mod C$ 

$(1<=A,C<=1000000000,1<=B<=10^{1000000}).$

``` c++
#include<iostream>
using namespace std;

typedef long long ll;

ll gcd(ll a,ll b)
{
    if(b==0) return a;
    return gcd(b,a%b);
}

//O sqrt(p) 求解欧拉函数
ll phi(ll x)
{
    ll res=x;
    for(int i=2;i*i<=x;++i){
        if(x%i==0){
            res-=res/i;
            do{
                x/=i;
            }while(x%i==0);
        }
    }
    if(x>1) res-=res/x;
    return res;
}

ll fast_exp(ll a,ll b,ll c)
{
    ll res=1;
    a=a%c;
    //assert(b>=0);
    while(b)
    {
        if(b&1){
            res=res*a%c;
        }
        a=a*a%c;
        b>>=1;
    }
    return res;
}

ll fun(string b,ll mod)
{
    ll res=0;
    for(int i=0;i<b.size();++i){
        res=res*10+b[i]-'0';
        res%=mod;
    }
    return res;
}

int main()
{
    ios::sync_with_stdio(false);
    ll a,c;
    string b;
    while(cin>>a>>b>>c)
    {
        if(b.size()<11) cout<<fast_exp(a,fun(b,1e12),c)<<endl;//fun(b,1e12) 相当于将b转为数字
        else if(gcd(a,c)==1) cout<<fast_exp(a,fun(b,phi(c)),c)<<endl;
        else cout<<fast_exp(a,fun(b,phi(c))+phi(c),c)<<endl;
    }
    return 0;
}

```



## 二分快速乘

```c++
ll mod_mul(ll a,ll b,ll c){//a*b %c 乘法改加法 防止超long long
    ll res=0;
    a=a%c;
    assert(b>=0);
    while(b)
    {
        if(b&1) res=(res+a)%c;
        b>>=1;
        a=(a+a)%c;
    }
    return res;
}
```





## 二分快速幂

```c++
typedef long long ll;

ll fast_exp(ll a,ll b,ll c){
    ll res=1;
    a=a%c;
    assert(b>=0);
    while(b>0)
    {
        if(b&1) res=(a*res)%c;
        b=b>>1;
        a=(a*a)%c;
    }
    return res;
}
```





## 矩阵快速幂求递推式(以fib为例)

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



```c++
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
    //f[0]=0 f[1]=1
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



## 离散对数

给定$a,b,m$，求$a^x\equiv b \ (mod \ m)$的解。

**Baby Step Giant Step：**   

首先解决$m=p$是质数的情况，我们设$x=A\left \lceil \sqrt{p} \right \rceil -B$，其中$0\le B< \left \lceil \sqrt{p} \right \rceil$， $0< A\le \left \lceil \sqrt{p} \right \rceil+1$，这样化简后的方程是
$$
a^{A\left \lceil \sqrt{p} \right \rceil}\equiv b\cdot a^B \quad (mod \ p)
$$
由于$A$和$B$值域都是$O(\sqrt{p})$级别的，所以可以先计算右边部分的值，存入$Hash$表，然后从小到大枚举$A$计算左边的值，在$Hash$表中查找。（当然，可以这样做的原因是一定存在$a$的逆元）即只要$gcd(a,m)=1$，上面的方法就是有效的。

**当$m$不是质数时**，我们要求解的是$a^x+km=b$，设$g=gcd(a,m)$，如果$g$不整除$b$，则无解，否则方程两边同除以$g$，得到$\frac{a}{g}a^{x-1}+k\frac{m}{g}=\frac{b}{g}$

这样便消去了一个因子，得到方程
$$
\frac{a}{g}a^{x-1}\equiv \frac{b}{g}\quad (mod \   \frac{m}{g})
$$
令${m}'=\frac{m}{g} ,\ b'=\frac{b}{g}(\frac{a}{g})^{-1}$， 得到
$$
a^{x'}\equiv b' \quad (mod \ m')
$$
得到的解加1即$x=x'+1$为原方程的解。

**但是**，进行一次这样的操作，新的方程不一定可以用bsgs求解，所以会进行多次。

如果中途出现$b'=1$则$x'=0$ 



**时间复杂度**  $O(\sqrt{m}log\sqrt{m})$        如果hash不用map则可以降低常数

求最小的$x$：

```c++
//SPOJ - MOD
//sqrt(m)*10  用map所以*10
//用二分常数较小*7大概
#include<bits/stdc++.h>
using namespace std;
typedef long long ll;

struct pli{
    ll first;
    int second;
    pli(){}
    pli (ll x_,int y_){
        first=x_;
        second=y_;
    }
    bool operator < (const pli &b) const {
        if(first==b.first) return second>b.second;//因为second更大的解更小，所以重载>
        return first<b.first;
    }
};

ll fast_exp(ll a,ll b,ll c){
    ll res=1;
    a=a%c;
    assert(b>=0);
    while(b>0)
    {
        if(b&1) res=(a*res)%c;
        b=b>>1;
        a=(a*a)%c;
    }
    return res;
}

ll gcd(ll a,ll b){
    if(b==0) return a;
    return gcd(b,a%b);
}

//a^x\equiv b (mod m)
ll bsgs(ll a,ll b,ll m)
{
    a%=m,b%=m;
    if(b==1) return 0;
    int cnt=0;
    ll t=1;
    for(ll g=gcd(a,m);g!=1;g=gcd(a,m)){
        if(b % g) return -1;
        m/=g , b/=g , t = t * a / g % m;//记录下要算逆元的值
        ++cnt;
        if(b==t) return cnt;
    }
    //cout<<a<<" "<<b<<" "<<m<<endl;
    //bsgs
    int M=int(sqrt(m)+1);//这里的m不等于参数中的值了
    ll base=b;
//    std::unordered_map<ll,int> hash;
//    for(int i=0;i!=M;++i){
//        hash[base]=i;//存的是大的编号，所以可以保证最小解
//        base=base*a%m;
//    }
//    base=fast_exp(a,M,m);//必要时要用快速乘
//    ll now=t;
//    for(int i=1;i<=M+1;++i){
//        now=now*base%m;//这里乘在左边了 相当于右边乘逆元
//        if(hash.count(now)) return i*M-hash[now]+cnt;
//    }
    pli hash[int(1e5)];
    for(int i=0;i!=M;++i){
        hash[i]=pli(base,i);
        base=base*a%m;
    }
    sort(hash,hash+M);//默认以first
    base=fast_exp(a,M,m);
    ll now=t;
    for(int i=1;i<=M+1;++i){
        now=now*base%m;
        //找大于等于now的值
        //注意下面M+10  这样可以保证解最小，若设为-1,则找不到解了
        int id=lower_bound(hash,hash+M,pli(now,M+10))-hash;//默认是first
        assert(id>=0 &&id<=M);
        if(id!=M && hash[id].first==now) return i*M-hash[id].second+cnt;//减去的编号second越大越好
    }
    return -1;
}

//10 1 7  1
int main()
{
    ll a,b,m;
    while(cin>>a>>m>>b,m)
    {
        ll ans=bsgs(a,b,m);
        if(ans==-1) cout<<"No Solution"<<endl;
        else{
            cout<<ans<<endl;
            assert(fast_exp(a,ans,m)==b%m);
        }
    }
    return 0;
}
```

## 逆元

**模质数**	   费马小定理

**模非质数**    拓展欧几里得

```c++
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

```





## 线性求逆元

以求阶乘为例 

**线性逆元原理：**

设$p=ki+b$ ，那么$ki+b\equiv 0  (mod p)$ 

两边同乘以$i^{-1}，b^{-1}$ 得到$kb^{-1}+i^{-1}\equiv 0 \  (mod p)$  

$i^{-1}\equiv -kb^{-1}$ 

$i^{-1}\equiv -\lfloor\frac{p}{i}\rfloor * (p\%i)^{-1}$

通常在前面加上p(即mod) 

` inv[i]=((mod-mod/i)*(inv[mod%i]))%mod;`

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
```

也可以用欧拉线性筛，但没必要