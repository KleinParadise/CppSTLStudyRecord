## 悬锤指针
```Cpp
Point *p = new Point();
Point *p1 = p;
Point *p2 = p;
delete p;//此时p1和p2就成了悬锤指针
```
*当有多个指针指向同一个基础对象时，如果某个指针delete了该基础对象，对这个指针来说它是明确了它所指的对象被释放掉了，所以它不会再对所指对象进行操作，但是对于剩下的其他指针来说呢？它们还傻傻地指向已经被删除的基础对象并随时准备对它进行操作。于是悬垂指针就形成了。*



## 友元类
```Cpp
//SmartPointer是ObjContainer的友元类,通过SmartPointer中的成员变量oc能访问到ObjContainer类中的objVector数据
class ObjContainer {
    vector<Obj*> objVector;
public:
    void add(Obj* o){
        objVector.push_back(o);
    }
    friend class SmartPointer;
};

class SmartPointer{
    ObjContainer oc;
    int index;
public:
    SmartPointer(ObjContainer container){
        oc = container;
        index = 0;
    };
    
    bool operator++(){
        if(index >= oc.objVector.size() - 1) return false;
        if(oc.objVector[++index] == 0){
            return false;
        };
        return true;
    };
    
    bool operator++(int){
        return operator++();
    };
    
    Obj* operator->(){
        if(!oc.objVector[index]){
            return (Obj*)0;
        };
        return oc.objVector[index];
    };
};
```
*友元提供了一种 普通函数或者类成员函数 访问另一个类中的私有或保护成员 的机制类A中的成员函数访问类B中的私有或保护成员。*


## 重载operator-> 来实现通过类封装普通指针，来达到管理普通指针的目的，避免悬锤指针 即智能指针。
```Cpp
#include <iostream>
#include <vector>
using namespace std;

class Obj {
    static int i,j;
public:
    void f() {
        cout << i++ << endl;
    };
    
    void g(){
        cout << j++ << endl;
    };
};

int Obj::i = 10;
int Obj::j = 12;

class ObjContainer {
    vector<Obj*> objVector;
public:
    void add(Obj* o){
        objVector.push_back(o);
    }
    friend class SmartPointer;
};

class SmartPointer{
    ObjContainer oc;
    int index;
public:
    SmartPointer(ObjContainer container){
        oc = container;
        index = 0;
    };
    
    bool operator++(){
        if(index >= oc.objVector.size() - 1) return false;
        if(oc.objVector[++index] == 0){
            return false;
        };
        return true;
    };
    
    bool operator++(int){
        return operator++();
    };
    
    Obj* operator->(){
        if(!oc.objVector[index]){
            return (Obj*)0;
        };
        return oc.objVector[index];
    };
};


int main(int argc, const char * argv[]) {
    const int size = 10;
    //初始化Obj数组 里面放了10obj
    Obj o[size];
    ObjContainer oc;
    for (int i = 0; i < 10; i++) {
    	//将这10obj的普通指针放到vector中进行管理
        oc.add(&o[i]);
    }
    //初始化智能指针类 将10个普通指针交给智能指针类 管理
    SmartPointer sp(oc);
    do{
    	//由于重载了智能指针类->操作符，通过sp->操作返回的一个普通的obj对象指针，通过这个普通的obj对象指针来达到访问obj对象的目的。
        sp->f();
        sp->g();
    }while (sp++);
    return 0;
}
```


