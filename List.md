```cpp
//list节点结构
template <class T>
struct _list_node {
    _list_node<T>* next;//指向下一个节点结构的指针
    _list_node<T>* prev;//指向上一个节点结构的指针
    T data;//list中单个数据
};

//list迭代器
template <class T>
struct _list_iterator {
    typedef _list_iterator<T> self;
    typedef T valuetype;
    typedef valuetype* pointer;
    typedef valuetype& reference;
    typedef _list_node<T>* linktype;
    //构造函数
    _list_iterator(linktype rhs):node(rhs){};
    _list_iterator(){};
    _list_iterator(const self& x):node(x.node){};
    //运算符重载
    //函数后const表示该函数为只读函数 不改变数据成员
    bool operator==(const self& x)const{
        return x.node == node;
    };
    bool operator!= (const self& x)const{
        return !(operator==(x));
    }
    reference operator*()const{
        return (*node).data;
    }
    pointer operator->(){
        return &(operator*());
    }
    
    self& operator++(){
        node = (linktype)((*node).next);
        return *this;
    };
    
    self operator++(int){
        self temp = *this;
        ++*this;
        return temp;
    }
    
    self& operator--(){
        node = (linktype)((*node).prev);
        return *this;
    };
    
    self operator--(int){
        self temp = *this;
        --*this;
        return temp;
    }
    linktype node;
};

template <class T>
class TestList {
public:
    typedef T valuetype;
    typedef valuetype& reference;
    typedef _list_node<T>* linktype;
    typedef _list_iterator<T> iterator;
    linktype node;//list为环装链表,node指向list尾端一个空白节点，符合开闭后开的原则
    //常用函数实现
    
    iterator begin(){
        return (linktype)((*node).next);//环装链表 最后一个空白节点的next指向第一个节点
    };
    
    iterator end(){
        return node;
    };
    
    bool empty()const{
        return node->next == node;//环装链表 最后一个空白节点的next指向第一个节点
    }
    
    size_t size(){
        size_t result = 0;
        std::distance(begin(), end(),result);//distance是计算两个iterator直接的距离
        return  result;
    }
    
    reference front(){
        return *(begin());//取list第一个值data,返回第一个迭代器,重载*运算符取值
    }
    
    reference back(){
        return *(--end());//end()为list尾端一个空白节点
    }
    //创建一个节点
    linktype create_node(const T& x){};
    //销毁一个节点
    void destory_node(linktype p){}
    
    //在指定位置新增一个位置
    iterator insert(iterator position,const T& x){
        linktype temp = create_node(x);//创建一个节点
        temp->next = position.node;//设置新创建节点的next节点为position
        temp->prev = position.node->prev;//设置新创建节点的prev节点为position的prev节点
        (linktype)(position.node->prev)->next = temp;//设置position的prev节点的next节点为新创建的temp节点
        position.node->prev = temp;//设置position节点的prev节点为新创建的temp节点
        return temp;
    }
    //插入一个节点 为头节点
    void push_front(const T& x){
        insert(begin(), x);//在第一个节点前面加入新节点
    }
    //插入一个节点为尾节点
    void push_back(const T& x){
        insert(end()(), x);//在末端空白节点加入新节点
    }
    
    //删除一个位置上的节点
    iterator erase(iterator position){
        iterator next_node = position.node->next;//获取需要删除节点的next节点
        iterator prev_node = position.node->prev;//获取需要删除节点的prev节点
        next_node->prev = prev_node;//将需要删除节点的next节点的prev节点设置为需要删除节点的prev节点
        prev_node->next = next_node;//将需要删除节点的prev节点的next节点设置为需要删除节点的next节点
        destory_node(position.node);//删除需要删掉的节点
        return next_node;
    }
    
    //移除头节点
    void pop_front(){
        erase(begin());
    }
    //移除尾节点
    void pop_back(){
        erase(--end());
    }
    
    //清除链表所有节点
    void clear(){
        linktype cur = (linktype)node->next; //获取第一个节点
        while (cur != node) {
            linktype temp = cur;
            cur = (linktype)cur->next;//获取下一个节点
            destory_node(temp);//销毁当前节点
        }
        //当所有节点删除完毕，将空白节点的next和prev节点设置为自己。相当于调用list()构造函数构造了一个新的链表
        node->next = node;
        node->prev = node;
    }
    
    //将数值为value的值都移除
    void remove(const T& value){
        iterator frist = begin();//获取第一个节点
        iterator last = end();//获取第二个节点
        while (frist != last) {//遍历所有节点
            iterator next = frist;
            ++next;//获取下一个节点
            if(*frist == value){//如果该节点value等于要移除的value 移除该节点
                erase(frist);
            }
            frist = next;
        }
    }
    
    //移除数字相同的连续元素 **只有连续相同元素 才会被移除
    void unique(){
        iterator frist = begin();//获取第一个节点
        iterator last = end();//获取第二个节点
        if(frist == last) return;
        iterator next = frist;
        while (++next != last) {
            if(*frist == *next){//如果有相同值就移除
                erase(frist);
            }else{
                frist = next;//设置从next指针进行遍历
                next = frist;//设置从next指针进行遍历，修正遍历范围
            }
        }
    }
    
    //将[fast,last)内所有的元素移动到position之前
    void transfer(iterator position,iterator first,iterator last){
        (*(linktype((*last.node).prev))).next = position.node;//设置last的上一个节点next节点为position节点
        (*(linktype((*first.node).prev))).next = last.node;//设置first的prev节点为last节点
        (*(linktype((*position.node).prev))).next = first.node;//设置position的
        linktype temp = (linktype)(*(*position.node).prev);
        (*position.node).prev = (*last.node).prev;
        (*last.node).prev = (*first.node).prev;
        (*first.node).prev = temp;
    }

};
```
