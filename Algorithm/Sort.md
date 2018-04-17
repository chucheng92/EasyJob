## 排序算法算法复习

### 1.插入排序 **稳定**

原理：从有序序列中选择合适的位置进行插入

复杂度：最好 - 最坏 - 平均 O(n) - O(n^2) - O(n^2) 

```java
public void selectionSort(int[] a) {
	if (null ==a || a.length < 2) {
		return;
	}
	for (int i = 1; i < a.length; i++) {
		int temp = a[i]; // 暂存
		int j = i - 1;
		while (j >= 0 && temp < a[j]) {
			a[j+1] = a[j];
			j--;
		}
		a[j+1] = temp;
	}
}
```

### 2.冒泡排序 **稳定**

原理：相邻两个元素比较大小进行交换，一趟冒泡后会有一个元素到达最终位置

复杂度：最好 - 最坏 - 平均 O(n) - O(n^2) - O(n^2) 

```java
public void bubbleSort(int[] a) {
	if (null == a || a.length < 2) {
		return;
	}
	boolean flag;
	for (int i = 0; i < a.length - 1; i++) {
		flag = false;
		for (int j = 0; j < a.length - 1 - i; j++) {
			if (a[j] > a[j+1]) {
				int temp = a[j];
				a[j] = a[j+1];
				a[j+1] = temp;
				flag = true;
			}
			if (flag == false) {
				return;
			}
		}
	}
}
```

### 3.希尔排序(缩小增量排序) **不稳定**

按步长进行分组，组内直接插入，缩小增量再次进行此步骤，增量为1时相当于一次直接插入。

复杂度：最好O(n) - 最坏O(n^s 1<s<2) - 平均O(n^1.3)

```java
public void shellSort(int[] a) {
	if (null == a || a.length < 2) {
		return;
	}
	for (int d = a.length/2; d > 0; d/=2) {
		for (int i = d; i < a.length; i++) {
		// 内部直接插入
			int temp = a[i];
			int j = i - d;
			while (j >= 0 && temp < a[j]) {
				a[j+d] = a[j];
				j -= d;
			}
			a[j+d] = temp;
		}	
	}
}
``` 

### 4.选择排序 **不稳定**

原理：每次从无序序列选择一个最小的

复杂度：最好O(n^2) - 最坏O(n^2) - 平均O(n^2)

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
		if (k != i) {
			int temp = a[k];
			a[k] = a[i];
			a[i] = temp;
		}
	}
}

```

### 5.快速排序 **不稳定**

原理：分治+递归

复杂度：最好O(nlgn) - 最坏O(n^2) - 平均O(nlgn)

```java
public void quickSort(int[] a, int low, int high) {

	if (low < high) {
		int mid = partition(a, low, high);
		quickSort(a, low, mid - 1);
		quickSort(a, mid + 1, high);
	}
}

private int partition(int[] a, int low, int high) {
	int pivot = a[low];

	while (low < high) {
		while (low < high && a[high] >= pivot) {
			high--;
		}
		a[low] = a[high];
		while (low < high && a[low] <= pivot) {
			low++;
		}
		a[high] = a[low];
	}
	a[low] = pivot;

	return low;
}
```
选取pivot的方式：固定基准元 随机基准 三数取中

快排的优化：针对随机数组+有序数组+重复数组

1.当待排序序列的长度分割到一定大小后，使用插入排序<三数取中+插入排序>：效率提高一些，但是都解决不了重复数组的问题。

2.在一次分割结束后，可以把与Key相等的元素聚在一起，继续下次分割时，不用再对与key相等元素分割
<三数取中+插排+聚集相同元素>

### 6.归并排序 **稳定**

原理：两个有序序列的合并，方法：分治 + 递归

复杂度：最好O(nlgn) - 最坏O(nlgn) - 平均O(nlgn)

```java
public void mergeSort(int[] a, int low, int high) {
	int mid = (low + high) / 2;
	if (low < high) {
		//左边
		mergeSort(a, low, mid);
		//右边
		mergeSort(a, mid + 1, high);
		//有序序列归并
		merge(a, low, mid, high);
	}
}

private void merge(int[] a, int low, int mid, int high) {
	int[] temp = new int[high - low + 1];
	// 左指针
	int i = low;
	// 右指针
	int j = mid + 1;
	// 临时数组指针
	int k = 0;

	while (i <= mid && j <= high) {
		if (a[i] < a[j]) {
			temp[k++] = a[i++];
		} else {
			temp[k++] = a[j++];
		}
	}

	//左边剩余
	while (i <= mid) {
		temp[k++] = a[i++];
	}

	//右边剩余
	while (j <= high) {
		temp[k++] = a[j++];
	}

	// 倒出
	for (int t = 0; t < temp.length; t++) {
		a[t + low] = temp[t]; 
	}
}

```

### 堆排序

原理：利用堆的特性

复杂度：O(nlogn) [平均 - 最好 - 最坏]

```java
// 堆排序
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
	for (int i = a.length/2; i >= 0; i--) {
		adjustHeap(a, a.length, i);
	}
}

// 调整堆
private void adjustHeap(int[] a, int size, int parent) {
	int left = 2 * parent + 1;
	int right = 2 * parent + 2;
	int largest = parent;

	if (left < size && a[left] > a[largest]) {
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