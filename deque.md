# 1.deque与vector比较
  * deque为双端数组,vector是单端的。
  * deque允许于常数时间内对起头端进行元素的插入或移除操作,对vector头端插入或者移除会引起vector内存空间重新调整
  * deque是动态地以分段连续的空间组合而成，随时可以增加一段新的空间并连接起来，即没有容量的概念。vector依赖容量建立，一旦内存空间达到上限，便会执行另觅更大空间,将原数据复制过去,释放原空间三部曲
  * deque为了维护分段连续线性空间，迭代器实现很复杂,vector的迭代器依赖普通指针实现。
  
  *对deque进行排序，最高效率，可将deque先完整的复杂到一个vector，将该vector排序后，再复制回deque*

# 2.deque的中控器map
  * map是一小块连续空间，其中每个元素（此处称为一个节点，node）都是指针，指向另一段（较大的）连续线性空间，称为缓冲区。缓冲区才是deque的储存空间主体。SGI STL 允许我们指定缓冲区大小，默认值0表示将使用512 bytes 缓冲区.
  * map其实是一个T**,是一个指针，所指之物又是一个指针，指向类型为T的一块空间。
  
下图为deque的架构图
![Image of deque](https://github.com/KleinParadise/CppSTLStudyRecord/blob/master/deque.JPG)
  
# 3.deque迭代器
  * 源码实现
  ```cpp
  //如果n不等于0 传回n 表示buff_size为用户自定义
//如果n等于0 表示buff_size使用默认值
inline size_t _deque_buf_size(size_t n,size_t sz){
    return n != 0 ? n:(sz < 512 ? size_t(512/sz):size_t(1));
}

template <class T,size_t Buf_size>
struct _deque_iterator {
    typedef T value_type;
    typedef value_type* pointer;
    typedef value_type& reference;
    typedef T** map_pointer;
    typedef _deque_iterator self;
    typedef ptrdiff_t difference_type;//两个指针相减的结果的类型为ptrdiff_t
    
    pointer cur;//此迭代器所指缓冲区的current元素
    pointer first;//此迭代器所指缓冲区的first元素
    pointer last;//此迭代器所指缓冲区的last元素
    map_pointer node;//node指针指向中控区map的一个指针 这个指针又指向此迭代器所指缓冲区的第一个元素
    
    //此迭代器所指缓冲区的size
    static size_t buffer_size(){
        return _deque_buf_size(Buf_size, sizeof(T));
    }
    
    //迭代器行进到缓冲区的边缘时 需要切换到下一个缓冲区 会用到set_node这个关键函数
    void set_node(const map_pointer new_node){
        node = new_node;
        first = *new_node;
        last = first + difference_type(buffer_size());
    }
    
    //deque迭代器几个比较重要的重载函数
    reference operator*()const{
        return  *cur;
    }
    
    pointer operator->()const{
        return &(operator*());
    }
    
    //两个迭代器相减
    difference_type operator-(const self& x)const{
        return difference_type(buffer_size()) * (node - x.node -1) + (cur - first) + (x.last -x.cur);
    }
    
    self& operator++(){
        ++cur;
        if(cur == last){//如果已经到该缓冲区的最后一个节点 切换至下一个缓冲区
            set_node(node + 1);
            cur = first;
        }
        return *this;
    }
    
    self& operator--(){
        if(cur == last){//如果已经到该缓冲区的最后一个节点 切换至前一个缓冲区
            set_node(node - 1);
            cur = last;
        }
        --cur;
        return *this;
    }
};
  ```
  * 中控区map与迭代器以及缓冲区关系图
  ![Image of deque](https://github.com/KleinParadise/CppSTLStudyRecord/blob/master/deque_struct.png)
  

# 4.deque的数据结构
```cpp
template <class T,size_t Buf_size>
class deque {
public:
    typedef T value_type;
    typedef value_type* pointer;
    typedef size_t size_type;
    typedef _deque_iterator<T , Buf_size> iterator;
    typedef pointer* map_pointer;
protected:
    iterator start;//第一个节点
    iterator finsh;//最后一个节点
    map_pointer map;//中控区map map为一个连续的空间 里面每一个指针指向一个缓冲区
    size_type map_size;//map中指针的数量
};
```

# 5.通过对deque的使用来理解deque的构造与内存分配
 1.构造deque并添加三个元素
   ```cpp
   deque<int> ideq(20,9);//构造deque令其保留20个空间初始值为9
   //重新设置deque里面元素的值
   for (int i=0; i<ideq.size(); i++) {
       ideq[i] = i;
   }
   //在deque尾端加三个元素 由于此时最后一个缓冲区还有4个空间 所以该操作不会造成空间的重新配置 直接插入数据
   for (int i=0; i<3; i++) {
       ideq.push_back(i);
   }
   ```
  - 此时deque在内存的模型大概如下图（假如设置deque缓冲区的为8）

2.此时在deque尾端在新增一个元素
  ```cpp
  ideq.push_back(3);//由于最后一个缓冲区还剩1个空间 该插入操作会使deque配置一块新的缓冲区，然后设妥新值，最后改变迭代器finsh的状态
  ```
  - 执行ideq.push_back(3)后deque的内存模型

3.在deque前端插入一个新元素
  ```cpp
  //如果第一缓冲区有空间直接插入该新元素 如无无备用空间该操作会使deque配置一块新缓冲区，然后将指向新缓冲区的节点安置在map上,最后调整迭代器start的状态
  ideq.push_front(99);
  ```
  - 执行ideq.push_back(3)后deque的内存模型
  
4.在deque前端又插入两个新元素
  ```cpp
  ideq.push_front(98);
  ideq.push_front(97);
  ```
  - 执行后的内存模型如下图



