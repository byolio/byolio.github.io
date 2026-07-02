---
layout: post
title: Algorithm -- 3 Algorithmic Logics for the Multiple Knapsack Problem
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

> 🌐 [中文](/2025-1-25-MultipleBag/) | [English](/en/en/_posts/2025-1-25-MultipleBag/)

> This article aims to optimize the algorithmic logic for the multiple knapsack problem.
> Test link for this chapter: [https://www.luogu.com.cn/problem/P1776](https://www.luogu.com.cn/problem/P1776)

## What is the Multiple Knapsack Problem
The multiple knapsack problem is a problem that lies between the unbounded knapsack and the 0/1 knapsack problem. It involves n types of items, but each type has a limited quantity. Due to this uniqueness, the algorithmic logic for solving the multiple knapsack problem is closely related to the solution methods of these two knapsack problems.

## Basic Algorithmic Logic for the Multiple Knapsack Problem
When we see the multiple knapsack problem, it's easy to think of the unbounded knapsack and transform it into a variant of the unbounded knapsack. [Test link](https://www.luogu.com.cn/problem/P1776)
(Here, we use the directly space-optimized multiple knapsack code. For the optimization logic, please refer to my previous [article](https://byolio.top/2025/01/24/oneToTwoOptimize/)):
```c++
vector<int> costs; // weight cost for each treasure
vector<int> vals;  // value of each treasure
vector<int> nums;  // total quantity of each treasure
vector<int> dp;    // current knapsack capacity -> current value
int n, m;		   // total number of treasures, total knapsack capacity

// Strict position-dependent space optimization
int compute()
{
	dp = vector<int>(m + 1, 0);
	// Multiple knapsack
	for (int i = 1; i <= n; ++i)
	{
		for (int j = m; j >= 0; --j)   // Prevent overwriting previous values
		{
			// Purchase this treasure
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
We can see the result is TLE (Time Limit Exceeded). This is because the time complexity of the multiple knapsack problem is O(N * M * K), where N is the number of item types, M is the knapsack capacity, and K is the quantity of items. This leads to high time complexity and failure to pass the test. Therefore, the algorithmic logic needs to be optimized.

## Binary Grouping Algorithmic Logic for the Multiple Knapsack Problem
### Algorithmic Logic Example
As I mentioned earlier, the multiple knapsack problem lies between the unbounded knapsack and the 0/1 knapsack problem. When the unbounded knapsack solution has high time complexity, we can transform the multiple knapsack problem into a 0/1 knapsack problem and then solve it using the algorithmic logic of the 0/1 knapsack problem. Let's look at the code first. The test link is the same as above (if this problem is not space-optimized, it will cause MLE (Memory Limit Exceeded), so a space-optimized version is needed):
```c++
vector<int> costs;	// Total weight cost for each group
vector<int> vals;	// Total value of treasures in each group
vector<int> dp;     // Group index idx + current knapsack capacity -> current value
int n, m;			// Total number of groups (invalid in 0/1 knapsack), total knapsack capacity
int len;            // Total number of items in the 0/1 knapsack
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
	// Add a redundant element
	costs = vector<int>(1);
	vals = vector<int>(1);
	// Rewrite data into grouped knapsack
	for (int i = 1, v, w, c; i <= n; ++i)   // Groups 1 to n
	{
		cin >> v >> w >> c;
		for (int j = 1; j <= c; j <<= 1)   // Start from selection, skip the entire group if not selected
		{
			costs.push_back(w * j);  // Cost of this group
			vals.push_back(v * j);	// Value of this group
			c -= j;
		}
		if (c > 0)			// Ensure no value exceeds the total, convert grouped knapsack to 0/1 knapsack
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
### Analysis of Binary Grouping Algorithmic Logic
In the multiple knapsack problem, each type of item can be selected multiple times, but cannot exceed the maximum quantity in a group. Therefore, the key to converting it into a 0/1 knapsack problem is the "grouping" strategy. Grouping is done by continuously multiplying by 2 until it is no longer possible to continue grouping, then the remainder is treated as a separate group. In this way, even if all items are selected, the total is still less than the maximum quantity, but any quantity can be achieved through different combinations. Finally, the grouped problem can be solved using the algorithmic logic of the 0/1 knapsack problem. This reduces the time complexity to O(N * M * logK), where N is the number of item types, M is the knapsack capacity, and K is the quantity of items. This is a significant optimization compared to the previous O(N * M * K) time complexity.

## Monotonic Queue Algorithmic Logic for the Multiple Knapsack Problem
Using binary grouping still has a problem: if the number of items is too large, the number of groups after grouping will also be large, leading to high space complexity. Therefore, we need to optimize the algorithmic logic. The monotonic queue algorithmic logic is a good optimization method, which can optimize the time complexity to O(N * M), where N is the number of item types and M is the knapsack capacity. \
The test link is the same as above (using the space-optimized version). The code is as follows:
```c++
vector<int> costs;			// Cost
vector<int> nums;			// Quantity
vector<int> vals;			// Value
vector<int> dp;				// 1D dp

int l, r;   // Monotonic queue pointers
int n, t;	// Number of item types and knapsack capacity

int value1(int i, int j) {
	return dp[j] - j / costs[i] * vals[i];
}

// Space optimization
int compute()
{
	dp = vector<int>(t + 1, 0);		// Dynamic programming array
	deque<int> q;					// Use deque to implement monotonic queue
	for (int i = 1; i <= n; ++i)
	{
		for (int mod = 0; mod <= min(t, costs[i] - 1); ++mod)		// Minimum quantity or cost-1, group by adding costs[i] for different mods
		{
			q.clear();
			for (int j = t - mod, cnt = 1; j >= 0 && cnt <= nums[i]; j -= costs[i], ++cnt)  // Get the value at the correct position
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
		cin >> vals[i] >> costs[i] >> nums[i]; // Input value, weight, and quantity
	}
	cout << compute();
	return 0;
}
```
This solution requires using congruence class grouping because adding an integer multiple of the cost under the same remainder allows the use of the value function to make the pattern clearer. Under the same upper limit j, a monotonic queue is used for optimization.


## Summary
The above are the three algorithmic logics for the multiple knapsack problem. I hope they are helpful to you.



