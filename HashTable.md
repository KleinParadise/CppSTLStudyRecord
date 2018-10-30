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

# hashtable数据结构及部分实现
```cpp
//hashtable
//Value 节点的实值类型
//Key 节点的键值类型
//HashFcn hashfunction的函数型别
//ExtractKey 从节点中取出键值得方法
//EqualKey 判断键值相同与否的方法
//Alloc 空间配置器
template <class Value,class Key,class HashFcn,class ExtractKey,class EqualKey,class Alloc>
class hashtable{
public:
    typedef HashFcn hasher;
    typedef EqualKey key_equal;
    typedef Key key_type;
    typedef Value value_type;
    typedef size_t size_type;
private:
    hasher hash;
    key_equal equals;
    ExtractKey get_key;
    typedef _hashtable_node<Value> node;
    
    std::vector<node*,Alloc> buckets;//桶已vector完成
    size_type num_elements;
    
public:
    size_type bucket_count() const{
        return buckets.size();
    }
    
// 构造函数
    hashtable(size_type n，const HashFcn& hf,const EqualKey& eql):
    hash(hf),equals(eql),get_key(ExtractKey()),num_elements(0){
        const size_type n_buckets = next_size(n);
        buckets.reserve(n_buckets);
        buckets.insert(buckets.begin(), n_buckets,(node*)0);
        num_elements = 0;
    }
    
//判断函数的落脚处bkt_num
    //版本1
    size_type bkt_num(const value_type& value,size_t n){
        return bkt_num_key(get_key(value), n);
    }
    
    //版本2
    size_type bkt_num(const value_type& value){
        return bkt_num_key(get_key(value));
    }
    
    //版本3
    size_type bkt_num_key(const key_type& key){
        return bkt_num_key(key,buckets.size())
    }
    
    //版本4
    size_type bkt_num_key(const key_type& key,size_t n){
        return hash(key) % n;
    }
    
//hashfunction 实际计算元素位置的函数。针对char，int，long等整数型别，忠实的返回原值。对字符串const char* 就设计了一个转换函数。string double float stl没有对应处理的hash函数,需要用户自定义
    
    
    
};

//以下函数判断hashtable需不需要重建vector桶表格 如果需要就新建一个更大的桶 将数据复制过去
template <class V,class K,class HF,class EX,class Eq,class A>
void hashtable<V, K, HF, EX, Eq, A>::resize(size_t num_elements_hint){
    const size_type old_n = buckets.size();
    if(num_elements_hint > old_n){
        const size_type n  = next_size(num_elements_hint);
        if (n > old_n){
            std::vector<node* ,A> tmp(n,(node*)0);
            for (size_type bucket = 0; bucket < buckets.size(); bucket++) {
                node* first = buckets[bucket];
                while (first) {
                    //找出该节点放在新桶那个位置
                    size_type new_bucket = bkt_num(first->val,n);
                    buckets[bucket] = first->next;
                    first->next = tmp[new_bucket];
                    tmp[new_bucket] = first;
                    first =buckets[bucket];
                }
            }
        }
    }
}
```
