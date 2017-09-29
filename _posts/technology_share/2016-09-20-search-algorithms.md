---
layout: post
title: 常用的几种查找算法
category: 技术分享
tags: 查找、算法
---


- 顺序查找

顺序查找针对有序或无序的数组都可以，这种方法是从数据结构线形表的一端开始，顺序扫描，按数组下标顺序逐一和关键字比对，直到找到关键字为止，若扫描结束仍没有找到关键字，表示查找失败。它的时间复杂度为`O(n)`。

```
/**顺序查找平均时间复杂度 O（n）
 * @param searchKey 要查找的值
 * @param array 数组（从这个数组中查找）
 * @return  查找结果（数组的下标位置）
 */
public int orderSearch(int searchKey, int[] array) {
    if (array == null || array.length < 1)
        return -1;
    for (int i = 0; i < array.length; i++) {
        if (array[i] == searchKey) {
            return i;
        }
    }
    return -1;
}
```



- 二分查找（折半查找）

二分查找是针对有序数组的查找算法，它是从数组的中间元素开始，如果中间元素正好是要查找的元素，则搜素过程结束；如果关键字大于或者小于中间元素，则在数组大于或小于中间元素的那一半中查找，而且跟开始一样从中间元素开始比较。如果在某一步骤数组为空，则代表找不到。这种搜索算法每一次比较都使搜索范围缩小一半。

```
/**二分查找平均时间复杂度 O（logn）
 * @param searchKey 要查找的值
 * @param array 数组（从这个数组中查找）
 * @return  查找结果（数组的下标位置）
 */
public int binarySearch(int searchKey, int[] array) {
    int low = 0;
    int high = array.length - 1;
    int middle;
    while (low <= high) {
        middle = (low + high) / 2;
        if (searchKey == array[middle]) {
            return middle;
        } else if (searchKey < array[middle]) {
            high = middle - 1;
        } else {
            low = middle + 1;
        }
    }
    return -1;
}
```


- 二叉排序树查找

二叉查找树是先对待查找的数据进行生成树，确保树的左分支的值小于右分支的值，然后在就行和每个节点的父节点比较大小，查找最适合的范围。 这个算法的查找效率很高，但是如果使用这种查找方法要首先创建树。

二叉查找树:也叫二叉搜索树，或称二叉排序树,或者是一棵空树，或者是具有下列性质的二叉树:

    - 1.若任意节点的左子树不空，则左子树上所有结点的值均小于它的根结点的值；
    - 2.若任意节点的右子树不空，则右子树上所有结点的值均大于它的根结点的值；
    - 3.任意节点的左、右子树也分别为二叉查找树。

二叉查找树性质：对二叉查找树进行中序遍历，即可得到有序的数列。

复杂度分析：它和二分查找一样，插入和查找的时间复杂度均为O(logn)，但是在最坏的情况下仍然会有O(n)的时间复杂度。原因在于插入和删除元素的时候，树没有保持平衡。


```
// 生成二叉排序树
public class TreeNode {

    private TreeNode left;
    private TreeNode right;
    private int data;

    public void insertNode(int data) {

        if (data < this.data) {
            if (this.left == null) {
                this.left = new TreeNode(data);
            } else {
                this.left.insertNode(data);
            }
        } else if (data > this.data) {
            if (this.right == null) {
                this.right = new TreeNode(data);
            } else {
                this.right.insertNode(data);
            }
        }

    }
}


private TreeNode binaryTreeSearch(TreeNode root, int data) {
    if (root == null) {
        return null;
    }
    //定义当前节点
    TreeNode current = root;

    while(current != null) {
        if (current.data < data) {
            //如果当前节点的值比value小，则从其右子树中开始找
            current = current.right;
        } else if (current.data > data) {
            //如果当前节点的值比value大，则从其左子树中开始找
            current = current.right;
        } else if (current.data == data) {
            //找到则返回这个节点
            return current;
        }
    }

    return null;
}
```



- 哈希查找

哈希查找是通过计算数据元素的存储地址进行查找的一种方法。它的时间复杂度为O(1)。哈希查找的本质是先将数据映射成它的哈希值。

哈希查找的操作步骤：首先用给定的哈希函数构造哈希表；然后根据选择的冲突处理方法解决地址冲突；最后在哈希表的基础上执行哈希查找。

哈希函数的规则是：通过某种转换关系，使关键字适度的分散到指定大小的的顺序结构中，越分散，则以后查找的时间复杂度越小，空间复杂度越高。

