---
title: 【algorithm】剑指office习题
tags: [algorithm]
date: 2018-4-6
---

JDK版本 1.8

## 二维数组中的查找

> 在一个二维数组中，每一行都按照从左到右递增的顺序排序，每一列都按照从上到下递增的顺序排序。请完成一个函数，输入这样的一个二维数组和一个整数，判断数组中是否含有该整数。  

```java
//测试用例:
// int[][] array = {{1,2,8,9},{2,4,9,12},{4,7,10,13},{6,8,11,15}};
//7,[[1,2,8,9],[2,4,9,12],[4,7,10,13],[6,8,11,15]]
public class Solution {
    public boolean Find(int target, int [][] array) {
        for(int[] i:array){
            for(int j:i){
                if(j == target){
                    return true;
                }
            }
        }
        return false;
    }
}

// 更优解
/* 思路
* 利用二维数组由上到下，由左到右递增的规律
* 矩阵是有序的，从左下角来看，向上数字递减，向右数字递增，
* 因此从左下角开始查找，当要查找数字比左下角数字大时。右移column++
* 要查找数字比左下角数字小时，上移row--
*/
public class Solution {
    public boolean Find(int target, int [][] array) {
        int row  = array.length - 1;
        int column = 0;
        while(row >= 0 && column < array[0].length){
            if (array[row][column] > target){
                row--;
            } else if (array[row][column] < target){
                column++;
            } else {
                return true;
            }
        }
        return false;
    }
}
```

## 替换空格

>请实现一个函数，将一个字符串中的空格替换成“%20”。例如，当字符串为We Are Happy.则经过替换之后的字符串为We%20Are%20Happy。

```java
// 测试用例:
// We Are Happy.
public class Solution {
    public String replaceSpace(StringBuffer str) {
    	return str == null? null:str.toString().replaceAll(" ","%20");
    }
}
```