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

# 4.deque的构造与内存分配