建立哈希表操作步骤：第一步是取数据元素的关键字key，计算其哈希函数值。若该地址对应的存储空间还没有被占用，则将该元素存入；否则执行第二步解决冲突。第二步是根据选择的冲突处理方法，计算关键字key的下一个存储地址。若下一个存储地址仍被占用，则继续执行第二步，直到找到能用的存储地址为止。

解决哈希冲突的方法一般有开放寻址法和链地址法。开放寻址法就是如果两个数据元素的哈希值相同，则在哈希表中为后插入的数据元素另外选择一个表项。当程序查找哈希表时，如果没有在第一个对应的哈希表项中找到符合查找要求的数据元素，程序就会继续往后查找，直到找到一个符合查找要求的数据元素，或者遇到一个空的表项。链地址法则是将哈希值相同的数据元素存放在一个链表中，在查找哈希表的过程中，当查找这个链表时，必须采用线性查找方法。


```
/**** 
 * Hash表检索数据 
 *  
 * @param hash 
 * @param hashLength 
 * @param key 
 * @return 
 */  
public static int searchHash(int[] hash, int hashLength, int key) {  
    // 哈希函数  
    int hashAddress = key % hashLength;  

    // 指定hashAdrress对应值存在但不是关键值，则用开放寻址法解决  
    while (hash[hashAddress] != 0 && hash[hashAddress] != key) {  
        hashAddress = (++hashAddress) % hashLength;  
    }  

    // 查找到了开放单元，表示查找失败  
    if (hash[hashAddress] == 0)  
        return -1;  
    return hashAddress;  

}  

/*** 
 * 数据插入Hash表 
 *  
 * @param hash 哈希表 
 * @param hashLength 
 * @param data 
 */  
public static void insertHash(int[] hash, int hashLength, int data) {  
    // 哈希函数  
    int hashAddress = data % hashLength;  

    // 如果key存在，则说明已经被别人占用，此时必须解决冲突  
    while (hash[hashAddress] != 0) {  
        // 用开放寻址法找到  
        hashAddress = (++hashAddress) % hashLength;  
    }  

    // 将data存入字典中  
    hash[hashAddress] = data;  
}  

int[] nums = new int[] {23, 65, 12, 3, 8, 76, 12, 345, 90, 21, 75, 34, 61};
int[] hash = new int[nums.length];
for (int num : nums) {
    insertHash(hash, nums.length, num);
}
System.out.println("哈希查找位置为：" + searchHash(nums, nums.length, 12));
```


- 分块查找

分块查找又称索引顺序查找，它是顺序查找的一种改进方法。算法思想：将n个数据元素“按块有序”划分为m块（m ≤ n）。每一块中的结点不必有序，但块与块之间必须”按块有序”；即第1块中任一元素的关键字都必须小于第2块中任一元素的关键字；而第2块中任一元素又都必须小于第3块中的任一元素，即下述步骤:

1.首先将查找表分成若干块，在每一块中数据元素的存放是任意的，但块与块之间必须是有序的（假设这种排序是按关键字值递增的，也就是说在第一块中任意一个数据元素的关键字都小于第二块中所有数据元素的关键字，第二块中任意一个数据元素的关键字都小于第三块中所有数据元素的关键字，依次类推）；

2.建立一个索引表，把每块中最大的关键字值按块的顺序存放在一个辅助数组中，这个索引表也按升序排列；

3.查找时先用给定的关键字值在索引表中查找，确定满足条件的数据元素存放在哪个块中，查找方法既可以是折半方法，也可以是顺序查找；

4.再到相应的块中顺序查找，便可以得到查找的结果。

```
/**
 * 分块查找
 *
 * @param index 索引表，其中放的是各块的最大值
 * @param st 顺序表
 * @param key 要查找的值
 * @param m 顺序表中各块的长度相等，为m
 * @return
 */
public int blockSearch(int[] index, int[] st, int key, int m) {
    // 在序列st数组中，用分块查找方法查找关键字为key的记录
    // 在index[] 中折半查找，确定要查找的key属于哪个块中
    int i = binarySearch(key, index);
    if (i >= 0) {
        int j = i > 0 ? i * m : i;
        int len = (i + 1) * m;
        // 在确定的块中用顺序查找方法查找key
        for (int k = j; k < len; k++) {
            if (key == st[k]) {
                return k;
            }
        }
    }
    return -1;
}
```




