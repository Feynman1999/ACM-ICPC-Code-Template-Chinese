## Dijkstra

##### 注意不能有负权边，否则松弛失效

* **主要思想**：维护一个从$V_1$到其他各点的最短距离D数组，初始值就是$v_1$到其它各点的距离。由于边权非负，那么现在D数组中距离的最小值（比如到$V_x$） ，那么从$v_1$到$v_x$的最短距离一定就是这个最小值了。因为若再经过其它点，只会使得距离变大（边权非负）。**然后**，从$v_1$到$v_x$再到其余各点的距离更新D数组，也就是**松弛操作。**每一次这样的过程都会确定一个最短路径，所以循环n次即可。

- 邻接矩阵
- 时间复杂度  $O(N^2)$



```c++
//求v1到其他点最短路径中longest path
//未优化O(N^2)
#include<iostream>
#include<string.h>
#include<cstdio>
#include<stdlib.h>
using namespace std;

const int maxn=1000,inf=0x3f3f3f3f;

int dis[maxn];//记录从源点到某一点的最短距离
int G[maxn][maxn],ans;
bool v[maxn];//标记是否求得到某一点的最短距离

int n;

void dijkstra()
{
    for(int i=1;i<=n;++i) dis[i]=inf;
    dis[1]=0;
    memset(v,0,sizeof(v));
    for(int i=1;i<=n;++i){
        int mark=-1,mindist=inf;
        for(int j=1;j<=n;++j){
            //printf("%d ",dis[j]);
            if(!v[j]&&dis[j]<mindist){
                mindist=dis[j];
                mark=j;
            }
        }
        //printf("\n%d\n",mark);
        v[mark]=1;
        ans=max(ans,mindist);
        for(int j=1;j<=n;++j){//松弛
            if(!v[j]&&dis[mark]+G[mark][j]<dis[j]){
                dis[j]=dis[mark]+G[mark][j];
            }
        }
    }
}

int main()
{
    //ios::sync_with_stdio(false);
    string s;
    s.resize(10);
    scanf("%d",&n);
    ans=-inf;
    for(int i=1;i<=n;++i){
        G[i][i]=0;
    }
    for(int i=2;i<=n;++i){
        for(int j=1;j<i;++j){
            scanf("%s",&s[0]);
            //printf("%s\n",s.c_str());
            if(s[0]=='x') G[i][j]=inf;
            else G[i][j]=atoi(s.c_str());//printf("%d\n",G[i][j]);
            G[j][i]=G[i][j];
        }
    }
    dijkstra();
    printf("%d\n",ans);
    return 0;
}
```







* 邻接表

- 使用**优先队列**优化
- $O(Nlog(N)+M)$     稀疏图效果明显

```c++
//求v1到其他点最短路径中longest path
//优先队列优化O(Nlog(N)+M)
#include<iostream>
#include<string.h>
#include<cstdio>
#include<stdlib.h>
#include<queue>
using namespace std;
#define mp make_pair

typedef pair<int,int> node;
const int maxn=1000,inf=0x3f3f3f3f;

vector<node> G[maxn];//first 点的编号 second 距离

void addEdge(int u,int v,int w){
    G[u].push_back(mp(v,w));
    G[v].push_back(mp(u,w));
}

int dis[maxn];//记录从源点到某一点的最短距离
int n;//点数
int ans;

void dijkstra(int s)
{
    priority_queue<node,vector<node>,greater<node> > que;//小根堆 默认比较first
    //first存起点到该点最短距离 second是点的编号
    for(int i=1;i<=n;++i) dis[i]=inf;
    dis[s]=0;
    que.push(mp(0,s));//起点
    while(!que.empty())
    {
        node p=que.top();que.pop();
        int v=p.second;//顶点的编号
        if(dis[v]<p.first) continue;
        //否则更新
        ans=max(ans,p.first);
        for(int i=0;i<G[v].size();++i){//松弛
            node e=G[v][i];
            if(dis[v]+e.second<dis[e.first]){
                dis[e.first]=dis[v]+e.second;
                que.push(mp(dis[e.first],e.first));
            }
        }
    }
}

int main()
{
    //ios::sync_with_stdio(false);
    string s;
    s.resize(10);
    scanf("%d",&n);
    ans=-inf;
    for(int i=2;i<=n;++i){
        for(int j=1;j<i;++j){
            scanf("%s",&s[0]);
            //printf("%s\n",s.c_str());
            if(s[0]=='x') continue;
            else addEdge(i,j,atoi(s.c_str()));
        }
    }
    dijkstra(1);
    printf("%d\n",ans);
    return 0;
}
```





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

## SPFA

SPFA算法其实是Bellman-Ford算法的一种优化版本 

**SPFA用来求单源最短路（可以为负权值），但不能存在负权回路**

用数组dist记录每个结点的最短路径估计值，用邻接表来存储图g。

