## IO

### Fast_In

```
int read()
{
    char ch='*';
    while(!isdigit(ch=getchar()));//不是数字读掉
    int num=ch-'0';
    while(isdigit(ch=getchar())) num=num*10+ch-'0';
    return num;
}
```

```c++
//从文件读
//注意和ios::sync_with_stdio(false);冲突
typedef long long LL;
namespace fastIO {
    #define BUF_SIZE 100000
    //fread -> read
    bool IOerror = 0;
    inline char nc() {
        static char buf[BUF_SIZE], *p1 = buf + BUF_SIZE, *pend = buf + BUF_SIZE;
        if(p1 == pend) {
            p1 = buf;
            pend = buf + fread(buf, 1, BUF_SIZE, stdin);
            if(pend == p1) {
                IOerror = 1;
                return -1;
            }
        }
        return *p1++;
    }
    inline bool blank(char ch) {
        return ch == ' ' || ch == '\n' || ch == '\r' || ch == '\t';
    }
    inline void read(int &x) {
        char ch;
        while(blank(ch = nc()));
        if(IOerror)
            return;
        for(x = ch - '0'; (ch = nc()) >= '0' && ch <= '9'; x = x * 10 + ch - '0');
    }
    #undef BUF_SIZE
};
using namespace fastIO;
```





### 整行字符串的读入

**读char[]**  使用`cin.getline(str,50)` 其中第一个参数为字符串的地址  第二个参数为长度（直接写maxn）

**读string类** 使用`getline(cin,s)`            

两者均支持读EOF   

代码如下

**读char** 

```c++
#include<bits/stdc++.h>
using namespace std;

char str[50];

int main()
{
    //freopen("in.txt","r",stdin);
    while(cin.getline(str,50)){//注意由于用流 当数据很多时较慢
        printf("%s\n",str);
    }
    return 0;
}
```



**读string**

```c++
#include<bits/stdc++.h>
using namespace std;

string s;

int main()
{
    //freopen("in.txt","r",stdin);
    while(getline(cin,s)){
        cout<<s<<endl;
    }
    return 0;
}

```



### char[]和string之间的转换

**string转char[]**

```c++
#include<bits/stdc++.h>
using namespace std;

char str[50];
string s;

int main()
{
    //string 转 char[]
    cin>>s;
    strcpy(str,s.c_str());//速度较快
    printf("%s\n",str);
    return 0;
}
```



**char[]转string**

```c++
#include<bits/stdc++.h>
using namespace std;

char str[50];
string s;

int main()
{
    //char[] 转 string
    scanf("%s",str);
    s=str;
    cout<<s<<endl;
    return 0;
}
```





### 统计每行文本单词总个数(流方法)

**char[]**

```c++
#include<bits/stdc++.h>
#include<sstream>
using namespace std;

char str[50];

int main()
{
    freopen("in.txt","r",stdin);
    int cnt=0;
    while(cin.getline(str,50)){
        stringstream ss(str);
        char tmp[50];
        while(ss>>tmp) cnt++;
    }
    cout<<cnt<<endl;
    return 0;
}

```



**string**

```c++
#include<bits/stdc++.h>
#include<sstream>
using namespace std;

string s;

int main()
{
    freopen("in.txt","r",stdin);
    int cnt=0;
    while(getline(cin,s)){
        stringstream ss(s);
        string tmp;
        while(ss>>tmp) cnt++;
    }
    cout<<cnt<<endl;
    return 0;
}

```



