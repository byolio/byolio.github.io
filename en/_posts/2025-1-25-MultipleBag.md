---
layout: post
title: Algorithm -- Three Algorithmic Logics for the Multiple Knapsack Problem
subtitle: 优化多重背包算法逻辑
date: 2025-1-26
author: Byolio
header-img: img/MultipleBag.jpg
catalog: true
tags:
  - c++
  - algorithm
lang: en
---

> 🌐 [中文](https://www.byolio.blog/2025-1-25-MultipleBag/) | [English](https://www.byolio.blog/en/en/_posts/2025-1-25-MultipleBag/)

> This article aims to optimize the algorithmic logic of the multiple knapsack problem.
> Test link for this chapter: [https://www.luogu.com.cn/problem/P1776](https://www.luogu.com.cn/problem/P1776)

## What is the Multiple Knapsack Problem
The multiple knapsack problem is a problem that lies between the unbounded knapsack problem and the 0/1 knapsack problem. That is, there are n types of items, but each type has a limited number. Due to this uniqueness, the algorithmic logic for solving the multiple knapsack problem is closely related to the solution methods of these two knapsack problems.

## Basic Algorithmic Logic for the Multiple Knapsack Problem
When we see the multiple knapsack problem, we can easily think of the unbounded knapsack problem and transform it into a variant of the unbounded knapsack problem. [Test link](https://www.luogu.com.cn/problem/P1776)
(Here we use the multiple knapsack code with direct space optimization. For the optimization logic, please refer to my previous [article](https://byolio.top/2025/01/24/oneToTwoOptimize/)):
```c++
vector<int> costs; // weight cost for each treasure
vector<int> vals;  // value of each treasure
vector<int> nums;  // total quantity of each treasure
vector<int> dp;    // current knapsack capacity -> current value
int n, m;		   // total number of treasures, total knapsack capacity

// Space optimization with strict position dependency
int compute()
{
	dp = vector<int>(m + 1, 0);
	// Multiple knapsack
	for (int i = 1; i <= n; ++i)
	{
		for (int j = m; j >= 0; --j)   // prevent overwriting previous values
		{
			// Purchase the treasure
			for (int t = 1; t <= nums[i] && j - t * costs[i] >= 0; ++t)
			{
				dp[j] = max(dp[j], dp[j - t * costs[i]] + t * vals[i]); 
			}
		}
	}
	return dp[m];
}
int main()
{
	cin >> n >> m;
	costs = vector<int>(n + 1);
	vals = vector<int>(n + 1);
	nums = vector<int>(n + 1);
	for (int i = 1; i <= n; ++i)
	{
		cin >> vals[i] >> costs[i] >> nums[i];
	}
	cout << compute();
	return 0;
}
```
The submission result is as follows:
![MultipleBagSimple](https://cdn.jsdelivr.net/gh/byolio/tc3@main/img/MultipleBag1.png)
We can see the result is TLE (Time Limit Exceeded). This is because the time complexity of the multiple knapsack problem is O(N * M * K), where N is the number of item types, M is the knapsack capacity, and K is the quantity of each item. This leads to high time complexity and failure to pass the test. Therefore, the algorithmic logic needs to be optimized.

## Binary Grouping Algorithmic Logic for the Multiple Knapsack Problem
### Algorithmic Logic Example
As I mentioned earlier, the multiple knapsack problem lies between the unbounded knapsack problem and the 0/1 knapsack problem. When the time complexity of the unbounded knapsack solution is too high, we can transform the multiple knapsack problem into a 0/1 knapsack problem and then solve it using the algorithmic logic of the 0/1 knapsack problem. Let's look at the code first. The test link is the same as above (if this problem is not space-optimized, it will cause MLE (Memory Limit Exceeded), so a space-optimized version is used):
```c++
vector<int> costs;	// total weight cost of each group
vector<int> vals;	// total value of each group
vector<int> dp;     // group index + current knapsack capacity -> current value
int n, m;			// total number of groups (invalid in 0/1 knapsack), total knapsack capacity
int len;            // total number of items in 0/1 knapsack
int compute1()
{
	dp = vector<int>(m + 1, 0);
	// 0/1 knapsack
	for (int i = 1; i <= len; ++i)
	{
		for (int j = m; j >= costs[i]; --j)
		{
			dp[j] = max(dp[j], dp[j - costs[i]] + vals[i]);
		}
	}
	return dp[m];
}
int main()
{
	cin >> n >> m;
	// Add a dummy element
	costs = vector<int>(1);
	vals = vector<int>(1);
	// Rewrite data into grouped knapsack
	for (int i = 1, v, w, c; i <= n; ++i)   // groups 1 to n
	{
		cin >> v >> w >> c;
		for (int j = 1; j <= c; j <<= 1)   // start from 1, skip the entire group if not selected
		{
			costs.push_back(w * j);  // cost of this group
			vals.push_back(v * j);	// value of this group
			c -= j;
		}
		if (c > 0)			// ensure no value exceeds the total, convert grouped knapsack to 0/1 knapsack
		{
			costs.push_back(w * c);
			vals.push_back(v * c);
		}
	}
	// After grouping, it can be treated as a 0/1 knapsack
	len = vals.size();
	cout << compute1();
	return 0;
}
```
The result is as follows:
![MultipleBagBinary](https://cdn.jsdelivr.net/gh/byolio/tc3@main/img/MultipleBag2.png)
All tests passed!!!
### Explanation of Binary Grouping Algorithmic Logic
In the multiple knapsack problem, each type of item can be selected multiple times, but cannot exceed the maximum quantity in a group. Therefore, the key to converting it into a 0/1 knapsack problem is the "grouping" strategy. We continuously group by multiplying by 2 until we can no longer group, then treat the remainder as a separate group. In this way, even if all items are selected, the total is still less than the maximum quantity, but any quantity can be achieved through different combinations. Finally, we solve the grouped problem using the algorithmic logic of the 0/1 knapsack problem. This reduces the time complexity to O(N * M * logK), where N is the number of item types, M is the knapsack capacity, and K is the quantity of each item. Compared to the previous O(N * M * K), this is a significant optimization.

## Monotonic Queue Algorithmic Logic for the Multiple Knapsack Problem
Using binary grouping still has a problem: if the number of items is too large, grouping will result in too many groups, leading to high space complexity. Therefore, we need to optimize the algorithmic logic. The monotonic queue algorithmic logic is a good optimization method, which can reduce the time complexity to O(N * M), where N is the number of item types and M is the knapsack capacity. \
The test link is the same as above (using the space-optimized version). The code is as follows:
```c++
vector<int> costs;			// cost
vector<int> nums;			// quantity
vector<int> vals;			// value
vector<int> dp;				// 1D dp

int l, r;   // monotonic queue pointers
int n, t;	// number of item types and knapsack capacity

int value1(int i, int j) {
	return dp[j] - j / costs[i] * vals[i];
}

// Space optimization
int compute()
{
	dp = vector<int>(t + 1, 0);		// dynamic programming array
	deque<int> q;					// use deque to implement monotonic queue
	for (int i = 1; i <= n; ++i)
	{
		for (int mod = 0; mod <= min(t, costs[i] - 1); ++mod)		// minimum quantity or cost-1, group by adding costs[i] under different mod
		{
			q.clear();
			for (int j = t - mod, cnt = 1; j >= 0 && cnt <= nums[i]; j -= costs[i], ++cnt)  // get the value at the correct position
			{
				while (!q.empty() && value1(i, q.back()) <= value1(i, j))
				{
					q.pop_back();
				}
				q.push_back(j);
			}
			for (int j = t - mod, enter = j - costs[i] * nums[i]; j >= 0; j -= costs[i], enter -= costs[i])
			{
				if (enter >= 0)
				{
					while (!q.empty() && value1(i, q.back()) <= value1(i, enter))
					{
						q.pop_back();
					}
					q.push_back(enter);
				}
				dp[j] = value1(i, q.front()) + j / costs[i] * vals[i];
				if (q.front() == j)
				{
					q.pop_front();
				}
			}
		}
	}
	return dp[t];
}


int main()
{
	cin >> n >> t;
	vals = vector<int>(n + 1);
	nums = vector<int>(n + 1);
	costs = vector<int>(n + 1);
	for (int i = 1; i <= n; i++) {
		cin >> vals[i] >> costs[i] >> nums[i]; // input value, weight, and quantity
	}
	cout << compute();
	return 0;
}
```
This solution requires congruence class grouping because adding an integer multiple of the cost under the same remainder makes the pattern clearer. Using the value function makes the pattern more apparent, and under the same upper limit j, the monotonic queue is used for optimization.

## Summary
The above are the three algorithmic logics for the multiple knapsack problem. I hope they are helpful to you.