**思路**：设立一个**队列**用来保存待优化的结点，优化时，每次取出队首结点u，并用u点当前的最短路径估计值对u点所指向的结点v进行松弛操作，如果v点的最短路径估计值有所调整，且若v点不在当前的队列中，就将v点放入队尾。这样不断从队列中取出结点来进行松弛操作，直至队列为空。

由于图中不存在**负权回路**，所以每个结点都有最短路径值。

$dist[i]$数组表示源点$src$到$i$的最短距离



**时间复杂度**    一般为$O(常数*m)$    最坏为$O(n*m)$

 

```c++
#include<bits/stdc++.h>
using namespace std;
typedef pair<int,int>  pa;

const int maxn=1e5+10;

int n,m,src,aim;
vector<pa> g[maxn];

int dist[maxn];
bool inQue[maxn];

queue<int> que;

void spfa()
{
    memset(dist,63,sizeof(dist));//63~1e9
    dist[src]=0;
    while(!que.empty()) que.pop();
    que.push(src);
    inQue[src]=true;
    while(!que.empty())
    {
        int u=que.front();
        que.pop();
        for(int i=0;i<g[u].size();++i){
            if(dist[u]+g[u][i].second<dist[g[u][i].first]){
                dist[g[u][i].first]=dist[u]+g[u][i].second;
                if(!inQue[g[u][i].first]){
                    inQue[g[u][i].first]=true;
                    que.push(g[u][i].first);
                }
            }
        }
        inQue[u]=false;
    }
}

int main()
{
    //freopen("in.txt","r",stdin);
    ios::sync_with_stdio(false);
    cin>>n>>m>>src>>aim;
    for(int i=0;i<m;++i){
        int t1,t2,t3;
        cin>>t1>>t2>>t3;
        pa tp1=make_pair(t2,t3);
        pa tp2=make_pair(t1,t3);
        g[t1].push_back(tp1);
        g[t2].push_back(tp2);
    }
    spfa();

    cout<<dist[aim]<<endl;
    return 0;
}

```

## 哈密顿回路

