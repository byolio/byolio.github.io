---
layout: post
title: The Algorithmic Logic Behind Optimizing a 2D Array into a 1D Array
subtitle: 从本质上理解为何部分算法中可以将二维数组优化为一维数组
date: 2025-1-24
author: Byolio
header-img: img/oneToTwoOptimize.jpg
catalog: true
tags:
  - c++
  - algorithm
  - algorithm optimization
lang: en
---

> 🌐 [中文](https://www.byolio.blog/2025-1-24-oneToTwoOptimize/) | [English](https://www.byolio.blog/en/en/_posts/2025-1-24-oneToTwoOptimize/)

> This article mainly introduces the essence of the algorithmic logic behind optimizing a 2D array into a 1D array.
> Test link: [https://www.luogu.com.cn/problem/P1048](https://www.luogu.com.cn/problem/P1048)

## Introduction
When encountering problems involving trading time for space (such as DP, knapsack problems, subarray sum problems, etc.), we can optimize a 1D array into a variable, a 2D array into a 1D array, a 3D array into a 2D array, and so on. \
Among these, optimizing a 2D array into a 1D array is the most common. Therefore, this article will start from the essence of the algorithmic logic behind this optimization and introduce how to optimize a 2D array into a 1D array.

## Why Optimize a 2D Array into a 1D Array
Optimizing a 2D array into a 1D array reduces the space complexity from O(n * m) to O(m) or O(n), improving program space utilization. When data volume and values are large, this can significantly save space and disk I/O read/write time, enhancing program execution efficiency.

## Why Can a 2D Array Be Converted into a 1D Array
The essence is that the 2D array is traversed row by row during use, and it has the property of not being revisited. That is, the previous data will not be used in subsequent traversals. Therefore, we can replace unused data with upcoming data, effectively overwriting it, thus optimizing the 2D array into a 1D array. \
Similarly, any array with the property of not being revisited can be optimized: a 1D array can be optimized into a variable, a 3D array into a 2D array, a 4D array into a 3D array, and so on.

## What Is Handled When Optimizing a 2D Array into a 1D Array
What is handled is the dependency relationship between elements in the 2D array, i.e., the dependencies between elements in different rows and within each row. In a 2D array, we often need to copy elements from the previous row and use them as dependencies for basic calculations. Therefore, in a 1D array, we can retain the processed elements from the previous row as dependencies for the next row, provided they are not overwritten by new data. In problems with complex dependency relationships, we can use two 1D arrays for rolling optimization. In cases with sequential dependencies, we can even use a single 1D array to achieve optimization through forward or backward sequential dependencies, greatly saving space. \
Because this method primarily relies on dependency relationships, similar to the strict position dependency approach, it is used as a space optimization technique in algorithms that implement strict position dependencies.

## How to Optimize a 2D Array into a 1D Array
When optimizing a 2D array into a 1D array, attention must be paid to the order of dependencies: whether it is from front to back or from back to front. This determines the direction of the loop handling dependencies in the 1D array.\
Here is a classic 0/1 knapsack problem ([test link](https://www.luogu.com.cn/problem/P1048)): 
```c++
vector<int> dp;
vector<int> costs; // cost
vector<int> vals;  // value
int t, n;          // total cost, number of items
int main()
{
	cin >> t >> n;
	dp = vector<int>(t + 1, 0); 
	costs = vector<int>(n + 1, 0);
	vals = vector<int>(n + 1, 0);
	for (int i = 1; i <= n; ++i)
	{
		cin >> costs[i] >> vals[i]; 
	}
	for (int i = 1; i <= n; ++i)
	{
		for (int j = t; j >= costs[i]; --j)  // dependency loop
		{
			dp[j] = max(dp[j], dp[j - costs[i]] + vals[i]);
		}
	}
	cout << dp[t];
	return 0;
}
```
The dependency loop above is from back to front. If analyzed in 2D, we can see that each row is based on the result at position j - costs[i] of the previous row, i.e., dp[i - 1][j - costs[i]]. Therefore, in the 1D array, we need to retain the data from the previous row without it being updated before it is used to modify the current row's data. Hence, we need to determine that the modification of the 1D array is completed by decreasing from the end.

## Summary
This is the optimization principle of converting a 2D array into a 1D array. I hope it helps you.
