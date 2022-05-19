# Leetcode 跳跃游戏

## 题目描述 (难度中等)
给定一个非负整数数组 nums ，你最初位于数组的 第一个下标 。
数组中的每个元素代表你在该位置可以跳跃的最大长度。
判断你是否能够到达最后一个下标。

示例 1：

>输入：nums = [2,3,1,1,4]
输出：true
解释：可以先跳 1 步，从下标 0 到达下标 1, 然后再从下标 1 跳 3 步到达最后一个下标。

示例 2：
>输入：nums = [3,2,1,0,4]
输出：false
解释：无论怎样，总会到达下标为 3 的位置。但该下标的最大跳跃长度是 0 ， 所以永远不可能到达最后一个下标。

## 解题思路
* 能否到达最后一个下标，需要判断数组是否存在零。
   * 如果不存在零，则一定能到达最后一个下标。
   * 如果存在零，则看零前面的位置能不能跳过零，如果不能跳过，返回false。
```
public boolean canJump(int[] nums) {
        for (int i = 0; i < nums.length - 1; i++) {
		if (nums[i] == 0) {
				int j = i - 1;
				boolean canSkipZero = false;
				while (j >= 0) {
					if (nums[j] + j > i) {
						canSkipZero = true;
						break;
					}
					j--;
				}
				if (!canSkipZero) {
					return false;
				}
			}
		}
		return true;
    }
```

## 进阶
* 如果当前为零，不需要往前每次遍历，只需要记录前面能跳最远的最大值。
**前面能跳的最大值一定是能跳的最远的位置**
```
public boolean canJump(int[] nums) {
        //记录能跳的最远的下标
		int max = 0;
		for (int i = 0; i < nums.length - 1; i++) {
			if (nums[i] == 0 && max <= i) {
				return false;
			}
			max = Math.max(max,nums[i] + i);
		}
		return true;
    }
```
## 总结
* 能达到最后一个位置首先判断是否为零
    * 非零一定可以往后跳。
    * 为零需要判断前面的位置能不能跳过零的位置。
*  零前面的能不能跳过有两种方法
    * 往前遍历，看前面能不能跳过零的位置
    * 记录前面能跳最远的下标，判断跳的最远的位置是否大于零的位置
