# 原理
  * 用数组构成许多桶,数组每一个节点代表一个桶,单个桶维护一个多节点的链表。利用hash函数，对key进行映射到不同的桶进行保存。
  
# 链表中单个节点定义
 ```cpp
//hashtable桶(bucket)中节点的定义
template <class Value>
struct _hashtable_node {
    _hashtable_node* next;//指向下一个节点
    Value val;//当前节点的值
};
 ```
 
 # hashtable迭代器实现
 ```cpp
//hashtable的迭代器
template <class Value,class Key,class HashFcn,class ExtractKey,class EqualKey,class Alloc>
struct _hashtable_iterator {
    typedef hashtable<Value,Key,HashFcn,ExtractKey,EqualKey,Alloc> hashtable;//hashtable类型
    typedef _hashtable_iterator<Value,Key,HashFcn,ExtractKey,EqualKey,Alloc> iterator;//迭代器类型
    typedef _hashtable_node<Value> node;//节点类型
    
    typedef Value value_type;
    typedef Value* pointer;//迭代器指针类型
    typedef Value& reference;//迭代器引用类型
    typedef sig_t size_type;
    
    node* cur;//迭代器所指节点
    hashtable* ht;//迭代器与容器的联系。(跳转到下一个节点会用到)
    
    reference operator*() const{
        return cur->val;
    }
    
    pointer operator->() const{
        return &(operator*());
    }
    
    bool operator==(const iterator& it){
        return cur == it.cur;
    }
    
    bool operator!=(const iterator& it){
        return cur != it.cur;
    }
    
    iterator& operator++(){
        const node* old = cur;
        cur = cur->next;//如果cur->next有值，就取当前值 如果无值就要跳转到下一个桶取值。
        if(!cur){
            size_type bucket = ht->bkt_num(old->val);
            while(!cur && ++bucket < ht->buckets.size()){
                cur = ht->buckets[bucket];
            }
        }
        return *this;
    };
    
    iterator operator++(int){
        iterator temp = *this;
        ++*this;
        return temp;
    };
 ```
