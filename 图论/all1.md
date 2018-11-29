## 邻接矩阵

以**Dijkstra**为例

时间复杂度	$O(N^2)$



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

## 边集数组

以**Kruskal（union_find优化）**求解最小生成树为例

时间复杂度	$O(MlogM)$



```c++
//求minispan_tree中longest edge
//O(MlogM)
#include<iostream>
#include<algorithm>
#include<cstdio>
using namespace std;
const int maxn=100010,inf=0x3f3f3f3f;

int n,m;
int ans;

struct Edge{
    int from,to,dist;
    Edge() {}//必须加上 否则新声明结构体时必须加参数
    Edge(int x,int y,int w):from(x),to(y),dist(w){}
    bool operator < (const Edge & e) const {//后面的const必须加上
        return (dist<e.dist);
    }
};

Edge edges[maxn];//这里有无参数初始化Edge型的变量  前面Edge结构体必须加上Edge() {}


int father[maxn];

//fa[i]为i所在连通块的代表
int Find(int x)
{
    return father[x]= father[x]==x? x: Find(father[x]);
}

void kruskal()
{
    sort(edges,edges+m);
    //初始化并查集
    for(int i=1;i<=n;++i) father[i]=i;

    for(int i=0;i<m;++i){
        int fa1=Find(edges[i].from);
        int fa2=Find(edges[i].to);
        if(fa1!=fa2){
            father[fa1]=fa2;
            ans=max(ans,edges[i].dist);
            //printf("(%d,%d) %d\n",edges[i].from,edges[i].to,edges[i].dist);
        }
    }
}

int main()
{
    //ios::sync_with_stdio(false);
    scanf("%d %d",&n,&m);
    ans=-inf;
    for(int i=0;i<m;++i){
        scanf("%d %d %d",&edges[i].from,&edges[i].to,&edges[i].dist);
    }
    kruskal();
    printf("%d\n",ans);
    return 0;
}
```

## 邻接表

以**Prim**求解最小生成树为例

时间复杂度	$O(N^2)$



- **vector中存每个点所连边的序号 通过边集数组索引**

```c++
//求minispan_tree中longest edge
//O(N^2)
//vector中存每个点所连边的序号 通过边集数组索引
#include<iostream>
#include<vector>
#include<cstdio>
using namespace std;

struct Edge{
    int from,to,dist;
    Edge(int u,int v,int w):from(u),to(v),dist(w){}
};

const int maxn=10010;
const int inf=0x3f3f3f3f;
int ans;

vector<Edge> edges;

vector<int> G[maxn];

int n,m;

void addEdge(int u,int v,int w){//无向图一次加进去两条边
    edges.push_back(Edge(u,v,w));
    edges.push_back(Edge(v,u,w));
    int size=edges.size();
    G[u].push_back(size-2);
    G[v].push_back(size-1);
}

int adjvex[maxn];//最小生成树上每个点的前驱
int lowcost[maxn];//前驱到这个点的距离,即目前最小生成树中到该点的最短距离
bool v[maxn];//是否在生成树中的标记

void Prim()
{
    for(int i=1;i<=n;++i) lowcost[i]=inf;
    lowcost[1]=0;
    for(int i=1;i<=n;++i) adjvex[i]=0,v[i]=0;
    adjvex[1]=1;
    for(int i=1;i<=n;++i){
        //for(int i=1;i<=n;++i) cout<<lowcost[i]<<' ';
        int minn=inf,mark=-1;
        for(int j=1;j<=n;++j){
            if(!v[j]&&lowcost[j]<minn){
                minn=lowcost[j];
                mark=j;
            }
        }
        if(mark==-1) break;
        //printf("(i:%d,%d,%d)\n",i,adjvex[mark],mark);
        ans=max(minn,ans);
        v[mark]=1;
        for(int j=0;j<G[mark].size();++j){//松弛
            if(!v[edges[G[mark][j]].to]&&edges[G[mark][j]].dist<lowcost[edges[G[mark][j]].to]){
                lowcost[edges[G[mark][j]].to]=edges[G[mark][j]].dist;
                adjvex[edges[G[mark][j]].to]=mark;
            }
        }
    }
}

int main()
{
    //freopen("read.in","r",stdin);
    scanf("%d %d",&n,&m);
    int tu,tv,tw;
    for(int i=0;i<m;++i){
        scanf("%d %d %d",&tu,&tv,&tw);
        addEdge(tu,tv,tw);
    }
    ans=-inf;
    Prim();
    printf("%d\n",ans);
    return 0;
}
```





- **vector中直接存每个边的信息（常用pair）first表示所连点的编号 second表示距离**
- pair有默认的重载小于号，即先first，后second

```c++
//求minispan_tree中longest edge
//O(N^2)
//vector中直接存每个边的信息（常用pair）first表示所连点的编号 second表示距离
#include<iostream>
#include<vector>
#include<cstdio>
using namespace std;
#define mp make_pair

const int maxn=10010;
const int inf=0x3f3f3f3f;
int ans;

vector<pair<int,int> > G[maxn];

int n,m;
void addEdge(int u,int v,int w){
    G[u].push_back(mp(v,w));
    G[v].push_back(mp(u,w));
}

int adjvex[maxn];//最小生成树上每个点的前驱
int lowcost[maxn];//前驱到这个点的距离
int v[maxn];//是否在生成树中的标记

void Prim()
{
    for(int i=1;i<=n;++i) lowcost[i]=inf;
    lowcost[1]=0;
    for(int i=1;i<=n;++i) adjvex[i]=0,v[i]=0;
    adjvex[1]=1;
    for(int i=1;i<=n;++i){
        //for(int i=1;i<=n;++i) cout<<lowcost[i]<<' ';
        int minn=inf,mark=-1;
        for(int j=1;j<=n;++j){
            if(!v[j]&&lowcost[j]<minn){
                minn=lowcost[j];
                mark=j;
            }
        }
        if(mark==-1) break;
        //printf("(i:%d,%d,%d)\n",i,adjvex[mark],mark);
        ans=max(minn,ans);
        v[mark]=1;
        for(int j=0;j<G[mark].size();++j){
            if(lowcost[G[mark][j].first]!=0&&G[mark][j].second<lowcost[G[mark][j].first]){
                lowcost[G[mark][j].first]=G[mark][j].second;
                adjvex[G[mark][j].first]=mark;
            }
        }
    }
}

int main()
{
    //freopen("read.in","r",stdin);
    scanf("%d %d",&n,&m);
    int tu,tv,tw;
    for(int i=0;i<m;++i){
        scanf("%d %d %d",&tu,&tv,&tw);
        addEdge(tu,tv,tw);
    }
    ans=-inf;
    Prim();
    printf("%d\n",ans);
    return 0;
}
```

