# 数据结构与算法

1. **快排**

   ```c++
   void qsort(vector<int> &array, int left, int right) {
       if (left >= right) return;
       int low = left, high = right, target = array[left];
       while (low < high) {
           while (low < high && array[high] >= target) high--;
           if (low < high) array[low++] = array[high];
           while (low < high && array[low] <= target) low++;
           if (low < high) array[high--] = array[low];
       }
       array[low] = target;
       qsort(array, left, low-1);
       qsort(array, low+1, right);
   }
   ```

2. **堆排**

   ```c++
   void adjust(vector<int> &arr, int len, int index) {
       int left = index * 2 + 1;
       int right = index * 2 + 2;
       int idx = index;
       if (left < len && arr[left] < arr[idx]) { idx = left; }
       if (right < len && arr[right] < arr[idx]) { idx = right; }
       if (idx == index) return;
       swap(arr[idx], arr[index]);
       adjust(arr, len, idx);
   }
   
   void heap_sort(vector<int> &arr, int size) {
       for (int i = size / 2 - 1; i >= 0; i--) { adjust(arr, size, i); }
       for (int i = size - 1; i >= 1; i--) {
           swap(arr[0], arr[i]);
           adjust(arr, i, 0);
       }
   }
   ```

3. **二叉树**

4. **B+树**

5. **红黑树**

6. **哈希表**

7. **堆**