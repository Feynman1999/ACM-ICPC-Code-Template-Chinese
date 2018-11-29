## 最小费用最大流(Dijkstra实现)



```c++
#include<bits/stdc++.h>
using namespace std;
#define mp make_pair
const int maxn=2000+5,inf=1e9;
pair<int,int> a[maxn+5];
struct edge
{
    int to,cap,flow,cost,rev;
};
vector<edge> g[maxn+5];
struct heapnode
{
    int id,v;
    bool operator < (const heapnode &t) const
    {
        return v>t.v;
    }
};
priority_queue<heapnode> q;
int n,m,S,T,flow,cost;
int h[maxn+5],dis[maxn+5],cur[maxn+5];
bool ins[maxn+5];
void addedge(int from,int to,int cap,int cost)
{
    g[from].push_back({to,cap,0,cost,(int)g[to].size()});
    g[to].push_back({from,0,0,-cost,(int)g[from].size()-1});
}
bool Dijkstra()
{
    for (int i=0;i<=T;++i)
    {
        h[i]=min(h[i]+dis[i],inf);
        dis[i]=i==S?0:inf;
    }
    q.push(heapnode{S,0});
    while (!q.empty())
    {
        heapnode x=q.top();
        q.pop();
        if(x.v>dis[x.id]) continue;
        for(int i=0;i<g[x.id].size();++i)
        {
            edge e=g[x.id][i];
            if(e.cap>e.flow&&x.v+h[x.id]+e.cost-h[e.to]<dis[e.to])
            {
                dis[e.to]=x.v+h[x.id]+e.cost-h[e.to];
                q.push(heapnode{e.to, dis[e.to]});
            }
        }
    }
    return dis[T]<inf;
}
int dfs(int x,int a)
{
    if(x == T) return a;
    int ans = 0;
    ins[x] = 1;
    for(int &i=cur[x];i<g[x].size();++i)
    {
        edge &e=g[x][i];
        if(!ins[e.to]&&dis[x]+h[x]+e.cost-h[e.to]==dis[e.to]&&e.cap>e.flow)
        {
            int now=dfs(e.to,min(a,e.cap-e.flow));
            e.flow += now;
            g[e.to][e.rev].flow-=now;
            ans+=now;
            a-=now;
            if (!a) break;
        }
    }
    ins[x] = 0;
    return ans;
}
int main()
{
    freopen("ce.in","r",stdin);
    scanf("%d%d",&n,&m);
    for(int i=1;i<=n;++i) scanf("%d%d",&a[i].first,&a[i].second); // bottle
    for(int i=n+1;i<=n+m;++i) scanf("%d%d",&a[i].first,&a[i].second); // people
    scanf("%d%d",&a[n+m+1].first,&a[n+m+1].second);
    S=0,T=n+m+2;
    for(int i=n+1;i<=n+m;++i) addedge(S,i,1,0);
    addedge(S,n+m+1,n-1,0);
    for(int i=1;i<=n;++i) addedge(i,T,1,0);
    for(int i=n+1;i<=n+m+1;++i)
        for(int j=1;j<=n;++j)
            addedge(i,j,1,abs(a[i].first-a[j].first)+abs(a[i].second-a[j].second));
    while(Dijkstra())
    {
        memset(cur,0,sizeof(cur));
        int now=dfs(S,inf);
        flow+=now;
        cost+=now*(dis[T]+h[T]-h[S]);
    }
    for(int i=1;i<=n;++i) cost+=abs(a[i].first-a[n+m+1].first)+abs(a[i].second-a[n+m+1].second);
    printf("%d",cost);
    return 0;
}
```

