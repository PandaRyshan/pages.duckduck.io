---
title: "二分查找 Binary Search"
author: aold619
date: 2022-08-25 18:00
categories: [Note]
tags: [algorithm, leetcode]
---

## 定义

二分查找主要是解决在“一堆数中找出指定的数”这类问题，而想要应用二分查找，需要满足以下特征：

* 存储在数组中
* 有序排列（无序使用二分法可以用作猜答案）

## 代码模板：

```java
start = 0; end = len - 1;
while (start <= end) {
    mid = (end - start) / 2 + start;
    if (nums[mid] < target) {
        start = mid + 1;
    } else {
        end = mid - 1;
    }
}
return
```

## 经典题型：
* [第一个错误版本](https://leetcode.cn/problems/first-bad-version/)
* [分割数组的最大值](https://leetcode.cn/problems/split-array-largest-sum/)
* [最长递增子序列](https://leetcode.cn/problems/longest-increasing-subsequence/)

