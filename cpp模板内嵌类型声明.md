## 模板内嵌类型声明
**如何以迭代器所指对象的类型声明返回类型？**
* 通过在迭代器内部声明内嵌类型来实现

```cpp
#include <iostream>
template <typename T>
class Iterator {
public:
    //在迭代器类中声明内嵌类型 typedef在这里的作用是定义T类型的别名 即 T = Iterator::value_type
    typedef T value_type;
    Iterator(T *p=0):m_ptr(p){};
    T& operator*(){
        return *m_ptr;
    };
private:
    T *m_ptr;
};
//func的返回类型必须加上typename，意为告诉编译器这是个类型
//typename Iterator::value_type 为func函数的返回类型
template <typename Iterator>
typename Iterator::value_type func(Iterator ite) {
    std::cout << "*ite="<<*ite<<std::endl;
    return *ite;
}

int main(int argc, const char * argv[]) {
    // insert code here...
    std::cout << "Hello, World!\n";
    
    Iterator<int> testIter(new int(8));
    func(testIter);
    
    return 0;
}
```
