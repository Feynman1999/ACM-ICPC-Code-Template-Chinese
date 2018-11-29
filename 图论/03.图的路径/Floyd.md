## Floyd

**解决任意两点间的最短路径，可以正确处理有向图或负权的最短路径问题，同时也被用于计算有向图的传递闭包。**

设$d[i][j][k]$为从$i$到$j$只以$1\sim k$中的结点为中间结点的最短路径的长度，则：

* 若最短距离经过k，那么$d[i][j][k]=d[i][k][k-1]+d[k][j][k-1]$ 
* 若最短距离不经过点k，那么$d[i][j][k]=d[i][j][k-1]$

因此，$d[i][j][k]=min(d[i][j][k-1],d[i][k][k-1]+d[k][j][k-1])$

将k放在外层循环即可。



**时间复杂度**  $O(n^3)$

**空间复杂度**  $O(n^2)$



使用b数组辅助记录信息，可以用来求解路径



```c++
#include<bits/stdc++.h>
using namespace std;

const int maxn=1e2+10;
const int inf=0x3f3f3f3f;

int n,m;
int d[maxn][maxn];
int b[maxn][maxn];//辅助 可以用来dfs求路径

int main()
{
    //freopen("in.txt","r",stdin);
    ios::sync_with_stdio(false);
    cin>>n>>m;
    //cout<<n<<" "<<m<<endl;
    for(int i=1;i<=n;++i){
        for(int j=1;j<=n;++j){
            d[i][j]=inf;
            b[i][j]=j;
        }
    }
    for(int i=1;i<=m;++i){
        int t1,t2,t3;
        cin>>t1>>t2>>t3;
        if(t3<d[t1][t2]){
            d[t1][t2]=t3;
            d[t2][t1]=t3;
        }
    }
    for(int k=1;k<=n;++k){
        for(int i=1;i<=n;++i){
            for(int j=1;j<=n;++j){
                if(d[i][j]>=d[i][k]+d[k][j]){
                    //cout<<"***"<<endl;
                    d[i][j]=d[i][k]+d[k][j];
                    b[i][j]=b[i][k];
                }
            }
        }
    }
    //cout<<"**"<<endl;
    for(int i=1;i<=n;++i){
        for(int j=1;j<=n;++j){
            if(i==j) cout<<0<<" ";
            else{
                cout<<d[i][j]<<" ";
            }
        }
        cout<<endl;
    }
    return 0;
}

```

