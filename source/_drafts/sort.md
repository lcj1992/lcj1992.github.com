---
layout: post
title: 数据结构学习笔记
date: 2014-11-09
categories: data_structure
keywords:
    - 数组
    - 链表
    - 树
    - 图
    - 遍历
    - 排序
---

3种基本数据类型
* 基本类型
* 组合类型
* 抽象数据类型ADT

4种基本数据结构
* 线性表
    * 数组
    * 链表
    * 栈
    * 队列
    * 串
* 树
* 图
* 哈希表

6种常见数据运算
* 增
* 删
* 改
* 查
* 排序
* 遍历


ADT 抽象数据类型名{
    数据对象: <数据对象的定义>
    数据关系: <数据关系的定义>
    基本操作: <基本操作的定义>
} ADT 抽象数据类型名


# 栈

# 队列

# LRU

# 树
树的定义：
节点的度：一个节点含有的子树的个数称为该节点的度；
树的度：一棵树中，最大的节点的度称为树的度；
叶节点或终端节点：度为零的节点；
非终端节点或分支节点：度不为零的节点；
父亲节点或父节点：若一个节点含有子节点，则这个节点称为其子节点的父节点；
孩子节点或子节点：一个节点含有的子树的根节点称为该节点的子节点；
兄弟节点：具有相同父节点的节点互称为兄弟节点；
节点的层次：从根开始定义起，根为第1层，根的子节点为第2层，以此类推；
深度：对于任意节点n,n的深度为从根到n的唯一路径长，根的深度为0；
高度：对于任意节点n,n的高度为从n到一片树叶的最长路径长，所有树叶的高度为0；
堂兄弟节点：父节点在同一层的节点互为堂兄弟；
节点的祖先：从根到该节点所经分支上的所有节点；
子孙：以某节点为根的子树中任一节点都称为该节点的子孙。
森林：由m（m>=0）棵互不相交的树的集合称为森林；

树的分类：
无序树：树中任意节点的子节点之间没有顺序关系，这种树称为无序树，也称为自由树；
有序树：树中任意节点的子节点之间有顺序关系，这种树称为有序树；
    二叉树：每个节点最多含有两个子树的树称为二叉树；
        完全二叉树：对于一颗二叉树，假设其深度为d（d>1）。除了第d层外，其它各层的节点数目均已达最大值，且第d层所有节点从左向右连续地紧密排列，这样的二叉树被称为完全二叉树；
            满二叉树：所有叶节点都在最底层的完全二叉树；
        平衡二叉树（AVL树）：当且仅当任何节点的两棵子树的高度差不大于1的二叉树；
        排序二叉树(二叉查找树（英语：Binary Search Tree))：也称二叉搜索树、有序二叉树；
    霍夫曼树：带权路径最短的二叉树称为霍夫曼树或最优二叉树；
    B树：一种对读写操作进行优化的自平衡的二叉查找树，能够保持数据有序，拥有多于两个子树

## 存储
父节点表示法

孩子链表表示法


边界case

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
