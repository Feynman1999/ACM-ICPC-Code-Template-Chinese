## 拓扑排序

**对一个有向无环图拓扑排序。**

**主要思路：**用队列实现，先把入度为0的点放入队列。然后考虑不断在图中删除队列中的点，每次删除一个点会产生一些新的入度为0的点。把这些点插入队列。

* 时间复杂度 $O(n+m)$



**简单变形**

将$queue$改为**优先队列**，**其余不变**，则含义有所改变：

有N个比赛队，编号依次为1，2，3，... ，N进行比赛，比赛结束后，裁判委员会要将所有参赛队伍从前往后依次排名，但现在裁判委员会不能直接获得每个队的比赛成绩，只知道每场比赛的结果，即P1赢P2，用P1，P2表示，排名时P1在P2之前。现在请你编程序确定排名。 **没有胜负关系的即排名不确定，按照点的序号排序。**

```c++
#include<bits/stdc++.h>
using namespace std;

const int maxn=1e5+10;
vector<int> g[maxn];

int du[maxn];//每个点的入度
int n,m;
int L[maxn];

bool toposort()//返回是否可以拓扑排序
{
    memset(du,0,sizeof(du));
    for(int i=1;i<=n;++i){
        for(int j=0;j<g[i].size();++j){
            du[g[i][j]]++;//统计入度
        }
    }
    int tot=0;
    priority_queue<int ,vector<int>,greater<int> > Q;
    for(int i=1;i<=n;++i){
        if(!du[i]) Q.push(i);//从度为0的点开始脱
    }
    while(!Q.empty())
    {
        int x=Q.top();Q.pop();
        L[++tot]=x;
        for(int j=0;j<g[x].size();++j){
            int t=g[x][j];
            du[t]--;
            if(!du[t]) Q.push(t);
        }
    }
    if(tot==n) return true;
    else return false;
}

int main()
{
    //freopen("in.txt","r",stdin);
    ios::sync_with_stdio(false);
    while(cin>>n>>m)
    {
        memset(du,0,sizeof(du));
        for(int i=1;i<=n;++i) g[i].clear();
        for(int i=1;i<=m;++i){
            int t1,t2;
            cin>>t1>>t2;
            g[t1].push_back(t2);
        }
        if(toposort())
        {
            for(int i=1;i<n;++i) cout<<L[i]<<" ";
            cout<<L[n];
        }
        cout<<endl;
    }
    return 0;
}
```