## 前向星

关键：head      next   to

以**树形DP**为例 

**给定一棵n个点的树，问其中有多少条长度为偶数的路径**。路径的长度为经过的边的条数。x到y与y到x被视为同一条路径。路径的起点与终点不能相同。



```c++
#include<iostream>
#include<string.h>
#include<cstdio>
using namespace std;

const int maxn=100010;
int head[maxn<<1];
int tot,n;//tot辅助存边,n为结点数

struct Edge{
    int next,to;
}edges[maxn<<1];//注意，无向图开辟空间应为边数*2

void init()
{
    memset(head,-1,sizeof(head));
    tot=0;
}

void add_edge(int u,int v){
    edges[tot].to=v;
    edges[tot].next=head[u];
    head[u]=tot++;
}

int dp1[maxn],dp2[maxn];
//1为奇路径方案数，2为偶路径方案数

long long ans=0;//答案

void dfs(int rt,int f)
{
    dp1[rt]=0;
    dp2[rt]=1;
    for(int i=head[rt];i!=-1;i=edges[i].next){
        int tt=edges[i].to;
        if(tt==f) continue;
        dfs(tt,rt);
        ans+=dp1[rt]*dp2[tt]+dp2[rt]*dp1[tt];
        dp1[rt]+=dp2[tt];
        dp2[rt]+=dp1[tt];
    }
}

int main()
{
    //ios::sync_with_stdio(false);
        int u,v;
        scanf("%d",&n);
        init();
        for(int i=1;i<n;++i){
            scanf("%d %d",&u,&v);
            add_edge(u,v);
            add_edge(v,u);
        }
        dfs(1,-1);
        cout<<ans<<endl;
    return 0;
}
```





## DFS

时间复杂度	$O(m)$



**邻接表**

```c++
bool s[maxn]={0};

void DFS(int x)
{
    s[x]=true;
    cout<<x<<endl;
    for(int i=0;i<G[x].size();++i)
    {
        //对于每个未被访问的相邻结点，使用深度优先遍历。遍历返回后，尝试其他支路
        //用edges存储信息 G[x][i]为这条边在edges中的id
        //也可以使用pair 但信息量有限
        if(!s[edges[G[x][i]].to])
        {
            DFS(edges[G[x][i]].to);
        }
    }
}

```



**前向星**

```c++
bool s[maxn]={0};

void DFS(int x)
{
    s[x]=true;
    cout<<x<<endl;
    for(int i=head[x];i!=-1;i=edge[i].next)
    {
        //对于每个未被访问的相邻结点，使用深度优先遍历。遍历返回后，尝试其他支路
        if(!s[edge[i].to])
        {
            DFS(edge[i].to);
        }
    }
}

```

##  BFS

时间复杂度	$O(m)$



**邻接表**

```c++

bool s[maxn]={0};

void BFS(int x)
{
    queue<int> Q;
    Q.push(x);
    s[x]=true;
    while(!Q.empty())
    {
        int now=Q.front();
        Q.pop();
        cout<<now<<endl;
        for(int i=0;i<G[now].size();++i)
        {
            if(!s[edges[G[now][i]].to])
            {
                Q.push(edges[G[now][i]].to);
                s[edges[G[now][i]].to]=true;
            }
        }

    }
}
```





**前向星**

```c++

bool s[maxn]={0};

void BFS(int x)
{
    queue<int> Q;
    Q.push(x);
    s[x]=true;
    while(!Q.empty())
    {
        int now=Q.front();
        Q.pop();
        cout<<now<<endl;
        for(int k=head[now];k!=-1;k=edge[k].next)
        {
            if(!s[edge[k].to])
            {
                Q.push(edge[k].to);
                s[edge[k].to]=true;
            }
        }

    }
}
```

## 图的非递归遍历



```c++
void dfs(int k,int last)
{
   /* L[k]=++t;
    deep[k]=deep[last]+1;
    fa[k][0]=last;
    for(int i=0;i<g[k].size();++i)
        if(g[k][i]!=last) dfs(g[k][i],k);
    R[k]=t;*/
    while(!s.empty()) s.pop();
    memset(head,0,sizeof(head));//head[i]表示第i个点当前遍历到了第几个相邻点
    s.push(0);
    s.push(1);
    while(s.size()>1)
    {
        int k=s.top();
        s.pop();
        int last=s.top();
        s.push(k);
        if(!head[k])
        {
            L[k]=++t;
            deep[k]=deep[last]+1;
            fa[k][0]=last;
        }
        if(head[k]<g[k].size())
            if(g[k][head[k]]==last) ++head[k];
        if(head[k]==g[k].size())
        {
            R[k]=t;
            s.pop();
        }
        else
            s.push(g[k][head[k]++]);
    }
}
```

