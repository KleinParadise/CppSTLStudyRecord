## 迭代器
* 迭代器是一种行为类似指针的对象。指针最常用的功能就是内容提领（dereference) 即（*指针运算符）和成员访问（member access）即（->成员运算符）

	- 迭代器最重要的工作就是对*指针运算符和->成员运算符进行重载。

	- 对*指针运算符进行重载 返回被迭代的对象

	- 对->成员运算符进行重载 访问被迭代对象的成员

	- 对operator++() 进行重载 返回被迭代的对象的下一个被迭代的对象

	- 对operator== != 进行  对被迭代的对象和同一类型的对象进行进行比较*
  
**指针运算符的重载*
```Cpp
//重载解引用
//&    指针运算符 & 返回变量的地址。例如 &a; 将给出变量的实际地址。
//*    指针运算符 * 指向一个变量。例如，*var; 将指向变量 var。
#include <iostream>
using namespace std;

template<typename T> class DataContainer
{
    T *p;
public:
    DataContainer(T* pp)
    {
        p=pp;
    }
    
    ~DataContainer()
    {
        delete p;
    }
    
    T operator*()
    {
        return *p;
    }
    
    //template<typename T> friend T operator*(const DataContainer<T>&);
};

template<typename T> T operator*(const DataContainer<T>& d)
{
    return *(d.p);
};

int main()
{
    DataContainer<int> intData(new int(5));
    DataContainer<double> doubleData(new double(7.8));
    cout<<*intData<<endl;
    cout<<*doubleData<<endl;
    return 0;
}
```

*链表迭代器的模拟实现*
```Cpp
#include <iostream>

class ListItem{
public:
    int GetValue() const{
        return currentValue;
    };
    
    ListItem* GetNextItem() const {
        return next;
    };
    
//private:
    int currentValue;
    ListItem* next;
};

class List{
    void Insert_front(ListItem* item){};
    void Insert_end(ListItem* item){};
private:
    ListItem* front;
    ListItem* end;
    long size;
};

class ListIterator{
    ListIterator(ListItem* listItem):item(listItem){};
    
    ListItem* operator*(){
        return item;
    };
    
    ListItem& operator->(){
        return *item;
    };
    
    ListIterator& operator++(){
        item = item->GetNextItem();
        return *this;
    };
    
    ListIterator operator++(int){
        ListIterator temp = *this;
        ++*this;
        return temp;
    };
    
    bool operator==(ListIterator& iter){
        return item == iter.item;
    };
    
    bool operator!=(ListIterator& iter){
        return item != iter.item;
    };
    
//private:
    ListItem* item;
};

int main(int argc, const char * argv[]) {
    // insert code here...
    std::cout << "Hello, World!\n";
    return 0;
}
```
