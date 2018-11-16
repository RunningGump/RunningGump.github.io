---
title: 258. Add Digits
date: 2018-11-07 10:35:45
category: LeetCode
---

# 题目

**各位相加**

给定一个非负整数 `num`，反复将各个位上的数字相加，直到结果为一位数。

**示例:**

```
输入: 38
输出: 2 
解释: 各位相加的过程为：3 + 8 = 11, 1 + 1 = 2。 由于 2 是一位数，所以返回 2。
```

# 思路一

循环嵌套，外层循环当各位相加的和` s<10`时，退出循环；若` s>10`，则将s作为新的num再次进行和各位求和。内层循环是求各位的相加的和。

#   代码

```python
class Solution:
    def addDigits(self, num):
        """
        :type num: int
        :rtype: int
        """
        while True:
            s = 0
            while num != 0:
                s += num % 10
                num //= 10
            if s < 10:
                return s
            else:
                num = s
```

# 思路二

先考虑num是否为0，如果为0则返回0；如果不是0，则求出该数对9取余的数，如果余数为0，则返回9，否则返回余数。

假设一个四位数num=ABCD，即，num=A\*1000+B\*100+C\*10+D=A+B+C+D+（A\*999+B\*999+C*9），这样A+B+C+D就是各位数的和，而（A\*999+B\*999+C\*9）能被9整除。如果A+B+C+D大于9了，则仍然可以对9取余。比如$A+B+C+D=x_1x_2$，则$num=x_1+x_2\times9+(A\times999+B\times99+c\times9)$。所以，num不是0时，直接对9取余即可。

## 代码二

```python
class Solution {
    public int addDigits(int num) {
        if(num == 0){
            return 0;
        }
        int remainder = num%9;
        if(remainder==0){
            return 9;
        }else{
            return remainder;
        }
    }
}
```

##  代码二精简版

```python
class Solution:
    def addDigits(self, num):
        """
        :type num: int
        :rtype: int
        """
        if num == 0:
            return 0
        else:
            return((num-1)%9 +1)
```

