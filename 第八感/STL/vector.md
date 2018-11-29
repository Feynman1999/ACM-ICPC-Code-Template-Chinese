## vector



删除最后一个元素

```c++
v.erase(v.rbegin());//错误  
v.pop_back();//正确
v.erase(v.end()-1);//正确
```









