---
title: 最长递增子序列
date: 2021-03-21 22:39:15
categories: 算法
tags: 算法 动态规划
---

> 给你一个整数数组 nums ，找到其中最长严格递增子序列的长度。
>
> 子序列是由数组派生而来的序列，删除（或不删除）数组中的元素而不改变其余元素的顺序。例如，[3,6,2,7] 是数组 [0,3,1,6,2,2,7] 的子序列。

<!-- more -->

### 暴力解法

暴力解法就是枚举出所有的递增字串，从字串中获取长度最长的。暴力解法有点像全排列，只不过加了一个限制，当前数字必须比前面的都要大。暴力解法代码如下：

```java
private int ans;
// private List<List<Integer>> sequences = new ArrayList<>();
public int LISForce (int[] arr) {
    if (arr == null || arr.length == 0)
        return 0;
    getAllSubsequence(arr, 0, new ArrayList<Integer>());
    return ans;
}

private void getAllSubsequence(int[] arr, int index, List<Integer> list) {
    ans = Math.max(ans, list.size());
    // sequences.add(new ArrayList<>(list));
    for (int i = index; i < arr.length; i++) {
        if (list.size() == 0 || arr[i] > list.get(list.size() - 1)) {
            list.add(arr[i]);
            getAllSubsequence(arr, i + 1, list);
            list.remove(list.size() - 1);
        }
    }
}
```

这种暴力的方法的时间复杂度为 O(2<sup>n</sup>)，显然这个复杂度太高了，我们需要想一个复杂度更低的方法。

### 动态规划

现在我们只考虑以下标为 index 结尾的最长递增子序列的，然后求出以所有以 index 结尾的最长递增子序列长度的最大值，不就是全局最大值吗？那么怎么求呢？你可能会想到暴力方法，遍历 0 到 index-1 的所有元素，如果当前元素比前面的元素大，我们就可以在前面得到结果的基础上 + 1 作为自己的结果。这个结果可能有好几个，我们取最大值即可。

```java
public int LISDP (int[] arr) {
    if (arr == null || arr.length == 0)
        return 0;
    int[] dp = new int[arr.length];
    Arrays.fill(dp, 1);
    int res = 0;
    for (int i = 0; i < arr.length; i++) {
        for (int j = 0; j < i; j++) {
            if (arr[i] > arr[j]) {
                dp[i] = Math.max(dp[i], dp[j] + 1);
            }
        }
        res = Math.max(dp[i], res);
    }
    return res;
}
```

这种方法的时间复杂度是 O(n<sup>2</sup>)，比上面的暴力方法好多了。

### 二分优化

我们再换一种思考方式，我们认为长度为 n 的递增子序列的结尾值越小越好，这样就更可能与后面的值形成更长的递增子序列了。

```java
public int lengthOfLIS(int[] nums) {
    if (nums == null || nums.length == 0)
        return 0;
    int[] dp = new int[nums.length + 1];
    int k = 0;
    dp[0] = nums[0];
    for (int i = 1; i < nums.length; i ++) {
        if (nums[i] > dp[k]) {
            dp[++ k] = nums[i];
        } else {
            int index = searchRight(dp, nums[i], 0, k );
            if (dp[index] == nums[i]) {
                continue;
            } else if (dp[index] > nums[i]) {
                dp[index] = nums[i];
            }
        }
    }
    return k + 1;
}

public static int searchRight(int[] arr, int target, int low, int high) {
    if (low > high)
        throw new IllegalArgumentException();
    while (low <= high) {
        int mid = (high - low) / 2 + low;
        if (arr[mid] < target) {
            low = mid + 1;
        } else if (arr[mid] > target) {
            high = mid - 1;
        } else {
            return mid;
        }
    }
    return low;
}
```

