## bitset

$bitset$是STL库中的二进制容器，$bitset$可以看作$bool$数组，但优化了空间复杂度和时间复杂度，并且可以像整型一样按位操作。



**基本用法**

```c++
#include<bits/stdc++.h>
using namespace std;

bitset <50> ar1;
bitset <50> ar2(1023);//二进制位
bitset <50> ar3("10101001010010");

vector<bitset<50>> ar;

int main()
{
    ar.push_back(ar1);
    ar.push_back(ar2);
    ar.push_back(ar3);
    for(auto k:ar) cout<<k<<endl;
    //---------------------------
    cout<<ar1.size()<<endl;
    cout<<ar2.count()<<endl;//返回1的个数
    cout<<ar3.any()<<endl;//返回是否有1
    cout<<ar1.none()<<endl;//返回是否没有1
    cout<<ar2.test(0)<<endl;//返回第0位是不是1
    ar1.set();cout<<ar1<<endl;//全部置1
    ar2.set(49);cout<<ar2<<endl;//第49位设为1
    ar3.reset();cout<<ar3<<endl;//全部设为0
    ar1.reset(29);cout<<ar1<<endl;//第29位设为0
    ar1.flip();cout<<ar1<<endl;//全部翻转
    ar1.flip(29);cout<<ar1<<endl;//第29位翻转
    //ar2.to_ulong();
    //ar2.to_ullong();
    cout<<ar2.to_string()<<endl;
    return 0;
}

```

 

![img](https://s1.ax2x.com/2018/08/17/59MsN3.png) 





**例题1**  

一共有$n$个数，第 $i$ 个数$ x_i $可以取 $[a_i,b_i]$ 中任意值。 

设 $S = \sum{{x_i}^2}$，求 $S$ 种类数。 

$1≤n,ai,bi≤100 $

```c++
//经典题目 用bitset下标表示所有答案 暴力后1表示可能 0表示不能
#include<bits/stdc++.h>
using namespace std;

const int maxn=1e6+10;

bitset<maxn> s,t;

void add(int l,int r)
{
    t.reset();
    s<<=(l*l);
    t|=s;
    for(int i=l+1;i<=r;++i){
        s<<=((i<<1)-1);
        t|=s;
    }
    s=t;
}

int main()
{
    ios::sync_with_stdio(false);
    int n;
    cin>>n;
    int l,r;
    s.set(0);//开始只有0位置是1
    for(int i=0;i<n;++i){
        cin>>l>>r;
        add(l,r);
    }
    cout<<s.count()<<endl;
    return 0;
}

```





**例题2**  

在做$dp$转移的时候，有时候会需要将两个$dp$矩阵相乘，且矩阵的元素都是布尔型。

可以用$bitset$优化常数

```c++
//矩阵乘法
#include<bits/stdc++.h>
using namespace std;

const int N=1e3;

bitset<N> A[N],B[N],C[N];

void Multi(bitset<N> A[],bitset<N> B[],bitset<N> C[]){
    for(int i=0;i<N;++i)
        for(int j=0;j<N;++j)
            if(A[i][j]) C[i]|=B[j];
}

int main()
{
    ios::sync_with_stdio(false);
    for(int i=0;i<N/2;++i) A[i].set();
    for(int i=0;i<N;++i) B[i].set();
    Multi(A,B,C);
    //for(int i=0;i<N;++i) cout<<C[i]<<endl;
    return 0;
}
```

经典题：

http://codeforces.com/contest/781/problem/D

只能走图中两种路径，一种是$P$，一种是$B$，从1号结点出发，最远能走多少？要求走的方式为 $P, PB, PBBP, PBBPBPPB, PBBPBPPBBPPBPBBP\cdots\cdots$（将每一位取反接到后面）

**由于路径就$log$个，所以可以用矩阵乘法先计算出所有可达性矩阵。最后用set维护答案**

```c++
#include<bits/stdc++.h>
using namespace std;

typedef long long ll;

const int maxn=510;

bitset<maxn> bit[61][2][maxn];

int main()
{
    ios::sync_with_stdio(false);
    int n,m;
    cin>>n>>m;
    int u,v,w;
    for(int i=0;i<m;++i){
        cin>>u>>v>>w;
        u--;v--;
        bit[0][w][u][v]=1;
    }

    for(int k=1;k<=60;++k){
        for(int i=0;i<n;++i){
            for(int j=0;j<n;++j){
                if(bit[k-1][0][i][j]) bit[k][0][i]|=bit[k-1][1][j];
                if(bit[k-1][1][i][j]) bit[k][1][i]|=bit[k-1][0][j];
            }
        }
    }
    ll ans=0;
    set<int> s;s.insert(0);//0号结点出发
    for(int k=60,w=0;k>=0 && ans<=(ll)1e18;--k){
        set<int> ss;
        for(auto z:s){
            for(int j=0;j<n;++j){
                if(bit[k][w][z][j]) ss.insert(j);
            }
        }
        if(!ss.empty()){
            s=ss;
            w^=1;
            ans+=(1LL<<k);
        }
    }
    if(ans>(ll)1e18) ans=-1;
    cout<<ans<<endl;
    return 0;
}
```





**例题3**   bzoj 3687  

求一个集合所有子集的算术和的异或

元素个数$n\le 1000$，所有数是正整数，总和不超过$2000000$。 

**DP式子**      $dp[i][j]=dp[i-1][j-a[i]]+dp[i-1][j]$ 

**本题只要求和的异或，所以对于同一个和k，若有偶数个相当于没有，奇数个则计数。可以用bitset移位+异或即可** 

如果是取模操作，模数较小如2000，数很多如2e6，同样的思路，只是左移改为循环左移 (2017四川省赛k)

```c++
//子集和的异或 ∑ ai ≤ 2000000
#include<bits/stdc++.h>
using namespace std;

const int maxn=2e6+10;

bitset<maxn> f;

int main()
{
    freopen("in.txt","r",stdin);
    ios::sync_with_stdio(false);
    int n,a;
    cin>>n;
    f.set(0);
    for(int i=0;i<n;++i){
        cin>>a;
        f ^=(f << a);
        //f ^= (f<<a | f>>(size-a) );//循环左移
    }
    int ans=0;
    //cout<<clock()<<endl;
    for(int i=0;i<maxn;++i){
        if(f[i]) ans^=i;
    }
    cout<<ans<<endl;
    return 0;
}
```

