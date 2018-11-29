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



