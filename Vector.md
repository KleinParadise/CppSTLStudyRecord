# vector定义与常用函数实现
```cpp
template <class T,class Alloc = alloc>
class TestVector {
    // 嵌套类型定义
    typedef T value_type;
    typedef value_type*  Iterator;//迭代器的类型
    typedef value_type*  Point; //指针的类型
    typedef value_type&  reference; //引用的类型
    
    // 成员变量
    Iterator start;//开始位置
    Iterator finsh;//结束位置
    Iterator end_of_storage;//vector容器空间结束的位置
    
    //空间配置器
    typedef simple_alloc<value_type,Alloc> data_allocator;
    
public:
    // 函数
    
    //第一个迭代器位置
    Iterator begin(){
        return start;
    };
    //最后一个迭代器位置
    Iterator end(){
        return finsh;
    };
    //获取容器中数据的大小
    size_t size(){
        return size_t(finsh - start);
    };
    //获取容器的大小
    size_t capacity(){
        return size_t(end_of_storage - start);
    };
    //判断容器是否为空
    bool empty(){
        return start == finsh;
    };
    //获取容器中某一个位置的值
    reference operator[](size_t n){
        return *(begin() + n);
    };
    
    //构造函数
    TestVector():start(0),finsh(0),end_of_storage(0){};
    TestVector(size_t n, const T& value){
        fill_initialize( n, value);
    };
    TestVector(int n,const T& value){
        fill_initialize( n, value);
    };
    
    //析构函数
    ~TestVector(){
        //destory(start,finsh);
    };
    //第一个元素
    reference front(){
        return *begin();
    };
    //最后一个元素
    reference back(){
        return *finsh();
    };
    //将元素插入末端
    void push_back(const T& value){
        if(begin()!= end_of_storage){
            construct(finsh,value);
            ++finsh;
        }else{
            insert_aux(finsh, value);
        }
    };
    //将最后的元素取出
    void pop_bakc(){
        --finsh;
        destory(finsh);
    };
    //清除某个位置的元素
    Iterator ersae(Iterator position){
        if(position + 1 != end()){
            copy(position + 1, finsh, position);//position后面的数据从position位置处开始拷贝
            //删除最后一个元素
            --finsh;
            destory(finsh);
            return position;
        }
    };
    //在某个位置插入元素
    void insert_aux(Iterator position,const T& value){
        //如果容器的够插入数据 直接插入  如果不够申请一块更大的容器将容器数据迁移过去
        if(finsh != end_of_storage){
            construct(finsh,*(finsh -1));
            ++finsh;
            copy_backward(position, finsh-2, finsh-1);
            T copy_value = value;
            *position = copy_value;
        }else{
            //申请更大的容器 将原来容器的数据迁移过来
            // 每次重新配置 都会申请原容器两倍大小的内存
            //!!由于容器空间重新配置，容器地址会发生变化,指向原容器的迭代器会全部失效
        }
    };
    
    void fill_initialize(size_t n,const T& value){
    //stl 空间配置
    };
    
};
```


# vector 迭代器失效
*对Vector进行inster或者ersae操作时，如果vector空间不足便会引起vector空间的重新配置。即析构原vector容器，重新分配一个新vector容器，然后将原容器的数据
拷贝到新容器。这时候容器地址发生了变化，指向原容器的迭代器会全部失效*

