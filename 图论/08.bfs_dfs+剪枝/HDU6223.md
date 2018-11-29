## HDU 6223  (17沈阳)

告诉你每个点的权值，但从第一个点开始后，以后每一个点按照 $(i^2 + 1)\%N$ 去走，问一共走$N$个点的路径，字典序最大是多少？



## 思路

bfs+剪枝

开始时显然选择权值最大的那些点出发

于是多源bfs

两个剪枝：

* 对于每一层的点，我只取权值最大的那些点（可以用优先队列）
* 对于同一层的点，做标记，保证只拓展一次 （注意这里用map会超时 ，要用栈保存下在vis数组的下标，做到线性打标记/去标记）



## 代码

```c++
#include<bits/stdc++.h>
using namespace std;
const int maxn=200000;
int n;

short ans[maxn];
int D[maxn];//每个点的值
bool vis[maxn];
int sta[maxn];
int top=0;

struct node{
    node(){}
    node(int i_,int dep_){
        i=i_,dep=dep_;
    }
    int i,dep;
};

priority_queue <node> que;

bool operator < (node A,node B){
    if(A.dep == B.dep){
        return D[A.i]<D[B.i];
    }
    return A.dep>B.dep;
}

//有一个结论 由于题目i*i+1 %n 的限定 当前层不同的点数量会降下来 因此可以剪枝
//两个剪枝方法：1.同层的只取值最大的点  2.同一层走过的点不再走
void bfs(int maxx)
{
    int id=0,last=0;//last为0使得一开始就换层
    while(!que.empty())
    {
        node q=que.top();
        //cout<<q.i<<endl;
        que.pop();
        if(q.dep!=last){//换层
            assert(q.dep == last+1);
            ans[id++]=D[q.i];//换层的第一个点必为ans
            last=q.dep;//更新层
            maxx=D[q.i];//更新新一层的最大值
            while(top){vis[sta[top--]]=false;}
            sta[++top]=q.i;
            vis[q.i]=true;
            if(q.dep==n) continue;
            que.push(node((1LL*q.i*q.i+1)%n,q.dep+1));
        }
        else if(D[q.i]<maxx || vis[q.i] || q.dep==n) continue;//当前层变小了或者走过了就pass
        else{
            sta[++top]=q.i;
            vis[q.i]=true;
            que.push(node((1LL*q.i*q.i+1)%n,q.dep+1));
        }
    }
}

int main()
{
    //freopen("in.txt","r",stdin);
    ios::sync_with_stdio(false);
    cin.tie(0);
    cout.tie(0);
    int t;
    cin>>t;
    int T=0;
    while(t--)
    {
        T++;
        while(!que.empty()) que.pop();
        cin>>n;
        string s;
        cin>>s;
        int maxx=0;
        for(int i=0;i<s.size();++i){
            D[i]=s[i]-'0';
            maxx=max(maxx,D[i]);
        }
        for(int i=0;i<n;++i){
            if(D[i]==maxx) que.push(node(i,1));
        }
        bfs(maxx);
        cout<<"Case #"<<T<<": ";
        //cout<<n<<endl;
        for(int i=0;i<n;++i){
            cout<<ans[i];
        }
        cout<<endl;
    }
    return 0;
}
```

