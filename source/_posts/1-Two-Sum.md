---
title: 1. Two Sum
date: 2018-11-06 20:21:08
category: LeetCode
---

# 题目

**两数之和**

给定一个整数数组和一个目标值，找出数组中和为目标值的**两个**数。

你可以假设每个输入只对应一种答案，且同样的元素不能被重复利用。

**示例:**

```bash
给定 nums = [2, 7, 11, 15], target = 9

因为 nums[0] + nums[1] = 2 + 7 = 9
所以返回 [0, 1]
```

# 思路一

遍历整个列表

​	如果（target-当前值）在这个列表内

​		返回 当前值的索引，（target-当前值）的索引

这个思路容易想到，但是时间复杂度较高。

# 代码一

```python
class Solution:
    def twoSum(self, nums, target):
        """
        :type nums: List[int]
        :type target: int
        :rtype: List[int]
        """
        for i in range(len(nums)):
            if target-nums[i] in nums[i+1:]:
                return i, nums.index(target-nums[i], i+1)
```

# 思路二

借助一个字典来存储：key为**当前值**，value为**当前值的索引**

## 代码二

```python
class Solution:
    def twoSum(self, nums, target):
        """
        :type nums: List[int]
        :type target: int
        :rtype: List[int]
        """
        d = {}
        for i in range(len(nums)):
            another = target - nums[i]
            if another in d.keys():
                return d[another], i
            d[nums[i]] = i
```