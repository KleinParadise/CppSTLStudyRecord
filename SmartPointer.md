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
