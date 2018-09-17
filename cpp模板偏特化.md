## 模板偏特化
**通过在迭代器内部声明内嵌类型来获得迭代器所指对象的类型。这种只能方式只针对自定义迭代器类适用。**
**原生指针也是一种迭代器,但原生指针并不是一种类类型，它是无法定义内嵌类型的。如果获取原生指针所指对象的类型,就需通过模板偏特化来实现。**
```cpp
#include <iostream>

template <class T>
class Iter {
public:
    typedef T value_type; //自定义内嵌返回类型
    T* m_ptr;
    Iter(T* p=0):m_ptr(p){};
    T& operator*(){
        return *m_ptr;
    };
};

//通过该iterator_traits模板函数能获取自定义迭代器所指对象类型
template <class I>
struct iterator_traits {
    typedef typename I::value_type value_type;
};

通过该iterator_traits模板函数能获取原生指针所指对象类型
template <class I>
struct iterator_traits<I*> {
    typedef I value_type;
};

通过该iterator_traits模板函数能获取const指针所指对象类型 
iterator_traits<const int*>::value_type  //获得的value_type是const int，并不是int
用iterator_traits萃取出value_type并声明一个临时变量时，却发现声明的变量是const类型，并不能进行赋值
因此要为这种情况设计一个const偏特化版本
template <class I>
struct iterator_traits<const I*> {
    typedef I value_type;
};

template <class I>
typename iterator_traits<I>::value_type func(I iter) {
    std::cout<<"func enter *iter="<<*iter<<std::endl;
    return *iter;
}

int main(int argc, const char * argv[]) {
    // insert code here...
    std::cout << "Hello, World!\n";
    Iter<int> myIter(new int(8));
    func(myIter);
    
    int* p = new int(9);
    func(p);
    
    const char* ptr = "kbc";
    func(ptr);

    
    return 0;
}
```
