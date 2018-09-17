## 模板参数自动推导
**原理：**
* 当模板的形参与模板函数的形参在位置上，具有一定的对应关系时，这时调用模板函数，C++编译器会自动根据模板函数的实参来推测模板的实参。

**使用模板参数自动推导需要满足3个条件：**

1. 模板的形参必须与模板函数的形参在位置上存在一一对应的关系

2. 与模板函数返回值相关的模板参数无法进行自动推导

3. 需要推导的模板参数必须是连续位于模板参数列表的尾部，中间不能有不可推导的模板参数。

```cpp
#include <iostream>
using namespace std;

template <typename T1,typename T2> //T1 T2 模板形参
int get_max_type(T1 t1,T2 t2) { // t1 t2 模板函数形参
    int size = 0;
    if(sizeof(t1) >= sizeof(t2)){
        size = sizeof(t1);
        cout<< "---- t1"<<endl;
    }else{
        size = sizeof(T2);
        cout<< "---- t2"<<endl;
    }
    
    T1 m = 10;
    T1 n = 5;
    
    cout<< "---- m * n ="<< m * n <<endl;
    
    
    return size;
}



int main(int argc, const char * argv[]) {
    // insert code here...
    std::cout << "Hello, World!\n";
    //int nMax = get_max_type('a', 10);
    int nMax = get_max_type<char,int>('a', 10);// char int 模板实参  "a",10 模板函数实参
    cout<<"max:"<<nMax<<endl;
    return 0;
}
```
