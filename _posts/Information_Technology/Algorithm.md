# 排序

## 冒泡排序

冒泡排序（Bubble Sort）指每一轮排序都是最大的那个数冒到结尾。



### 定义

① **`从头开始index=1 比较`**每一对**`相邻`元素**，如果第1个比第2个大，就交换它们的位置；执行完一轮后，最末尾那个元素就是最大的元素。

② 忽略 ① 中曾经找到的最大元素，重复执行步骤 ①，直到全部元素有序。



```java
public void bubbleSort(){
    int[] array = {1, 3, 5, -1, -8};
    //比较 end 的结束范围，从最后一个元素-第二个元素
    for(int end = array.length - 1; end > 0; end--) {
        //冒泡排序-一轮的交换
        for(int begin = 1; begin <= end; begin++) {
            //当前元素的左边元素比较大时要交换
            if(array[begin] < array[begin-1]) {
                int temp = array[begin];
                array[begin] = array[begin-1];
                array[begin-1]=temp;
            }
        }
    }
}
```



### 优化

#### 提前有序，终止比较

```java
public void fastEndBubbleSort(){
    int[] array = {1, 3, 5, -1, -8};
    for(int end = array.length - 1; end > 0; end--) {
        //假设提前有序
        boolean sorted = true;
        for(int begin = 1; begin <= end; begin++) {
            if(array[begin] < array[begin-1]) {
                int temp = array[begin];
                array[begin] = array[begin-1];
                array[begin-1]=temp;
                //本轮有交换，证明假设提前有序不成立
                sorted = false;
            }
        }
        //果然提前有序，不用再比较了
        if(sorted) {
            break;
        }
    }
}
```



#### 尾部有序

```java
public void tailOrderedBubbleSort(){
    int[]  array = {1, 3, 5, -1, -8};
    for(int end = array.length - 1; end > 0; end--) {
        //记录最后一次交换的位置，初始值为1，是为了考虑一开始数组就是完全有序的
        int sortedIndex = 1;
        for(int begin = 1; begin <= end; begin++) {
            if(array[begin] < array[begin-1]) {
                int temp = array[begin];
                array[begin] = array[begin-1];
                array[begin-1]=temp;
                //记录最后一次交换的位置
                sortedIndex = begin;
            }
        }
        //更新比较范围到最后一次交换的位置
        end = sortedIndex;
    }
}
```



## 选择排序

选择排序（Selection sort）指每一轮都是把最大的那个数的位置给选择出来。



### 定义

① **从序列中找出最大的那个元素，然后与最末尾的元素交换位置**；执行完一轮后，最末尾的那个元素就是最大的元素。

② 忽略 ① 中曾经找到的最大元素，重复执行步骤 ①



选择排序本质上看是冒泡的优化：因为冒泡是从头到尾：相邻两个元素比较完就交换。选择排序是从头到尾：先找到最大元素位置，然后记录位置，最后才交换。



```java
public void selectionSort(){
    int[] array = {1, 3, 5, -1, -8};
    for(int end = array.length - 1; end > 0; end--) {
        int maxIndex = 0;
        //选择排序-找出本轮的最大值
        for(int begin = 1; begin <= end; begin++) {
            //取等号是为了变成稳定的排序算法
            // 10  10  8 --一轮比较--> 10   8   10
            if(array[maxIndex] <= array[begin]) {
                maxIndex = begin;
            }
        }
        //交换，把本轮找到的最大一个数放到结尾
        int temp = array[maxIndex];
        array[maxIndex] = array[end];
        array[end] = temp;
    }
}
```



## 堆排序

堆排序本质上看是冒泡的优化：选择排序是从头到尾：每一轮都在 从头到尾找到最大元素位置（内循环在找最大值）找最值，可以交给堆负责（优化）。



## 插入排序

## 归并排序

## 快速排序

## 希尔排序

# 参考资料

- []()
- []()
- []()
- []()
- []()
- []()
- []()
- []()
- []()
- []()
- []()
- [分布式一致性算法](https://www.cnblogs.com/aspirant/archive/2020/07.html)