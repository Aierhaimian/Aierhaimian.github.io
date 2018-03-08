---
layout:     post
title:      Sorting Algorithm---C Version
subtitle:   Summarize various sorting algorithms
date:       2018-03-08
author:     Earl Du
header-img: img/post-bg-3.jpg
catalog: true
tags:
    - Algorithm
---

> 精解各种排序算法（C语言实现）


# 排序算法相关术语解释： #

- **排序**：指将元素按照规定的顺序排列，通常有两种排序方法，升序排列和降序排列，**本文均以升序排序为例**。
- **稳定**：待排序元素中有a=b，排序前a在b之前，排序后a仍然在b之前，则该次排序是稳定的。

	**不稳定**：待排序元素中有a=b，排序前a在b之前，排序后b在a之前，则该次排序是不稳定的。
- **内部排序**：所有排序操作均在内存中完成。

	**外部排序**：由于需要排序的数据量太大，因此需要借助磁盘，将数据放在磁盘中，排序需要通过磁盘和内存间的数据传输才能够进行。

# 排序算法总结（图片来源于网络）： #

![](https://i.imgur.com/0yft2gn.png)

- n:数据规模
- k：“桶”的个数
- In-place：占用常数内存，不占用额外内存
- Out-place：占用额外内存

# 排序算法分类： #

![](https://i.imgur.com/7ribjOV.png)


----------

# 1. 冒泡排序（Bubble Sort） #

(1). 算法简介：

冒泡排序是一种简单的排序算法。它重复地访问要排序的序列，一次比较两个元素，如果它们的顺序错误，就把它们交换过来。访问序列的工作是重复地进行的，直到没有需要交换的两个元素为止，此时该序列已排序完成，对于大小为n的数组，需要n-1轮比较。

- 平均时间复杂度：O(n^2)
- 最好情况：O(n)
- 最坏情况：O(n^2)
- 空间复杂度：O(1)

(2). 算法描述：

- 比较相邻的两个元素，如果第一个比第二个大，交换两个元素顺序，否则不变；
- 对每一个相邻元素作同样的工作，从开始第一对元素到结尾的最后一对，这样经过一次遍历后序列的最后一个元素应该是最大的；
- 除了最后一个元素对整个序列重复以上过程；
- 重复以上三个步骤，直至排序完成。

(3). 冒泡排序动图演示（图片来源于网络）：

![](https://i.imgur.com/CjJbCbD.gif)

(4). 算法实现：

利用冒泡排序将数组data中的元素进行排序。data中元素的个数由size决定。而每个元素的大小由esize决定。

函数指针compare会指向一个用户定义的函数来比较元素大小。在递增排序中，


- 如果key1>key2，函数返回 1；
- 如果key1=key2，函数返回 0；
- 如果key1<key2，函数返回-1。

在递减排序中，返回值相反。当冒泡排序完成后data包含已排序的元素。算法实现如下：

	int bubble_sort(void *data, int size, int esize, int (*compare)(const void *key1, const void *key2)) {
	    char *a = data;
	    void *key;
	
	    if ((key = (char *)malloc(esize)) == NULL)
	        return -1;
	    for (int i = 0; i < size-1; ++i) {
	        for (int j = 0; j < size-1-i; ++j) {
	            memcpy(key, &a[j*esize],esize);
	            if (compare(key, &a[(j+1)*esize]) > 0) {
	                memcpy(&a[j*esize], &a[(j+1)*esize], esize);
	                memcpy(&a[(j+1)*esize], key, esize);
	            }
	        }
	    }
	    free(key);
	    return 0;
	}

(5). 算法改进：

设置一个标志位变量pos，用于记录每趟排序最后一次进行交换的位置。由于pos位置之后的记录均已交换到位,故在进行下一趟排序时只要扫描到pos位置即可。改进后算法实现如下：

	int bubble_sort_1(void *data, int size, int esize, int (*compare)(const void *key1, const void *key2)) {
	    char *a = data;
	    void *key;
	
	    if ((key = (void*)malloc(esize)) == NULL)
	        return -1;
	    int i = size - 1;
	    while (i > 0) {
	        int pos = 0;
	        for (int j = 0; j < i; ++j) {
	            memcpy(key, &a[j*esize], esize);
	            if (compare(key, &a[(j+1)*esize]) > 0) {
	                pos = j;
	                memcpy(&a[j*esize], &a[(j+1)*esize], esize);
	                memcpy(&a[(j+1)*esize], key, esize);
	            }
	        }
	        i = pos;
	    }
		free(key);
	    return 0;
	}

继续改进，传统冒泡排序中每一趟排序操作只能找到一个最大值或最小值，如果考虑在每趟排序中进行正向和反向两边冒泡的方法，一次可以得到两个最终值（最大值和最小值）。改进后算法实现如下：

	int bubble_sort_2(void *data, int size, int esize, int (*compare)(const void *key1, const void *key2)) {
	    char *a = data;
	    void *key;
	
	    if ((key = (void*)malloc(esize)) == NULL)
	        return -1;
	
	    int low = 0;
	    int high = size - 1;
	
	    while (low < high) {
	        for (int i = low; i < high; ++i) {
	            memcpy(key, &a[i*esize], esize);
	            if (compare(key, &a[(i+1)*esize]) > 0) {
	                memcpy(&a[i*esize], &a[(i+1)*esize], esize);
	                memcpy(&a[(i+1)*esize], key, esize);
	            }
	        }
	        -- high;
	
	        for (int i = high; i > low; --i) {
	            memcpy(key, &a[i*esize], esize);
	            if (compare(key, &a[(i-1)*esize]) < 0) {
	                memcpy(&a[i*esize], &a[(i-1)*esize], esize);
	                memcpy(&a[(i-1)*esize], key, esize);
	            }
	        }
	        ++ low;
	    }
		free(key);
	    return 0;
	}


# 2. 选择排序（Selection Sort） #

(1). 算法简介：

选择排序是一种简单直观的排序算法。首先在未排序的序列中找到最小元素，存放到排序序列的起始位置，然后再从剩余未排序元素中继续寻找最小元素，然后放到已排序序列末尾，以此类推，直至所有元素排列完毕。

- 平均时间复杂度：O(n^2)
- 最好情况：O(n^2)
- 最坏情况：O(n^2)
- 空间复杂度：O(1)

(2). 算法描述：

- 初始状态：无序区为arr[0...n-1]，有序区为空；
- 第i(i=1,2,3,...n-1)趟排序开始时，当前有序区和无序区分别为arr[0...i-2]和arr[i-1...n-1]。该趟排序从当前无序区中选出最小元素arr[k]，将它与无序区第一个元素arr[i-1]交换，使得有序区元素数加一，无序区减一；
- 当结束n-1趟排序后，排序序列整体有序。

(3). 选择排序动图演示（图片来源于网络）：

![](https://i.imgur.com/uqw7aXb.gif)

(4). 算法实现：

	int selection_sort(void *data, int size, int esize, int (*compare)(const void *key1, const void *key2)) {
	    char *a = data;
	    void *key;
	    int minIndx;
	
	    if ((key = (void*)malloc(esize)) == NULL)
	        return -1;
	
	    for (int i = 0; i < size-1; ++i) {
	        minIndx = i;
	        for (int j = i+1; j < size; ++j) {
	            if (compare(&a[minIndx*esize], &a[j*esize]) > 0) {
	                minIndx = j;
	            }
	        }
	        memcpy(key, &a[i * esize], esize);
	        memcpy(&a[i*esize], &a[minIndx*esize], esize);
	        memcpy(&a[minIndx*esize], key, esize);
	    }
	    free(key);
	    return 0;
	}


# 3. 插入排序（Insertion Sort） #

(1). 算法简介： 

从未排序序列中拿出一个元素，然后将其置于已排序序列的正确位置中。

- 平均时间复杂度：O(n^2)
- 最好情况：O(n)
- 最坏情况：O(n^2)
- 空间复杂度：O(1)

(2).算法描述：

- 从第一个元素开始，认为第一个元素已排序；
- 取下一个元素，与已排序元素从后向前分别比较；
- 如果已排序元素大于新元素，则从此元素移到下一个位置；
- 重复步骤三，直到已排序元素小于或等于新元素的位置；
- 将新元素插入该位置；
- 重复步骤二到五。

(3).插入排序动图演示（图片来源于网络）：

![](https://i.imgur.com/yJxz5iN.gif)

(4).算法实现：

	int insertion_sort(void *data, int size, int esize, int (*compare)(const void *key1, const void *key2)) {
	    char *a = data;
	    void *key;
	    int i, j;
	
	    if ((key = (char*)malloc(esize)) == NULL)
	        return -1;
	
	    for (i = 1; i < size; ++i) {
	        memcpy(key, &a[i*esize], esize);
	        j = i - 1;
	        while (j >= 0 && compare(&a[j*esize], key) > 0) {
	            memcpy(&a[(j+1)*esize], &a[j*esize], esize);
	            -- j;
	        }
	        memcpy(&a[(j+1)*esize], key, esize);
	    }
	    free(key);
	    return 0;
	}

(5). 算法改进：

在寻找插入位置时可以使用二分查找法。算法实现如下：

	int insertion_sort_1(void *data, int size, int esize, int (*compare)(const void *key1, const void *key2)) {
	    char *a = data;
	    void *key;
	    int low, high;
	
	    if ((key = (char*)malloc(esize)) == NULL)
	        return -1;
	
	    for (int i = 1;  i< size; ++i) {
	        memcpy(key, &a[i*esize], esize);
	        low = 0;
	        high = i-1;
	
	        while (low <= high) {
	            int mid = (high + low)/2;
	            if (compare(&a[mid*esize], key) > 0)
	                high = mid - 1;
	            else
	                low = mid + 1;
	        }
	        for (int j = i-1; j >= low; --j)
	            memcpy(&a[(j+1)*esize], &a[j*esize], esize);
	        memcpy(&a[(low)*esize], key, esize);
	
	    }
	    free(key);
	    return 0;
	}

# 4. 希尔排序（Shell Sort） #

(1). 算法简介：

希尔排序是插入排序的改进版，它与插入排序的不同之处在于，它会优先比较距离较远的元素。希尔排序又叫做缩小增量排序（Diminishing Increment Sort）。其核心在于间隔序列的设定。既可以提前设定好间隔序列，也可以动态定义间隔序列。动态定义间隔序列的算法是《算法（第4版》的合著者Robert Sedgewick提出的。

希尔排序是把记录按下标的一定增量分组，对每组使用直接插入排序算法排序；随着增量逐渐减少，每组包含的关键词越来越多，当增量减至1时，整个文件恰被分成一组，算法便终止。

- 平均时间复杂度：O(nlogn)
- 最好情况：O(nlog^2n)
- 最坏情况：O(nlog^2n)
- 空间复杂度：O(1)

(2). 算法描述：

先将整个待排序的记录序列分割成为若干子序列分别进行直接插入排序，具体算法描述：

- 选择一个增量序列t1,t2,...,tk，其中ti>tj，tk=1；
- 按增量序列个数k，对序列进行k趟排序；
- 每趟排序，根据对应的增量ti，将待排序虚了分割为若干长度为m的子序列，分别对个子序列进行直接插入排序。仅增量因子为1时，整个序列作为一个表处理，表长度即为整个序列的长度。

(3). 希尔排序图示（图片来源于网络）：

![](https://i.imgur.com/T7fWZ7E.jpg)

(4). 算法实现：

	int shell_sort(void *data, int size, int esize, int (*compare)(const void *key1, const void *key2)) {
	    char *a = data;
	    void *tmp;
	    int gap = 1;
	    int i, j;
	
	    if ((tmp = (char*)malloc(esize)) == NULL)
	        return -1;
	
	    while (gap < size/2) {
	        gap = gap*2 + 1;
	    }
	
	    for (gap; gap > 0 ; gap /= 2) {
	        for (i = gap; i < size; ++i) {
	            memcpy(tmp, &a[i*esize],esize);
	            for (j = i-gap; j >= 0 && (compare(&a[j*esize], tmp) > 0) ; j-=gap) {
	                memcpy(&a[(j+gap)*esize], &a[j*esize], esize);
	            }
	            memcpy(&a[(j+gap)*esize], tmp, esize);
	        }
	    }
	    free(tmp);
	    return 0;
	}




# 5. 归并排序（Merge Sort）#

(1). 算法简介：

与选择排序一样其性能不受输入数据的影响，但是表现比选择排序好，其代价就是需要额外的内存空间。归并排序是建立在归并操作上的一种排序算法。该算法是分治法（Divide and Conquer）的一种典型应用。归并排序是一种弄稳定的排序方法，它将已经有序的子序列合并，得到完全有序的序列，即先使得每个子序列有序，再使得子序列段间有序。若是将两个有序序列合并成一个有序序列，则称为二路归并排序。

- 平均时间复杂度：O(nlogn)
- 最好情况：O(nlogn)
- 最坏情况：O(nlogn)
- 空间复杂度：O(n)

(2). 分治法思路：

- 分：将数据集等分为两半；
- 治：分别在两个部分用递归的方式继续使用归并排序法；
- 合：将分开的两个部分合并成一个有序的数据集。

(3). 算法描述：

- 把长度为n的输入序列分成两个长度为n/2的子序列；
- 对这两个子序列分别采用归并排序；
- 将两个排序号的子序列合并成一个最终的排序序列。

(4). 归并排序的过程：

![](https://i.imgur.com/KomdHHw.jpg)

(5). 归并排序动图演示（图片来源于网络）：

![](https://i.imgur.com/aUSJNUM.gif)

(6). 算法实现：

	/* merge */
	/* 将data中i到j之间的数据集与j+1到k之间的数据集合并成一个i到k的有序数据集 */
	static int merge(void *data, int esize, int i, int j, int k, int (*compare)(const void *key1, const void *key2)) {
	    char *a = data,
	         *m;
	    int ipos,
	        jpos,
	        mpos;
	
	    /* Initialize the counters used in merging. */
	    ipos = i;
	    jpos = j + 1; //最初，指向每个有序集的头部
	    mpos = 0;
	
	    /* Allocate storage for the merged elements. */
	    if ((m = (char *)malloc(esize * ((k - i) + 1))) == NULL)
	        return -1;
	
	    /* Continue while either division has elements to merge. */
	    while (ipos <= j || jpos <= k) {
	        if (ipos > j) {
	            /* The left division has no more elements to merge. */
	            while (jpos <= k) {
	                memcpy(&m[mpos*esize], &a[jpos*esize], esize);
	                jpos++;
	                mpos++;
	            }
	            continue;
	        } else if (jpos > k) {
	            /* The right division has no more elements to merge. */
	            while (ipos <= j) {
	                memcpy(&m[mpos*esize], &a[ipos*esize], esize);
	                ipos++;
	                mpos++;
	            }
	            continue;
	        }
	
	        /* Append the next ordered element to the merged elements. */
	        if (compare(&a[ipos*esize], &a[jpos*esize]) < 0) {
	            memcpy(&m[mpos*esize], &a[ipos*esize], esize);
	            ipos++;
	            mpos++;
	        } else {
	            memcpy(&m[mpos*esize], &a[jpos*esize], esize);
	            jpos++;
	            mpos++;
	        }
	    }
	
	    /* Prepare to pass back the merged data. */
	    memcpy(&a[i*esize], m, esize*((k-i)+1));
	
	    /* Free the storage allocated for merging. */
	    free(m);
	
	    return 0;
	}
	/* merge_sort */
	//参数i和k定义当前进行排序的两个部分，其值分别初始化为0和size-1。
	int merge_sort(void *data, int size, int esize, int i, int k, int (*compare)(const void *key1, const void *key2)) {
	    int j;
	
	    /* Stop the recursion to divide the elements. */
	    if (i < k) {
	        /* Determine where to divide the elements. */
	        j = (int) ((i + k - 1) / 2);
	
	        /* Recursively sort the two divisions. */
	        if (merge_sort(data, size, esize, i, j, compare) < 0)
	            return -1;
	        if (merge_sort(data, size, esize, j+1, k, compare) < 0)
	            return -1;
	
	        /* Merge the two sorted divisions into a single sorted set. */
	        if (merge(data, esize, i, j, k, compare) < 0)
	            return -1;
	    }
	    return 0;
	}


**未完待续... ...**