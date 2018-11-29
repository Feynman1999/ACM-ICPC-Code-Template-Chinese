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





