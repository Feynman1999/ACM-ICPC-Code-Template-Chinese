## 最小生成树Prim算法

- **邻接表，vector中存每个点所连边的序号 通过边集数组索引** 
- 时间复杂度 $O(N^2)$

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

void addEdge(int u,int v,int w){
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





- **邻接表，vector中直接存每个边的信息（常用pair）first表示所连点的编号 second表示距离** 
- 时间复杂度 $O(N^2)$

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



- **使用堆优化**
- O()

```c++

```







## 最小生成树Kruskal算法

* 边集数组  

* 贪心思想 每次取剩下的边权最小的边（检查环用**并查集**维护）
* 时间复杂度$O(MlogM+M)$  (排序+贪心)  

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
    Edge() {}
    Edge(int x,int y,int w):from(x),to(y),dist(w){}
    bool operator < (const Edge & e) const {//后面的const必须加上
        return (dist<e.dist);
    }
};

Edge edges[maxn];//这里有Edge型的变量edges  前面Edge结构体必须加上Edge() {}


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