![img](https://s1.ax2x.com/2018/09/02/5Bt3q3.png)

![img](https://s1.ax2x.com/2018/09/02/5Bt9dK.png)



[HDU - 4337 ](https://vjudge.net/problem/30660/origin)      题目保证：every knight has at least (N + 1) / 2 other knights as his close friends.

```c++
#include<bits/stdc++.h>
using namespace std;
typedef long long ll;

const int maxn = 1000;
const int inf = 0x3f3f3f3f;

int n,m;
bool G[maxn][maxn];
int x[maxn];//记录回路中的第i个点

//下标从0开始
void hamilton()
{
    int k;
    bool s[maxn];
    for(int i=0;i<n;i++){
        x[i] = -1;
        s[i] = false;
    }
    k = 1;
    s[0] = true;
    x[0] = 0;
    while(k >= 0){
        x[k]++;
        while(x[k] < n){
            if(!s[x[k]] && G[x[k-1]][x[k]]) break;
            else x[k]++;
        }
        if((x[k] < n) && (k!= n-1)){
            s[x[k++]] = true;
        }
        else if((x[k] < n) && k==n-1 && G[x[k]][x[0]]) break;
        else{
            x[k] = -1;
            k--;
            s[x[k]] = false;
        }
    }
}

int main()
{
    while(scanf("%d%d",&n,&m) != EOF){
        memset(G,0,sizeof(G));
        memset(x,0,sizeof(x));
        for(int i=0;i<m;i++){
            int u,v;
            scanf("%d%d",&u,&v);u--;v--;
            G[u][v]=G[v][u]=true;
        }

        hamilton();

        for(int i=0;i<n;i++){
            if(i == n-1) printf("%d\n",x[i]+1);
            else printf("%d ",x[i]+1);
        }
    }
    return 0;
}

```





## 最短哈密顿回路

状压DP即可


## DAG最短路

## 欧拉回路

















**多笔画问题**

一个图多笔画需要的次数为$max(\frac{d}{2},1)$    $d$表示度数为奇数的点的个数  （一定是2的倍数）

当$d=0,2$时可以一笔画           $d=0$为欧拉回路，$d=1$为欧拉路径。

## 可持久化堆优化k短路

wmr

```c++
/*
可持久化堆优化k短路
求最短路径树：O(nlogn)
求k短路：O(mlogm+klogk)
空间复杂度：O(mlogm)，但具体开数组要注意常数，要看执行merge操作的次数
*/
#include<bits/stdc++.h>
using namespace std;
const int maxn=100000,maxm=200000,maxsize=3000000;//maxsize是堆的最大个数
const long long inf=1000000000000000LL;
int a[maxn+5];
long long s[maxn+5];
int n,k,m,len,tot;
int S,T;
vector<int> g1[maxn+5];
int mi[maxn+5],mx[maxn+5];
int head[maxn+5],nx[maxm+5];
long long dis[maxn+5];
int pos[maxn+5],rt[maxn+5];
struct Edge
{
    int to;
    long long w;
}e[maxm+5];
struct node
{
    /*
    u是当前堆顶的节点编号
    key是当前堆顶对应边的权值,边的权值定义为：走这条边要多绕多少路
    l,r分别是堆左右儿子的地址
    */
    int u;
    long long key;
    int l,r;
}H[maxsize+5];
struct heapnode
{
    /*
    求k短路时候用到的数据结构
    len表示1~倒数第二条边的边权和
    root表示倒数第二条边的tail的H[tail]，其中堆顶就是最后一条边
    */
    long long len;
    int root;
    bool operator < (const heapnode& x) const
    {
        return len+H[root].key>x.len+H[x.root].key;
    }
};
priority_queue<heapnode> q;
void addedge(int u,int v,long long w)
{

    //printf("%d %d %lld\n",u,v,w);
    e[++len]={v,w};
    nx[len]=head[u];
    head[u]=len;
}
void dfs(int k,int fa)
{
    s[k]=a[k];
    bool flag=0;
    mi[k]=n+1;
    mx[k]=0;
    for(int i=0;i<g1[k].size();++i)
        if(g1[k][i]!=fa)
        {
            flag=1;
            dfs(g1[k][i],k);
            s[k]+=s[g1[k][i]];
            mi[k]=min(mi[k],mi[g1[k][i]]);
            mx[k]=max(mx[k],mx[g1[k][i]]);
        }
    if(!flag) mi[k]=mx[k]=++m;
}
int newnode(int u,long long key)
{
    ++tot;
    H[tot]={u,key,0,0};
    return tot;
}
int merge(int u,int v)
{
    /*
    merge两个堆u和v
    这里是采用随机堆，方便合并，方便持久化
    */
    if(!u) return v;
    if(!v) return u;
    if(H[v].key<H[u].key) swap(u,v);
    int k=++tot;
    H[k]=H[u];
    if(rand()%2) H[k].l=merge(H[k].l,v);
    else H[k].r=merge(H[k].r,v);
    return k;
}
void Kshort()
{
    /*
    求k短路
    */
    dis[T]=0;
    for(int i=0;i<T;++i) dis[i]=inf;
    tot=0;
    for(int i=m-1;i>=0;--i)
    {
        /*
        DAG图求最短路径树
        */
        int fa=0;
        for(int j=head[i];j!=-1;j=nx[j])
            if(dis[i]>e[j].w+dis[e[j].to])
            {
                dis[i]=e[j].w+dis[e[j].to];
                pos[i]=j;
                fa=e[j].to;
            }
        rt[i]=rt[fa];
        for(int j=head[i];j!=-1;j=nx[j])
            if(j!=pos[i])
            {
                //printf("ce : %d %d\n",i,e[j].to);
                rt[i]=merge(rt[i],newnode(e[j].to,e[j].w+dis[e[j].to]-dis[i]));
            }
    }
    //printf("tot : %d\n",tot);
    //printf("len : %d\n",len);
    //printf("m : %d\n",m);
    //for(int i=0;i<=T;++i) printf("%d : %lld %d\n",i,dis[i],pos[i]);
    printf("%lld\n",dis[S]+s[1]);
    heapnode now={0LL,rt[S]};
    if(now.root) q.push(now);
    while(--k&&!q.empty())
    {
        /*
        每次从优先队列队首取出最小的边集
        每个边集对对应一种合法的k短路走法
        有两种扩展方法
        第一种：将最后一条边换成所在堆的次小元素（相当于从堆里把堆顶删除）
        第二种：新加一条边，即从最后一条边继续往后走
        */
        now=q.top();
        q.pop();
        printf("%lld\n",now.len+H[now.root].key+dis[S]+s[1]);
        int id=merge(H[now.root].l,H[now.root].r);
        //printf("%d : %d %lld\n",id,H[id].u,H[id].key);
        if(id)
            q.push({now.len,id});
        now.len+=H[now.root].key;
        if(rt[H[now.root].u])
            q.push({now.len,rt[H[now.root].u]});
    }
}
int main()
{
    srand(time(0));
    scanf("%d%d",&n,&k);
    for(int i=1;i<=n;++i) scanf("%d",&a[i]);
    for(int i=1;i<n;++i)
    {
        int u,v;
        scanf("%d%d",&u,&v);
        g1[u].push_back(v);
        g1[v].push_back(u);
    }
    dfs(1,0);
    for(int i=0;i<=n+1;++i) head[i]=-1;
    for(int i=2;i<=n;++i) addedge(mi[i]-1,mx[i],-s[i]);
    for(int i=0;i<m;++i) addedge(i,i+1,0);
    S=0,T=m;
    Kshort();
    return 0;
}
```



