---
title: 483. Smallest Good Base
date: 2018-11-05 13:58:34
category: LeetCode
---

#  题目

**最小好基数**

对于给定的整数 n, 如果n的k（k>=2）进制数的所有数位全为1，则称 k（k>=2）是 n 的一个***好基数***。

以字符串的形式给出 n, 以字符串的形式返回 n 的最小好进制。

**示例 1：**

```
输入："13"
输出："3"
解释：13 的 3 进制是 111。
```

**示例 2：**

```
输入："4681"
输出："8"
解释：4681 的 8 进制是 11111。
```

**示例 3：**

```
输入："1000000000000000000"
输出："999999999999999999"
解释：1000000000000000000 的 999999999999999999 进制是 11。
```

 

**提示：**

1. n的取值范围是 [3, 10^18]。
2. 输入总是有效且没有前导 0。

# 思路一

输入给定数为n，我们遍历[`2` , `n-1`]

​		如果当前遍历值`i`为`n`的基数，则找到了最小好基数

> 注意，`n-1`一定是`n`的一个好基数

那么，如何判断`i`为`n`的好基数呢？

若`i`为`n`的好基数，由辗转相除法的逆过程可知，`n`和`i`一定满足这样一个关系：

$ ((( 1 * k + 1)*k + 1)*k + 1)*k+1……=n$ ，其中k为基数。

# 代码一

```python
class Solution:
    def smallestGoodBase(self, n):
        def checkBase(base, n):
            currentVal = 1
            while currentVal < n:
                currentVal = currentVal * base + 1
            return currentVal == n
        
        thisNum = int(n)
        for i in range(2, thisNum):
            if checkBase(i, thisNum):
                return str(i)
```

#  思路二

先利用数学知识对问题进行分析，将问题简化，然后在去用代码实现。

n 为输入的整数，k为基，m为多项式的个数，根据好基数的定义知，n可以展开为如下式子：

$n=1+k+k^2+…+k^{m-1} 	\tag{1}$

可以明显看出，（1）式中的元素为等比数列。根据等比数列求和公式得：

$n=\frac{k^m-1}{k-1}	 \tag{2}$

此题中基数k>=2 ，我们要求的是最小的好基数，即求使（1）式成立的最小的k。观察式子可以发现，当n恒定的时候，k越小则m越大，m代表多项式的个数，由于k至少为2，那么（1）式肯定至少有两项，则m>=2。m的最大值是多少呢？当k取最小值2时，m的值最大，即数字n用二进制表示的时候可拆分出的项最多，计算得到m最大为$int(log_2(m+1)) + 1$，所以，

$ 2<= m <int(log_2(m+1)) + 1 	\tag{3}$

从（1）式中可以明显看出：$k^{m-1}<n$，将（3）式进行变形可得 $ k<n^{ \frac{1}{m-1} }$。此时会想到，如果$n^{ m-1}<k+1$也成立的话，就会为我们的计算带来很大的方便，因为如果$ k<n^{ \frac{1}{m-1} }<k+1$ 的话，我们对$ n^{ \frac{1}{m-1} }$取整就可以得到k的值。事实上，(4) 式确实是成立的。

$n^{ \frac{1}{m-1} }<k+1	\tag{4}$

由（4）式可得，

$n<(k+1)^{m-1} 	\tag{5}$ 

将（5）右边的式子进行牛顿二项式展开，与（1）式进行比对，可明显看出（5）式是成立的。

# 代码二

```python
import math   
class Solution(object):
    def smallestGoodBase(self, n):
        """
        :type n: str
        :rtype: str
        """        
        num = int(n)
        thisLen = int(math.log(num,2)) + 1
        while thisLen > 2:
            thisBase = int(num ** (1.0/(thisLen - 1)))
            if num * (thisBase - 1) == thisBase ** thisLen - 1:
                return str(thisBase)
            thisLen -= 1
        return str(num - 1)
```

```if num * (thisBase - 1) == thisBase ** thisLen - 1:```不能写成```if num == (base**goodBaseLen - 1) / (base - 1): ```，因为计算机精度的问题，当输入n很大时，一些测试用例会因为精度问题导致后面的表达式出错。所以，在计算机的运算式中，能用乘法表示就不用除法。