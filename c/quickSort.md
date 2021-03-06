# 快速排序

快排每次会选取一个基准值，每趟排序会确定基准值的位置。通过基准值将数组进行分割，一边都是比基准值大的，一边都是比基准值小的，分成两个数组，再分别递归两个数组进行排序。

```
void quickSort(int arr[], int low, int high) {
    int i = low, j = high;
    // 选取基准值
    int temp = arr[low];
    if (i < j) {
        // 判断i和j是否过界
        while (i < j) {
            // 先从最右至左依次判断，查找第一个小于基准值的下标，否则j--
            while (i < j && arr[j] > temp) {
                j--;
            }
            if (i < j) {
                // 查找到小于基准值的下标，将基准值的位置的值进行替换，查找到的下标位置当做被挖了个坑，待填充数据
                arr[i] = arr[j];
                // 由于i处的值已经与基准值做过比较，因此从i的下一个开始进行从左至右判断
                i++;
            }
            // 从左至右依次判断第一个大于基准值的i下标，否则i++
            while (i < j && arr[i] <= temp) {
                i++;
            }
            if (i < j) {
                // 查找到大于基准值的下标，将这里的值挖走，填到上面挖坑的地方，此处变成坑
                arr[j] = arr[i];
                // 因为j处的值已经比较过，肯定大于基准值，所以j--
                j--;
            }
        }
        // ij相遇，一遍循环完成，基准值的位置就是i，进行赋值，此时 i 左侧都比基准值小，i 右侧都比基准值大
        arr[i] = temp;
        // 将数组以 i 为界进行分割，从 low 到 i-1， 为比基准值小的数组，从 i+1 到 high 为都比基准值大的数组
        quickSort(arr, low, i-1);
        quickSort(arr, i+1, high);
        //递归完成两侧的调用
    }
}
```



























