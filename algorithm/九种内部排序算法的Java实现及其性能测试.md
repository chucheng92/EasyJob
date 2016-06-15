## 九种内部排序算法的Java实现及其性能测试

### 9种内部排序算法性能比较

第九种为java.util.Arrays.sort（改进的快速排序方法）

1. 100000的随机数据集
![](http://7xlkoc.com1.z0.glb.clouddn.com/sort1.jpg)
2. 200000的随机数据集
![](http://7xlkoc.com1.z0.glb.clouddn.com/sort2.jpg)
3. 500000的随机数据集
![](http://7xlkoc.com1.z0.glb.clouddn.com/sort3.jpg)

结论：归并排序和堆排序维持O(nlgn)的复杂度，速率差不多，表现优异。固定基准的快排表现很是优秀。而通过使用一个循环完成按增量分组后的直接插入的希尔排序，测试效果显著。
冒泡，选择，直接插入都很慢，而冒泡效率是最低。

### 1.插入排序[稳定]

适用于小数组,数组已排好序或接近于排好序速度将会非常快

复杂度：O(n^2) - O(n) - O(n^2) - O(1)[平均 - 最好 - 最坏 - 空间复杂度]

```java
public void insertionSort(int[] a) {
		if (null == a || a.length < 2) {
			return;
		}
		for (int i = 1; i < a.length; i++) {
			// 暂存当前值
			int temp = a[i];
			int j = i - 1;
			while (j >= 0 && temp < a[j]) {
				// 后移
				a[j + 1] = a[j];
				j--;
			}
			// 当前值归位
			a[j + 1] = temp;
		}
	}
```

### 2.希尔排序(缩小增量排序)[不稳定]

复杂度 平均 O(n^1.3) 最好O(n) 最差O(n^s)[1<s<2] 空间O(1)

内循环通过模拟并行的方式完成分组的内部直接插入排序，而不是一个一个分组分组的排，在10w的随机数据20w的随机数据均表现优异。

```java
public void shellSort(int[] a) {
		if (null == a || a.length < 2) {
			return;
		}
		for (int d = a.length/2; d > 0; d/=2) {
			// 从1B开始先和1A比较 然后2A与2B...然后再1C向前与同组的比较
			for (int i = d; i < a.length; i++) {
				// 内部直接插入
				int temp = a[i];
				int j = i - d;
				while (j >=0 && temp < a[j]) {
					a[j+d] = a[j];
					j -= d;
				}
				a[j+d] = temp;
			}
		}
	}
```

### 3.冒泡排序[稳定]

复杂度：O(n^2) - O(n) - O(n^2) - O(1)[平均 - 最好 - 最坏 - 空间复杂度]

```java
public void bubbleSort(int[] a) {
		if (null == a || a.length < 2) {
			return;
		}
		boolean flag;
		for (int i = 0; i < a.length-1; i++) {
			flag = false;
			for (int j = 0; j < a.length-1-i; j++) {
				if (a[j] > a[j+1]) {
					int temp = a[j];
					a[j] = a[j+1];
					a[j+1] = temp;
					flag = true;
				}
			}
			if (false == flag) {
				return;
			}
		}
	}
```

### 4.选择排序[不稳定]

原理：每次从无序序列选取最小的

复杂度：O(n^2) - O(n^2) - O(n^2) - O(1)[平均 - 最好 - 最坏 - 空间复杂度]

```java
public void selectSort(int[] a) {
		if (null == a || a.length < 2) {
			return;
		}
		for (int i = 0; i < a.length; i++) {
			int k = i;
			for (int j = i + 1; j < a.length; j++) {
				if (a[j] < a[k]) {
					k = j;
				}
			}
			if (k!=i) {
				int temp = a[k];
				a[k] = a[i];
				a[i] = temp;
			}
		}
	}
```

### 5.归并排序[稳定]

原理：采用分治法

复杂度：O(nlogn) - O(nlgn) - O(nlgn) - O(n)[平均 - 最好 - 最坏 - 空间复杂度]

```java
// 排序
	public void mergeSort(int[] a, int low, int high) {

		int mid = (low + high) / 2;
		if (low < high) {
			// 左边排序
			mergeSort(a, low, mid);
			// 右边排序
			mergeSort(a, mid + 1, high);
			// 有序序列合并
			merge(a, low, mid, high);
		}
	}
	
	// 合并
	private void merge(int a[], int low, int mid, int high) {
		// 临时数组
		int[] temp = new int[high - low + 1];
		// 左指针
		int i = low;
		// 右指针
		int j = mid + 1;
		// 临时数组索引
		int k = 0;
		
		while (i <= mid && j <= high) {
			if (a[i] < a[j]) {
				temp[k++] = a[i++];
			} else {
				temp[k++] = a[j++];
			}
		}
		
		// 把左边剩余的数移入数组  
		while (i <= mid) {
			temp[k++] = a[i++];
		}
		
		// 把右边剩余的数移入数组  
		while (j <= high) {
			temp[k++] = a[j++];
		}
		
		// 注意这里是low + t
		for (int t = 0; t < temp.length; t++) {
			a[low + t] = temp[t];
		}
	}
```

### 6.快速排序[不稳定]

原理：分治+递归

复杂度：O(nlgn) - O(nlgn) - O(n^2) - O(1)[平均 - 最好 - 最坏 - 空间复杂度]

栈空间0(lgn) - O(n)

```java
// 固定基准
public void quickSort(int[] a, int low, int high) {
		if (null == a || a.length < 2) {
			return;
		}
		if (low < high) {
			int mid = partition(a, low, high);
			quickSort(a, low, mid-1);
			quickSort(a, mid+1, high);
		}
	}

	private int partition(int[] a, int low, int high) {
		int pivot = a[low];

		while (low < high) {
			// 注意等于，否则死循环
			while (low < high && a[high] >= pivot) {
				high--;
			}
			a[low] = a[high];
			// 注意等于，否则死循环
			while (low < high && a[low] <= pivot) {
				low++;
			}
			a[high] = a[low];
		}
		a[low] = pivot;
		
		return low;
	}
```

### 7.堆排序[不稳定]

堆一般指二叉堆。

复杂度：O(nlogn) - O(nlgn) - O(nlgn) - O(1)[平均 - 最好 - 最坏 - 空间复杂度]

大顶堆实现从小到大的升序排列，小顶堆一般用于构造优先队列

```java
	public void heapSort(int[] a) {
		if (null == a || a.length < 2) {
			return;
		}
		
		buildMaxHeap(a);
		
		for (int i = a.length - 1; i >= 0; i--) {
			int temp = a[0];
			a[0] = a[i];
			a[i] = temp;
			
			adjustHeap(a, i, 0);
		}
	}

	// 建堆
	private void buildMaxHeap(int[] a) {
		int mid = a.length / 2;
		for (int i = mid; i >= 0; i--) {
			adjustHeap(a, a.length, i);
		}
	}
	
	// 递归调整堆
	private void adjustHeap(int[] a, int size, int parent) {
		int left = 2 * parent + 1;
		int right = 2 * parent + 2;
		
		int largest = parent;
		if (left < size && a[left] > a[parent]) {
			largest = left;
		}
		
		if (right < size && a[right] > a[largest]) {
			largest = right;
		}
		
		if (parent != largest) {
			int temp = a[parent];
			a[parent] = a[largest];
			a[largest] = temp;
			adjustHeap(a, size, largest);
		}
	}
```

### 8.基数排序[稳定]

原理：分配加收集

复杂度： O(d(n+r)) r为基数d为位数 空间复杂度O(n+r)

```java
// 基数排序
	public void radixSort(int[] a, int begin, int end, int digit) {
		// 基数
		final int radix = 10;
		// 桶中的数据统计
		int[] count = new int[radix];
		int[] bucket = new int[end-begin+1];
		
		// 按照从低位到高位的顺序执行排序过程
		for (int i = 1; i <= digit; i++) {
			// 清空桶中的数据统计
			for (int j = 0; j < radix; j++) {
				count[j] = 0;
			}
			
			// 统计各个桶将要装入的数据个数
			for (int j = begin; j <= end; j++) {
				int index = getDigit(a[j], i);
				count[index]++;
			}
			
			// count[i]表示第i个桶的右边界索引
			for (int j = 1; j < radix; j++) {
				count[j] = count[j] + count[j - 1]; 
			}
			
			// 将数据依次装入桶中
            // 这里要从右向左扫描，保证排序稳定性 
			for (int j = end; j >= begin; j--) {
				int index = getDigit(a[j], i);
				bucket[count[index] - 1] = a[j];
				count[index]--;
			}
			
			// 取出，此时已是对应当前位数有序的表
			for (int j = 0; j < bucket.length; j++) {
				a[j] = bucket[j];
			}
		}
	}
	
	// 获取x的第d位的数字，其中最低位d=1
	private int getDigit(int x, int d) {
		String div = "1";
		while (d >= 2) {
			div += "0";
			d--;
		}
		return x/Integer.parseInt(div) % 10;
	}
}
```

排序代码地址 [https://github.com/Lemonjing/TinyCoding/tree/master/src/main/java/com/tinymood/javase/sort](https://github.com/Lemonjing/TinyCoding/tree/master/src/main/java/com/tinymood/javase/sort)

性能测试代码地址 [https://github.com/Lemonjing/TinyCoding/tree/master/src/test/java/com/tinymood/javase/sort](https://github.com/Lemonjing/TinyCoding/tree/master/src/test/java/com/tinymood/javase/sort)