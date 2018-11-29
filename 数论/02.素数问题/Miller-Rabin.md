## Miller-Rabin素数判定

设$n$是奇素数，记$n-1=2^kq$，$q$是奇数，对不被$n$整除的某个$a$，如果下述**两个条件都成立**，则$n$一定是合数：（但有不成立的也不一定是素数）

- $a^q \not\equiv 1 \ (mod \ n)$
- 对所有的$i=0,1,2,...,k-1 ,\quad a^{2^iq}\not\equiv -1\ (mod \ n)$

和费马小定理测试相比，$miller\_rabin$测试不存在“卡米歇尔型数”，因为可以保证，如果$n$是奇合数，则1~n-1之间至少有75%的数可作为$miller\_rabin$的证据。即这些数作为$a$时，可以说明其合数性。

**换句话说，每个合数都有许多证据来说明它的合数性。**

例如随机选取$a$的100个值，若其中没有$n$的$miller\_rabin$证据，则$n$是合数的概率小于$0.25^{100}$，即认为是素数



**时间复杂度**  $O(log^2n)$



**注意看要不要使用$unsigned \ long \ long$**  

```c++
//O(lognlogn) miller_rabin素性测试
#include<bits/stdc++.h>
using namespace std;

typedef long long ll;

ll mod_mul(ll a,ll b,ll c){//a*b %c 乘法改加法 防止超long long
    ll res=0;
    a=a%c;
    while(b)
    {
        if(b&1) res=(res+a)%c;
        b>>=1;
        a=(a+a)%c;
    }
    return res;
}

ll fast_exp(ll a,ll b,ll c){
    ll res=1;
    a=a%c;
    while(b)
    {
        if(b&1) res=mod_mul(res,a,c);
        b>>=1;
        a=mod_mul(a,a,c);
    }
    return res;
}
//
//ll fast_exp(ll a,ll b,ll c){//a^b %c
//    ll res=1;
//    a=a%c;
//    while(b>0)
//    {
//        if(b&1) res=(a*res)%c;
//        b>>=1;
//        a=(a*a)%c;
//    }
//    return res;
//}

bool test(ll n,ll a,ll d)
{//d=n-1
    if(!(n&1)) return false;
    while(!(d&1))  d>>=1;//将d分解为奇数 至少有一个2因子，所以d!=n-1
    ll t=fast_exp(a,d,n);
    while(d!=n-1 && t!=1 && t!=n-1){
        t=mod_mul(t,t,n);
        d<<=1;
    }
    return ((t==n-1) || (d&1) ==1 );//两个条件都不成立则一定是合数; 若两个有一个成立，且多次，对于合数来说，可能性极小，所以可以认为是素数
}

bool is_prime(ll n){
    if(n<2) return false;
    if(n==2) return true;
    ll a[30];
    srand(time(0));
    for(int i=0;i<7;++i) {//测试7次
        a[i]=rand()%(n-2)+2;//取[2,n-1]随机数
        if(!test(n,a[i],n-1)) return false;//找到证据说明是合数
    }
    return true;//如果上面所有测试都是true
}

int main()
{
    int t;
    cin>>t;
    while(t--)
    {
        ll a;
        cin>>a;
        cout<<(is_prime(a) ? "Yes": "No")<<endl;
    }
    return 0;
}
```



