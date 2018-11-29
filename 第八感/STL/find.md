## find

寻找vector/array中的一个元素



待完善...

```c++
#include <bits/stdc++.h>
using namespace std;
#define all(x) (x).begin(),(x).end()
int a[10];


int main()
{
    for(int i=0;i<5;++i) a[i]=10+i;
    vector<int> v(a,a+5);
    cout<<v.size()<<endl;
    for(auto k:v) cout<<k<<endl;
    if(find(all(v),12)!=v.end()) cout<<find(v.begin(),v.end(),12)-v.begin()<<endl;
    else cout<<"g"<<endl;
    return 0;
}

```









