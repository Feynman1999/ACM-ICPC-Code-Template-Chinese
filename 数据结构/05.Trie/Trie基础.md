# Trie

## Tire

插入字符串   $O(Length)$

查询一个字符串是否存在 $O(Length)$

flag[i]代表结点i是否为一个字符串尾 或者经过该结点的单词数（根据需要）



**指针写法**

```c++
#include<bits/stdc++.h>
using namespace std;

const int charset=26;

struct Node
{
    Node *next[charset];
    int flag;
    Node(){
        for(int i=0;i<charset;++i){
            next[i]=NULL;
        }
        flag=0;
    }
};

Node *root;

void init()
{
    root= new Node();
}

void del(Node *p)
{
    for(int i=0;i<charset;++i){
        if(p->next[i]!=NULL) del(p->next[i]);
    }
    delete p;
}

char temp[20];

void ins(char *s)
{
    int len=strlen(s);
    Node *now=root;
    for(int i=0;i<len;++i){
        int to=s[i]-'a';
        if(now->next[to]==NULL) now->next[to]=new Node();
        now=now->next[to];
    }
    now->flag=1;
}

int fid(char *s)
{
    int len=strlen(s);
    Node *now=root;
    for(int i=len-1;i>=0;--i){
        int to=s[i]-'a';
        if(now->next[to]==NULL) return 0;
        now=now->next[to];
    }
    return now->flag;
}

int main()
{
    //ios::sync_with_stdio(false);
    //freopen("in.txt","r",stdin);
    init();
    int n;
    scanf("%d",&n);
    int ans=0;
    while(n--)
    {
        scanf("%s",temp);
        ins(temp);
        ans+=fid(temp);
    }
    printf("%d\n",ans);
    del(root);
    return 0;
}

```





**数组写法**

```c++
#include<bits/stdc++.h>
using namespace std;

const int charset=26;
const int maxn=1e6+10;

int Trie[maxn][charset];

bool flag[maxn];
int cnt;//关键
int root;

void init()
{
    root=0;
    cnt=0;
}

void del()
{
    memset(Trie,0,sizeof(Trie));
    memset(flag,0,sizeof(flag));
}

char temp[20];

void ins(char *s)
{
    int len=strlen(s);
    int now=root;
    for(int i=0;i<len;++i){
        int to=s[i]-'a';
        if(Trie[now][to]==0) Trie[now][to]=++cnt;
        now=Trie[now][to];
    }
    flag[now]=1;
}

int ans=0;

void fid(char *s)
{
    int len=strlen(s);
    int now=root;
    for(int i=len-1;i>=0;--i){
        int to=s[i]-'a';
        if(Trie[now][to]==0) return ;
        now=Trie[now][to];
    }
    ans+=flag[now];
}


int main()
{
    //ios::sync_with_stdio(false);
    //freopen("in.txt","r",stdin);
    init();
    int n;
    scanf("%d",&n);
    while(n--)
    {
        scanf("%s",temp);
        ins(temp);
        //reverse(temp,temp+strlen(temp));
        //cout<<temp<<endl;
        fid(temp);
    }
    printf("%d\n",ans);
    del();
    return 0;
}

```
