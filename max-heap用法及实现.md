# STL中构造最大堆 以及对最大堆进行排序
```cpp
#include <iostream>
using namespace std;

int main(int argc, const char * argv[]) {
    // insert code here...
    int a[9] = {0,2,4,5,7,9,8,1,3};
   
    vector<int> ivec(a,a+9);
    make_heap(ivec.begin(), ivec.end());
    for (int i = 0; i < ivec.size(); i++) {
        cout<<"make_hep ="<<ivec[i]<<endl;
    }
    
    sort_heap(ivec.begin(), ivec.end());
    for (int i = 0; i < ivec.size(); i++) {
        cout<<"make_hep ="<<ivec[i]<<endl;
    }
    return 0;
}
```


# 最大堆cpp实现
```cpp
#include <iostream>
using namespace std;

template <class T>
class MaxHeap {
public:
    MaxHeap(int capacity);
    bool instert(T val);//heap中新增数据
    bool remove(T data);//heap删除数据
    T getTop();//获取最大的数据
    bool createMaxHeap(T a[],int size);//创建一个二叉最大堆
    void print();
    ~MaxHeap();
private:
    int capacity;//容量
    int size;//堆中有效数据大小
    T* heap;
    
    void filterUp(int index);//从index往根节点调整堆
    void filterDown(int begin,int end);//从begin所在节点开始，向end方向调整堆
};

#include "MaxHeap.hpp"
//构造函数
template <class T>
MaxHeap<T>::MaxHeap(int capacity): capacity(capacity),size(0),heap(nullptr){
    heap = new T[capacity];
}

//析构函数
template <class T>
MaxHeap<T>::~MaxHeap<T>(){
    delete []heap;
}

//获取头值
template <class T>
T MaxHeap<T>::getTop() {
    return heap[0];
}

//根据一个vector创建一个heap
template <class T>
bool MaxHeap<T>::createMaxHeap(T a[],int arrySize) {
    if( arrySize > capacity){
        return false;
    }
    for (int i = 0; i < arrySize; i++) {
        cout<<"a[i]"<<a[i]<<endl;
        instert(a[i]);
    }
    return true;
}

//新增一个元素
template <class T>
bool MaxHeap<T>::instert(T val) {
    if(size >= capacity){
        return false;
    }
    heap[size] = val;
    filterUp(size);
    size++;
    return true;
}

//往maxheap里面插入数据
template <class T>
void MaxHeap<T>::filterUp(int index) {
    T value = heap[index];
    //递归 向上
    while (index > 0) {
        cout<<"index ="<<index<<endl;
        int parentIndex = (index - 1) / 2;//取该节点的父节点
        //如果插入的值小于父节点 终止递归
        if(value < heap[parentIndex]){
            break;
        }else{
            //如果插入的值大于父节点
            heap[index] = heap[parentIndex];
            index = parentIndex;
        }
    }
    heap[index] = value;
}

//删除一个元素
template <class T>
bool MaxHeap<T>::remove(T data) {
    if(size == 0) return false;
    int index;
    for (int i = 0; i < size; i++) {
        if(heap[i] == data){
            index = i;
            break;
        };
    }
    if(size == capacity) return false;
    
    heap[index] = heap[size - 1]; //使用最后一个节点来代替当前结点，然后再向下调整当前结点。
    filterDown(index, size--);
    return true;
}

template <class T>
void MaxHeap<T>::filterDown(int current, int end) {
    //取current节点的左子节点
    int child = current * 2 + 1;//左节点
    T value = heap[current];
    while (child <= end) {
        //算出current节点的左右两个节点那个比较大 取最大节点；
        if(child < end && heap[child] < heap[child+1]){
            child++;
        }
        if(value > heap[child]){
            break;
        }else{
            heap[current] = heap[child];
            current = child;
            child = current * 2 + 1;
        }
    }
    heap[current] = value;
}

template <class T>
void MaxHeap<T>::print(){
    cout<<"size ="<<size<<endl;
    for (int i = 0; i < size; i++)
        cout << heap[i] << " ";
}



#include <iostream>
#include <vector>
#include "MaxHeap.cpp"
using namespace std;

int main(int argc, const char * argv[]) {
    // insert code here...
    int a[9] = {0,2,4,5,7,9,8,1,3};
    MaxHeap<int> TestHeap(9);
    TestHeap.createMaxHeap(a, 9);
    TestHeap.print();
    return 0;
}
```
