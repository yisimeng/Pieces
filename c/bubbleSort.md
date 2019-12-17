# 冒泡排序

是一种交换排序方法，相邻的两个进行比较交换，每次会得出一个极值，时间复杂度是:O(n^2)，空间复杂度是:O(1)。

#### 测试用例

```
int arr[8] = {5,8,6,3,9,2,1,7};
```

### 基础写法

```
void bubbleSort(int arr[], int len) {
    int count = 0; // 总循环趟数
    for (int i = 0; i <= len-1 ; i++) {
        count++;
        for (int j = 0; j < len-1-i; j++) {
            if (arr[j] > arr[j+1]) {
                int temp = arr[j];
                arr[j] = arr[j+1];
                arr[j+1] = temp;
            }
        }
        // 每趟交换完成之后输出
        printf("%d time : ", count);
        arrayLog(arr, len);
    }
    // 输出总趟数
    printf("count:%d\n",count);
}
```
输出为：

```
1 time : 5,6,3,8,2,1,7,9,
2 time : 5,3,6,2,1,7,8,9,
3 time : 3,5,2,1,6,7,8,9,
4 time : 3,2,1,5,6,7,8,9,
5 time : 2,1,3,5,6,7,8,9,
6 time : 1,2,3,5,6,7,8,9,
7 time : 1,2,3,5,6,7,8,9,
8 time : 1,2,3,5,6,7,8,9,
count:8
```

> 问题：第6次之后就已经是有序的数组了，第7次已经没有交换了，所以第8趟是没有必要的，可以进行优化的。

### 标记有序，提前结束排序

设置一个标记，如果一趟中没有进行过交换，那么就已经是有序数列。

```
void bubbleSort(int arr[], int len) {
    int count = 0;
    for (int i = 0; i <= len-1 ; i++) {
        // 初始没有进行过交换
        bool hasExchanged = false;
        count++;
        for (int j = 0; j < len-1-i; j++) {
            if (arr[j] > arr[j+1]) {
                int temp = arr[j];
                arr[j] = arr[j+1];
                arr[j+1] = temp;
                // 进行了交换，修改标记
                hasExchanged = true;
            }
        }
        printf("%d time : ", count);
        arrayLog(arr, len);
        //判断是否进行过交换，如果没有进行交换，排序完成
        if (hasExchanged==false) {
            printf("count:%d\n",count);
            return;
        }
    }
    printf("count:%d\n",count);
}
```
输出为：

```
1 time : 5,6,3,8,2,1,7,9,
2 time : 5,3,6,2,1,7,8,9,
3 time : 3,5,2,1,6,7,8,9,
4 time : 3,2,1,5,6,7,8,9,
5 time : 2,1,3,5,6,7,8,9,
6 time : 1,2,3,5,6,7,8,9,
7 time : 1,2,3,5,6,7,8,9,
count:7
```

总趟数节省了一次。

> 问题：冒泡排序是从头至尾两两进行比较，每次将无序数组中的最值放到有序数组中，所以每次在右侧的有序数组中是不需要进行比较的。

### 设置无序数组边界

设置一个有序和无序数组的边界，只在无序数组中进行比较。

```
void bubbleSort(int arr[], int len) {
    int count = 0;
    int lastExIndex = 0;
    int sortBorderIndex = len-1;
    for (int i = 0; i <= len-1 ; i++) {
        bool isSorted = true;
        count++;
        for (int j = 0; j < len-1-i; j++) {
            if (arr[j] > arr[j+1]) {
                int temp = arr[j];
                arr[j] = arr[j+1];
                arr[j+1] = temp;
                
                isSorted = false;
                
                lastExIndex = j;
            }
        }
        sortBorderIndex = lastExIndex;
        printf("sortBorder : %d\n", sortBorderIndex);
        
        printf("%d time : ", count);
        arrayLog(arr, len);
        if (isSorted) {
            printf("count:%d\n",count);
            return;
        }
    }
    printf("count:%d\n",count);
}
```
输出为：
```
sortBorder : 6
1 time : 5,6,3,8,2,1,7,9,
sortBorder : 5
2 time : 5,3,6,2,1,7,8,9,
sortBorder : 3
3 time : 3,5,2,1,6,7,8,9,
sortBorder : 2
4 time : 3,2,1,5,6,7,8,9,
sortBorder : 1
5 time : 2,1,3,5,6,7,8,9,
sortBorder : 0
6 time : 1,2,3,5,6,7,8,9,
sortBorder : 0
7 time : 1,2,3,5,6,7,8,9,
count:7
```












