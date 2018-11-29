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



