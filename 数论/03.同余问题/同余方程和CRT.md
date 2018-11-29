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




