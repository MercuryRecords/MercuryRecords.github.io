---
layout: post
title: draft - 代码随想录
date: 2024-13-41 13:00:00 +0800
categories: [OC]
description: 没想好
keywords: A, B
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

# 数组

## 一、二分查找

> - 有序数组就可以想二分查找；
> - 如果数组中有重复元素，使用二分查找法返回的元素下标可能不是唯一的。

[704.二分查找](https://leetcode.cn/problems/binary-search/)


一种写法是认为目标在一个**左闭右闭**的区间中，也就是[left, right]。由此引出一些关键点：

- 初始赋值中 `r = nums.length - 1`
- while (left **<=** right)
- `nums[middle] != target` 时，左/右指针要“越过”middle

```java
class Solution {
    public int search(int[] nums, int target) {
        int l = 0, r = nums.length - 1, m;
        while(l <= r) {
            m = l + (r - l) / 2;
            if (nums[m] > target) {
                r = m - 1;
            } else if (nums[m] < target) {
                l = m + 1;
            } else return m;
        }
        return -1;
    }
}
```

### 相关题目

#### [35. 搜索插入位置](https://leetcode.cn/problems/search-insert-position/description/)

> 如果目标值不存在于数组中，返回它将会被按顺序插入的位置

这个变体只需要考虑目标值不存在情况下的返回值。此时要插入的位置 `pos` 满足 $nums[pos-1] < target < nums[pos]$ 。要综合考虑两种情况的话只需把右边的 $＜$ 改为 $≤$ 。所以本问题实际为「在一个有序数组中找第一个大于等于 $target$ 的下标」（下标可以等于数组实际长度）。

- 从 ***跳出循环的条件是 l > r*** 这个角度出发其实并不好想，应该适当分情况讨论，比如可以考虑内部的分支结构：
    - 退出循环时，target比最后一次的`middle`还大，应该插入到`middle + 1`的地方；最后执行的是`left = middle + 1`，可以返回`left`或`middle + 1`；
    - 退出循环时，target比最后一次的`middle`还小，应该插入到`middle`的地方；最后执行的是`right = middle - 1`，可以返回`middle`或`right + 1`；
    - 注意到二分搜索跳出循环时`left > right`，其实刚好是`left == right + 1`
    - 综上可以知道只需要把`return -1;`改为`return l;`

- 类似地，如果题目为「在一个有序数组中找最后一个小于等于 $target$ 的下标」，也可以分析出只需要把`return -1;`改为`return r;`。

#### [34. 在排序数组中查找元素的第一个和最后一个位置](https://leetcode.cn/problems/find-first-and-last-position-of-element-in-sorted-array/description/)

一个自然的想法是拆成两个二分来做。其实先做[278. 第一个错误的版本](https://leetcode.cn/problems/first-bad-version/solutions/824522/di-yi-ge-cuo-wu-de-ban-ben-by-leetcode-s-pf8h/)应该更好想一点。在更新边界的时候我一开始写了如下形式的代码：

```java

else {
    r = m - 1;
    if (nums[m] == target) {
        ret = r;
    }
}

```

实际上内部这个if是完全没必要的，只要数据保证要找的元素存在，二分查找最后总能找到边界，当然注意循环结束时的ret值并非要找的满足条件的边界值，需要有±1的调整。


注意数据类型转换：
- [69. x 的平方根](https://leetcode.cn/problems/sqrtx/description/)
- [367. 有效的完全平方数](https://leetcode.cn/problems/valid-perfect-square/description/)


## 二、双指针

### 相关题目

- [27. 移除元素](https://leetcode.cn/problems/remove-element/description/): 相当经典简单的题，不多说了

- [26. 删除有序数组中的重复项](https://leetcode.cn/problems/remove-duplicates-from-sorted-array/description/)：依然是双指针，感觉这种一眼类似的题可以先把结构写出来再想怎么改
- [283. 移动零](https://leetcode.cn/problems/move-zeroes/description/): 比26题还简单
- [844. 比较含退格的字符串](https://leetcode.cn/problems/backspace-string-compare/description/): 贴一下代码，倒着考虑怎么退格，主要是不能想简单了，需要用skip值来记录有多少个 **#** 要退格。话说这个算双指针的变种吧？

```Java
class Solution {
    public boolean backspaceCompare(String s, String t) {
        int sfast = s.length() - 1, tfast = t.length() - 1;
        int skipS = 0, skipT = 0;
        while (sfast >= 0 || tfast >= 0) {
            if (sfast >= 0 && s.charAt(sfast) == '#') {
                sfast --;
                skipS ++;
                continue;
            } else if (skipS > 0) {
                sfast --;
                skipS --;
                continue;
            }
            if (tfast >= 0 && t.charAt(tfast) == '#') {
                tfast --;
                skipT ++;
                continue;
            } else if (skipT > 0) {
                tfast --;
                skipT --;
                continue;
            }
            if (sfast >= 0 && tfast >= 0) {
                if (s.charAt(sfast) != t.charAt(tfast)) {
                    return false;
                } else {
                    sfast--;
                    tfast--;
                }
            } else {
                if (sfast >= 0 || tfast >= 0) {
                    return false;
                }
            }
        }
        return true;
    }
}
```

- [977. 有序数组的平方](https://leetcode.cn/problems/squares-of-a-sorted-array/description/): 双指针用于先把大的平方数插入到返回数组的末尾

## 三、滑动窗口

### 相关题目

- [209. 长度最小的子数组](https://leetcode.cn/problems/minimum-size-subarray-sum/description/): 这题得益于元素非负可以使用滑动窗口。如果元素可以是负数呢？
- [904. 水果成篮](https://leetcode.cn/problems/fruit-into-baskets/description/): 一般滑窗，但是写java居然没想起用哈希表，速度更快了（
