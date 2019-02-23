---
layout: post
title: 内部排序
date: 2014-11-09
categories: ds
description: 常用的排序算法
keywords:
    - 内部排序
    - shell
    - quick
    - heap
---


	#include <stdio.h>

	void quickSort(int a[], int len);
	void shellSort(int a[], int len);
	void heapSort(int a[], int len);
	//建堆
	void createHeap(int a[], int len);
	//调整堆
	void adjustHeap(int a[], int len);
	//求堆高度
	int height(int len);
	//交换两值
	void swap(int *i, int *j);
	//打印数组
	void print(int a[], int len);

	int main(){
		int a[10] = { 7, 2, 4, 9, 7, 3, 5, 10, 2, 14 };
		print(a,10);
		//quickSort(a, 10);
		//shellSort(a, 10);
		heapSort(a, 10);
		print(a, 10);
		getchar();
	}

	void print(int a[], int len){
		for (int i = 0; i < len; i++){
			printf("%d   ", a[i]);
		}
		printf("\n");
	}

### 快排

	/*快排
	先把第一个位置的值保存起来，之后就利用这个空位置，前后交换，真tm经典啊。
	*/
	void quickSort(int a[], int len){
		int tmp = a[0];
		int start = 0;
		int end = len - 1;
		if (len < 1){
			return;
		}
		while (start < end){
			for (; start < end; end--){
				if (a[end] < tmp){
					swap(a + start, a + end);
					break;
				}
			}
			for (; start < end; start++){
				if (a[start] >tmp){
					swap(a + start, a + end);
					break;
				}
			}
		}
		a[start] = tmp;
		quickSort(a, start);
		quickSort(a + start + 1,len - start - 1);
	}

	void swap(int *i, int *j){
		int tmp = *i;
		*i = *j;
		*j = tmp;
	}

### 堆排  

	/*堆排
	首先你得知道什么是选择排，就是每次选出一个来，至于你怎么选，那就看你了
	堆排就是通过堆每次选出堆顶的元素，然后不断的调整堆，关键点也就是建堆和调整堆了
	*/
	void heapSort(int *a, int len){
		createHeap(a, len);
		while (len > 0){
			swap(a + len - 1, a);
			len--;
			adjustHeap(a,len);
		}
	}


	/*如何建堆呢？
	以大顶堆为例，在大顶堆的世界里有一条原则，老子总比小子牛逼。
	所以你得逐个调整，从最后一个元素开始，如果出现儿子比他爸大的情况，父子换位。
	直到第0个元素。但是呢，这循环一次，只能保证一层是满足
	“老子总比小子牛逼的”这条哲理的，所以你得这么循环level - 1次
	*/
	void createHeap(int a[], int len){
		int level = height(len);
		while (level--){
			int i = len - 1;
			while ((i - 1) / 2 >= 0){
				if (a[i] > a[(i - 1) / 2]){
					swap(a + i, a + (i - 1) / 2);
				}
				i--;
			}
		}
	}

	int height(int len){
		int level = 0;
		while (len){
			level++;
			len /= 2;
		}
		return level;
	}

	/*如何调整堆呢？
	在调整堆之前我们做的处理是什么，把堆顶的元素和最后一个元素调了个个，
	并把len减了1，但是我们不能保证最后一个元素是最大的啊，所以要调整。
	但是除了堆顶其余都是满足大顶堆的规则的，我们如果采用建堆时那样未免大材小用了，
	比较堆顶孩子的大小关系，将堆顶和两孩子中较大的交换位置，直到叶子，
	这样就要保证是大顶堆了 ，弹出堆顶元素，继续堆调整……也很简单吧。
	*/
	void adjustHeap(int a[], int len){
		int level = height(len);
		int index = 0;
		while (len--){
			int maxIndex = a[2 * index + 1] > a[2 * index + 2] ?
					(2 * index + 1) : (2 * index + 2);
			if (maxIndex > len){
				break;
			}
			if (a[index] < a[maxIndex]){
				swap(a + index, a + maxIndex);
				index = maxIndex;
			}
		}
	}

### shell排

	/*shell
	有问题
	*/
	void shellSort(int a[], int len){
		int pace[3] = { 1, 3, 5};
		int paceIndex = 2;
		while (paceIndex--){
			//单看这个for循环，有木有冒泡的影子，将pace[paceIndex]换成1,那就变成一模一的冒泡了
			//所以说shell和冒泡是一样一样的，easy！
			for (int i = 0; i < len; i = i + pace[paceIndex]){
				for (int j = 0; j < len - i - pace[paceIndex]; j++){
					if (a[j] > a[j + pace[paceIndex]]){
						swap(a + j, a + j + pace[paceIndex]);
					}
				}
			}
		}
	}

### 参考

[复杂度速查]<http://bigocheatsheet.com/>