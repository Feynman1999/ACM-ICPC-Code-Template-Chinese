## LOG2 (ASM优化)

处理器能够在一个时钟周期内完成 `add`， `mov` 等指令，这些指令都需要访问全部的 32 个 bit，按理来说，处理器要想快速得到一个 1 的位置，是非常容易的。

IA-32 处理器确实也提供了这样的功能，被称为**位扫描（bit scan）**，位扫描指令分为 **`bsf`（bit scan forward）**和 **`bsr`（bit scan reversed）**，前者从最低位向最高位进行扫描直到找到第一个 1 为止，而后者相反。

有了 `bsr` 指令，我们就可以轻松快速地完成找到二进制位最高位的任务。



由于使用了` bsr` 指令，它在找到**最高位的 1** 时就会停止，所以其返回值，确实就是 `log2(x)`**取整后的值**。这个函数很有用，求 **RMQ 所用的 ST 表**、求 **LCA 所用的 log 在线算法**、以及许多**按位二分**的算法，都需要使用这样一个功能，而下面实现的 LOG2 函数，速度达到极致，又不需要多余的空间占用，堪称完美。   



```c++
#include<bits/stdc++.h>
using namespace std;

inline unsigned int LOG2(unsigned int x){
    unsigned int ret;
    __asm__ __volatile__ ("bsrl %1, %%eax":"=a"(ret):"m"(x));
    return ret;
}

int main()
{
    for(int i=0;i<32;++i){
        unsigned int tem=1;
        tem=(tem<<i);
        cout<<tem<<" :"<<LOG2(tem)<<endl;
    }
    cout<<LOG2(9)<<endl;
    return 0;
}

```





这里多提一句 `bsf` 指令，这条指令和 `bsr` 相反，是从**低位往高位**扫描找第一个 1，这首先让人联想到树状数组中使用的 `LOWBIT` 函数，不过使用 bsf 和 shl 实现 LOWBIT，并不见得会比 `x&-x` 来得快。我认为 bsf 指令最大的用处在于，可以用来**得到一个二进制数末尾有多少个 0**，并且一次性除去这些 0，在 `Miller Rubbin`、二进制 `gcd` 等算法中，都能发挥极大的作用。    