## 带引用计数的智能指针
```Cpp
#include <iostream>

class Point{
public:
    Point(int xValue,int yValue):x(xValue),y(xValue){};
    int GetX(){return x;}
    int GetY(){return y;}
private:
    int x,y;
};

class U_Ptr{
public:
    friend class SmartPtr;
    U_Ptr(Point *ptr):p(ptr),count(1){};
    ~U_Ptr(){
        delete p;
    };
//private:
    Point *p;
    int count;
};

class SmartPtr{
public:
	//构造函数
    SmartPtr(Point *ptr) :sp(new U_Ptr(ptr)){};
    //复制构造函数
    SmartPtr(const SmartPtr &sPtr):sp(sPtr.sp){
        ++sp->count;
    }
    //赋值函数
    SmartPtr &operator=(const SmartPtr &rhs){
        ++rhs.sp->count;
        if(--sp->count == 0){
            delete sp;
        }
        sp = rhs.sp;
        return *this;
    }
    
    ~SmartPtr(){
        if(--sp->count == 0){
            delete sp;
        }else{
            std::cout<< "还有" << sp->count << "个指针指向基础对象" << std::endl;
        }
    }
    
    Point &operator*(){
        return *(sp->p);
    }
    
    Point *operator->(){
        return sp->p;
    }
//private:
    U_Ptr *sp;
};

int main(int argc, const char * argv[]) {
    // insert code here...
    std::cout << "Hello, World!\n";
    Point *p = new Point(10,20);
    {
        SmartPtr sPtr1(p);{
            std::cout<< "1current the sPtr1.sp->count ="<<sPtr1.sp->count << std::endl;
            std::cout<< "1current the sPtr1.sp->count ="<<sPtr1->GetX() << std::endl;
            SmartPtr sPtr2(sPtr1);{
                std::cout<< "2current the sPtr1.sp->count ="<<sPtr1.sp->count << std::endl;
                SmartPtr sPtr3 = sPtr2;
                std::cout<< "3current the sPtr1.sp->count ="<<sPtr1.sp->count << std::endl;
            }
            std::cout<< "4current the sPtr1.sp->count ="<<sPtr1.sp->count << std::endl;
        }
        std::cout<< "5current the sPtr1.sp->count ="<<sPtr1.sp->count << std::endl;
    }
    //std::cout<< "3current the sPtr1.sp->count ="<<sPtr1.sp->count << std::endl;
    std::cout<< p ->GetX() << std::endl;
    return 0;
}
```
*引用计数是实现智能指针的一种通用方法。智能指针将一个计数器与类指向的对象相关联，引用计数跟踪共有多少个类对象共享同一指针。它的具体做法如下：
当创建类的新对象时，初始化指针，并将引用计数设置为1
当对象作为另一个对象的副本时，复制构造函数复制副本指针，并增加与指针相应的引用计数（加1）
使用赋值操作符对一个对象进行赋值时，处理复杂一点：先使左操作数的指针的引用计数减1（为何减1：因为指针已经指向别的地方），如果减1后引用计数为0，则释放指针所指对象内存。然后增加右操作数所指对象的引用计数（为何增加：因为此时做操作数指向对象即右操作数指向对象）。
析构函数：调用析构函数时，析构函数先使引用计数减1，如果减至0则delete对象。*


## ++i 与 i++的区别
1. i++返回原来的值，++i返回的是加1的值
2. i++不能作为左值，++i可以

- 什么是左值？
    - 左值是对应内存中有确定存储地址的对象的表达式的值，而右值是所有不是左值的表达式的值。
一般来说，左值是可以放到赋值符号左边的变量。
但能否被赋值不是区分左值与右值的依据。比如，C++的const左值是不可赋值的；而作为临时对象的右值可能允许被赋值。
左值与右值的根本区别在于是否允许取地址&运算符获得对应的内存地址。

```Cpp
// 前缀形式：
int& int::operator++() //这里返回的是一个引用形式，就是说函数返回值也可以作为一个左值使用
{//函数本身无参，意味着是在自身空间内增加1的
  *this += 1;  // 增加
  return *this;  // 取回值
}

//后缀形式:
const int int::operator++(int) //函数返回值是一个非左值型的，与前缀形式的差别所在。
{//函数带参，说明有另外的空间开辟
  int oldValue = *this;  // 取回值
  ++(*this);  // 增加
  return oldValue;  // 返回被取回的值
}
```

