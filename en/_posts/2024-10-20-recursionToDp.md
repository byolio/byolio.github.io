---
layout: post
title: From Recursion to DP
subtitle: 简单聊一聊dp实现的思考路径
date: 2024-10-20T00:00:00.000Z
author: Byolio
header-img: img/post_bg_recursionToDp.jpg
catalog: true
tags:
  - c++
  - algorithm
  - dp
lang: en
---

> 🌐 [中文](https://www.byolio.blog/2024-10-20-recursionToDp/) | [English](https://www.byolio.blog/en/en/_posts/2024-10-20-recursionToDp/)

> Let's step into the world of DP

Test link for this chapter: \
[https://leetcode.cn/problems/minimum-path-sum/description/](https://leetcode.cn/problems/minimum-path-sum/description/)  

## What is DP
`DP` stands for `Dynamic Programming`. Its core idea is to decompose the original problem into a series of overlapping subproblems, solve these subproblems step by step, and store their results to avoid repeated calculations, thereby improving algorithm efficiency. It is essentially an algorithmic concept of `trading space for time`. 

## The Correct Path to Implementing DP
Many people, when they first encounter `DP`, think of concepts like `overlapping subproblems`, `optimal substructure`, and `no aftereffect`. Although these are important, they are not very helpful for beginners. \
The true path to learning `DP` implementation should start from the most familiar `recursion` and move towards `DP`. \
Below, I will demonstrate the complete implementation approach from `recursion` to `DP`, using a `2D DP` problem from `leetcode` as an example:

*1. Initial Recursive Implementation* \
This is a very simple recursive implementation. Here is the code directly:
```c++
#include<climits>
#include<algorithm>
// Recursive processing
class Solution {
public:
    int n, m;
    // Backtrack from the end point
    int f(vector<vector<int>>& grid, int i, int j)
    {
        if (i == 0 && j == 0) return grid[0][0];
        int up = INT_MAX;
        int left = INT_MAX;
        if (i - 1 >= 0)
        {
            up = f(grid, i - 1, j); // Move up one step
        }
        if (j - 1 >= 0)
        {
            left = f(grid, i, j - 1);   // Move left one step
        }
        return grid[i][j] + min(up, left);
    }
    int minPathSum(vector<vector<int>>& grid) {
        n = grid.size();
        m = grid[0].size();
        return f(grid, n - 1, m - 1);
    }
};
```
![recursion](https://cdn.jsdelivr.net/gh/byolio/mytc@main/img/recursion.png)

You can see the design logic is correct, but the execution speed is very slow. Therefore, we need to optimize it with `DP`.\
\
*2. `Memoization Search` DP Implementation* \
I decided to use a `2D dp array` for dynamic programming during `recursion`, caching the results for each cell, and using `memoization search` to avoid repeated calculations.
```c++
#include<algorithm>
#include<climits>
class Solution {
public:
    // Memoization search
    vector<vector<int>> dp;
    int n, m;
    int f(vector<vector<int>>& grid, int i, int j)
    {
        if(dp[i][j] != -1)
        {
            return dp[i][j];
        }
        int ans;
        if(i == 0 && j == 0)
        {
            ans = grid[0][0];
        }
        else
        {
            int up = INT_MAX, left = INT_MAX;   // Ensure that when one side is invalid, the other side is valid
            if(i - 1 >= 0) 
            { up = f(grid, i - 1, j); }
            if(j - 1 >= 0)  
            { left = f(grid, i, j - 1); }
            ans = grid[i][j] + min(up, left);
        }
        dp[i][j] = ans;  // First assignment to dp[0][0]
        return ans;
    }
    int minPathSum(vector<vector<int>>& grid) {
        n = grid.size();
        m = grid[0].size();
        dp = vector<vector<int>>(n, vector<int>(m, -1));
        return f(grid, n - 1, m - 1);
    }
};
```

![Memoization Search](https://cdn.jsdelivr.net/gh/byolio/mytc@main/img/post-dp-1.png)

You can see the execution speed has improved significantly, but each `recursive` call introduces function call overhead, including parameter passing and stack operations. Therefore, we need to continue optimizing.\
\
*3. `Strict Position Dependency` DP Implementation* \
Based on the above `DP` implementation idea, we can find that `dp[i][j]` only depends on `dp[i-1][j]` and `dp[i][j-1]`. Therefore, we can correlate these `dp` positions and implement `DP` using a `for` loop.
The code is as follows:
```c++
#include<algorithm>
#include<climits>
class Solution {
public:
    int n, m;
    vector<vector<int>> dp;
    int minPathSum(vector<vector<int>>& grid) {
        n = grid.size();
        m = grid[0].size();
        if( n == 1 && m == 1)
        {
            return grid[0][0];
        }
        dp = vector<vector<int>>(n, vector<int>(m, INT_MAX));  // 0 <= grid[i][j] <= 200
        dp[n - 1][m - 1] = grid[n - 1][m - 1];
        for(int i = n - 1; i >= 0; --i)
        {
            for(int j = m - 1; j >= 0; --j)
            {
                if(i - 1 >= 0) dp[i - 1][j] = min(dp[i - 1][j], dp[i][j] + grid[i - 1][j]);  // Up
                if(j - 1 >= 0) dp[i][j - 1] = min(dp[i][j - 1], dp[i][j] + grid[i][j - 1]);  // Left
            }
        }
        return dp[0][0];
    }
};
```

![Strict Position Dependency](https://cdn.jsdelivr.net/gh/byolio/mytc@main/img/post-dp-3.png) 

You can see that after using `strict position dependency`, the code becomes more concise and efficient, but its memory consumption is still too high. Therefore, we can proceed to the next optimization step.\
\
*4. Space-Optimized `DP` Implementation* \
In the above code, we notice that `dp` only uses `dp[i-1][j]` and `dp[i][j-1]` in the later stages. Therefore, we can replace the `2D dp array` with two `1D dp arrays` to achieve space optimization.
```c++
// Use two dp arrays to roll the 2D dp, achieving space compression
#include<algorithm>
#include<climits>
class Solution {
public:
    int n, m;
    vector<int> dp1;
    vector<int> dp2;
    int minPathSum(vector<vector<int>>& grid) {
        n = grid.size();
        m = grid[0].size();
        if (n == 1 && m == 1)
        {
            return grid[0][0];
        }
        dp1 = vector<int>(m, INT_MAX);  // 0 <= grid[i][j] <= 200
        dp2 = vector<int>(m, INT_MAX);
        dp1[0] = grid[0][0];
        for (int i = 1; i < m; ++i)   // Initialize dp1
        {
            dp1[i] = dp1[i - 1] + grid[0][i];
        }
        for (int i = 1; i < n; ++i)
        {
            dp2[0] = dp1[0] + grid[i][0];
            for (int j = 1; j < m; ++j)
            {
                dp2[j] = min(dp1[j], dp2[j - 1]) + grid[i][j];
            }
            dp1 = dp2;
        }
        return dp1[m - 1];
    }
};
```

![post-dp-4](https://cdn.jsdelivr.net/gh/byolio/mytc@main/img/post-dp-4.png)

You can see that the space-optimized code consumes less storage and also saves memory.

## Summary
From `recursion` to `DP`, I believe this is the most authentic implementation approach for `DP`. In fact, the implementation idea of `DP` is not difficult, but to truly master the implementation approach of `DP`, one still needs to think more, summarize more, and practice more.


