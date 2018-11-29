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

