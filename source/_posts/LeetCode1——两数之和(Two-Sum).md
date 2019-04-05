---
title: LeetCode1——两数之和(Two Sum)
date: 2018-09-18 11:27:36
tags: 
  - LeetCode
categories: 
  - LeetCode
---

### 题目：
给定一个整数数组和一个目标值，找出数组中和为目标值的两个数。
你可以假设每个输入只对应一种答案，且同样的元素不能被重复利用。


<!--more-->


示例:
给定 nums = [2, 7, 11, 15], target = 9
因为 nums[0] + nums[1] = 2 + 7 = 9
所以返回 [0, 1]

### 思路分析：
我们首先想到的是利用两层循环取出两个数，判断两个数的和是否等于目标值，如果相等，则输出两个数对应的下标值。这种暴力法很简单，相信大部分人都能想到，可是两层循环，虽然空间复杂度为O(1)，但是时间复杂度为O(n^2)。想要降低时间复杂度，那就必须要用空间来换时间。这里我们想到可以用HashMap将数组元素和下标存储起来，然后用target减去遍历到的数字，然后在HashMap中寻找是否有对应的差值，注意插值不能是遍历的数字自身。这样就可以将查找效率提高到常数级，大大降低时间复杂度。

### 解法1：暴力法

    public int[] twoSum(int[] nums, int target) {
        for (int i = 0; i < nums.length; i++) {
            for (int j = i + 1; j < nums.length; j++) {
                if (nums[j] == target - nums[i]) {
                    return new int[] { i, j };
                }
            }
        }
        throw new IllegalArgumentException("No two sum solution");
    }
**复杂度分析：**
时间复杂度：O(n^2)，对于每个元素，我们试图通过遍历数组的其余部分来寻找它所对应的目标元素，这将耗费 O(n)的时间。因此时间复杂度为 O(n^2)。
空间复杂度：O(1)。 
### 解法2：HashMap法

    public int[] twoSum(int[] nums, int target) {
        Map<Integer, Integer> map = new HashMap<>();
        for (int i = 0; i < nums.length; i++) {
            map.put(nums[i], i);
        }
        for (int i = 0; i < nums.length; i++) {
            int complement = target - nums[i];
            if (map.containsKey(complement) && map.get(complement) != i) {
                return new int[] { i, map.get(complement) };
            }
        }
        throw new IllegalArgumentException("No two sum solution");
    }
**复杂度分析：**
时间复杂度：O(n)，我们把包含有n个元素的列表遍历两次。由于哈希表将查找时间缩短到O(1)，所以时间复杂度为O(n)。
空间复杂度：O(n)，所需的额外空间取决于哈希表中存储的元素数量，该表中存储了n个元素。 