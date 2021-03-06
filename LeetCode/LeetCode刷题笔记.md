# 要点总结
## 1.查找中，可以使用HashMap空间换时间减少查找速度
>比如第一题，给定target和一个数组，返回数组中和为target的数字的标号  

&emsp;&emsp;使用暴力解法，需要对数组中每一个元素遍历一遍数组，直至找到两个满足条件的数组元素，时间复杂度为O(n^2)
&emsp;&emsp;而使用HashMap，时间复杂度可以降低为O(n)，因为针对每一个数组元素，只需查询HashMap，而HashMap在有冲突的情况下，才会去遍历红黑树，时间复杂度为O(n)；无冲突情况下直接可以查询到结果，时间复杂度为O(1)

## 2.时间复杂度
#### 2.1 时间复杂度为O(n)
&emsp;&emsp;考虑使用二分法
递归算法的时间复杂度怎么计算？子问题个数乘以解决一个子问题需要的时间。

## 3.排序算法


## 4.股票最大利润类题目
### 4.1 不使用动态规划的
> 假设把某股票的价格按照时间先后顺序存储在数组中，请问买卖该股票一次可能获得的最大利润是多少？  
输入: [7,1,5,3,6,4]  
输出: 5  
解释: 在第 2 天（股票价格 = 1）的时候买入，在第 5 天（股票价格 = 6）的时候卖出，最大利润 = 6-1 = 5 。
     
来源：力扣（LeetCode）  
链接：https://leetcode-cn.com/problems/gu-piao-de-zui-da-li-run-lcof

题解：题目目的是找到a[j] - a[i](j>i)的最大差值，此时只需要不断记录最小值，遍历数组时将数组元素与最小值的差和最大差值作比较，判断是否更新最大差值。**并不是所有最大利润值都去用动态规划，本题想了半天也没想出来动态规划的方法，自己感觉动态规划倾向于处理上一步的结果从而得出本步骤的结果**


## 5.位运算
### 5.1 数组中数组出现的次数 

#### 5.1.1 除了一个数字出现一次，其他数字都出现了两次


#### 5.1.2 除了两个数字出现一次，其他数字都出现了两次


#### 5.1.3 除了一个数字出现一次，其他数字都出现了三次


## 6.栈、队列
### 6.1用栈实现队列
### 6.2用队列实现栈

## 7.指针


## 8.其他