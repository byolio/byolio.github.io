---
layout: post
title: algorihtm--fastPower
subtitle: 如何实现乘法快速幂和矩阵快速幂
date: 2025-10-31T00:00:00.000Z
author: Byolio
header-img: img/fastPower.jpg
catalog: true
tags:
  - c++
  - algorithm
lang: en
---

> 🌐 [中文](https://www.byolio.blog/2025-10-31-fastPower/) | [English](https://www.byolio.blog/en/en/_posts/2025-10-31-fastPower/)

> This article aims to explain how to implement fast exponentiation for multiplication and fast exponentiation for matrices.
> Test links for this chapter:
> Fast exponentiation for multiplication: [https://www.luogu.com.cn/problem/P1226](https://www.luogu.com.cn/problem/P1226)
> 1-dimensional k-order matrix fast exponentiation: [https://leetcode.cn/problems/n-th-tribonacci-number/](https://leetcode.cn/problems/n-th-tribonacci-number/)
> k-dimensional 1-order matrix fast exponentiation: [https://leetcode.cn/problems/count-vowels-permutation/](https://leetcode.cn/problems/count-vowels-permutation/)

## Introduction to Fast Exponentiation
Fast exponentiation is an algorithm for quickly computing powers, which uses the binary representation of the exponent to reduce the number of multiplication operations. It is mainly divided into two types:
1. Fast exponentiation for multiplication: used to compute powers like `a^b`.
2. Fast exponentiation for matrices: used to compute powers of matrices, which is further divided into 1-dimensional k-order matrix fast exponentiation and k-dimensional 1-order matrix fast exponentiation.


## Time Complexity of Fast Exponentiation
The time complexity of fast exponentiation is O(log n), where n is the exponent. Traditional exponentiation requires n multiplication operations, while fast exponentiation significantly reduces the number of multiplication operations by repeatedly dividing the exponent by 2, thereby improving computational efficiency. Therefore, fast exponentiation always involves repeatedly squaring and summing numbers.

## Fast Exponentiation for Multiplication
Let's first look at fast exponentiation for multiplication: [https://www.luogu.com.cn/problem/P1226](https://www.luogu.com.cn/problem/P1226) \
The solution is as follows:
```c++
#include<iostream>
#include<vector>
using namespace std;
int power(int a, int b, int p) {
	long long ans = 1;
	while (b > 0) {
		if ((b & 1) == 1) {
			ans = (ans * a) % p;
		}
		a = ((long long)a * a) % p;  // 自己乘以自己会1 2 4 8 16
		b >>= 1;
	}
	return ans;
}

int main() {
	int a, b, p;
	cin >> a >> b >> p;
	cout << a << "^" << b << " mod " << p << "=" << power(a, b, p);
	return 0;
}
```
1. In fast exponentiation for multiplication, we repeatedly divide the exponent b by 2 and check whether the current binary bit of the exponent is 1 to decide whether to multiply the current base a into the result ans. \
For example, when computing `2^10`, the binary representation of exponent 10 is 1010. We look from right to left. When we encounter the first 1, we multiply the current base 2 into the result ans, then square the base, shift the exponent right by one bit, and continue to check the next binary bit.
2. When we encounter the next 1, we multiply the current base 2 into the result ans, then square the base, shift the exponent right by one bit, and continue to check the next binary bit.\
We can see that each time we shift the exponent right by one bit, the base is squared once. Therefore, we only need log n multiplication operations to obtain the result.

## Fast Exponentiation for Matrices
Matrix fast exponentiation can be seen as an extension of fast exponentiation for multiplication. Fast exponentiation for multiplication is used for 1-dimensional 1-order calculations, while matrix fast exponentiation is used for k-dimensional 1-order and 1-dimensional k-order calculations.
### Overview of 1-dimensional k-order Matrix Fast Exponentiation Example
Here we use the Tribonacci sequence as an example. The problem link is: [https://leetcode.cn/problems/n-th-tribonacci-number/](https://leetcode.cn/problems/n-th-tribonacci-number/)
```c++
class Solution {
public:
	vector<vector<int>> multiply(vector<vector<int>> a, vector<vector<int>> b) {
		int n = a.size();
		int m = b[0].size();
		int k = a[0].size();
		vector<vector<int>> ans(n, vector<int>(m));
		for (int i = 0; i < n; ++i) {
			for (int j = 0; j < m; ++j) {
				for (int c = 0; c < k; ++c) {
					ans[i][j] += (long long)a[i][c] * b[c][j];
				}
			}
		}
		return ans;
	}

	// 矩阵快速幂
	vector<vector<int>> power(vector<vector<int>> a, int p) {
		int n = a.size();
		vector<vector<int>> ans(n, vector<int>(n));
		// 制造单位矩阵
		for (int i = 0; i < n; ++i) {
			ans[i][i] = 1;
		}
		while (p > 0) {
			if ((p & 1) == 1) {
				ans = multiply(ans, a);
			}
			a = multiply(a, a);
			p >>= 1;
		}
		return ans;
	}
	int tribonacci(int n) {
		// 先处理start最左边无法覆盖的情况
		if (n == 0) return 0;
		if (n == 1) return 1;
		vector<vector<int>> start = { {1, 1, 0} };
		vector<vector<int>> base = {
			{1, 1, 0},
			{1, 0, 1},
			{1, 0, 0}
		};
		vector<vector<int>> ans = multiply(start, power(base, n - 2));
		return ans[0][0];
	}
};
```
Its idea is similar to fast exponentiation for multiplication, except that the base is extended from 1-dimensional 1-order to 1-dimensional k-order. The multiplication in fast exponentiation for multiplication becomes the matrix multiplication function multiply in 1-dimensional k-order matrix fast exponentiation, used to compute the product of two matrices. Then a matrix fast exponentiation function power is defined to perform matrix exponentiation. The implementation of this function is also similar to fast exponentiation for multiplication, except that the base is replaced by a matrix. We repeatedly divide the exponent by 2 and check whether the current binary bit of the exponent is 1 to decide whether to multiply the current base matrix a into the result ans. The base matrix a is repeatedly squared, just like the base handling in fast exponentiation for multiplication.
### Underlying Idea Analysis of 1-dimensional k-order Matrix Fast Exponentiation
#### How to Know It Is 1-dimensional k-order Matrix Fast Exponentiation
Its form must satisfy F(n + k) = F(n + k - 1) + F(n + k - 2) + ... + F(n + k - k) \
where k is the meaning of k-order. Satisfying this equation is the condition for using 1-dimensional k-order matrix fast exponentiation. I mentioned in my previous [Fibonacci sequence blog](https://byolio.top/2024/10/06/fib/) that it is exactly 1-dimensional k-order matrix fast exponentiation, so I won't repeat it here.
#### How to Construct the start and base Matrices
1. The values in the start matrix are the values of F(n + k - 1), F(n + k - 2), ..., F(n + k - k) that cover the parameter range from the leftmost side. \
Example:  \
T0 = 0, T1 = 1, T2 = 1  \
Tn+3 = Tn + Tn+1 + Tn+2 \
Then the start matrix is {1, 1, 0}, covering T2, T1, T0, assigned from right to left, because the leftmost value is the final answer to output, and the rightmost is the first value within the input range of n. \
But note: \
If it is Tn+4 = Tn + Tn+3 \
T0 = 0, T1 = 1, T2 = 1, T3 = 2  \
The start matrix is {2, 1, 1, 0}, needing to cover T4, T3, T2, T1, the entire range on the right side of the equation.
2. The construction of the base matrix is also based on the relationship on the right side of the equation. \
The first column of the matrix is the coefficients on the right side of the equation. For example, Tn+3 = Tn + Tn+1 + Tn+2, the first column is {1, 1, 1}. \
If it is Tn+4 = Tn + Tn+3  \
It can be seen as: Tn+4 = 1 * Tn + 0 * Tn+1 + 0 * Tn+2 + 1 * Tn+3, then the first column of the base matrix is {1, 0, 0, 1}. \
The other columns are computed by setting unknowns: \
For example: Tn+3 = Tn + Tn+1 + Tn+2, let the base matrix be:
{1, a, d}
{1, b, e}
{1, c, f} \
Then by backward calculation, we find the values of a, b, c, d, e, f. For the Tribonacci sequence example, we get:
{1, 1, 0}
{1, 0, 1}
{1, 0, 0}
#### How to Obtain the Final Result
Taking the Tribonacci sequence as an example, after multiplying the start matrix by the base matrix processed by matrix fast exponentiation, the leftmost value is the required result. Why is it n - 2 instead of n - 1 or n? Because the start array has 3 elements, but only the leftmost one is useful. 3 - 1 = 2, so it is n - 2. Therefore, when n == 0 and n == 1, they are missed by the start evaluation, so we directly preprocess and return 0 and 1.

### Overview of k-dimensional 1-order Matrix Fast Exponentiation Example
Here we use [https://leetcode.cn/problems/count-vowels-permutation/](https://leetcode.cn/problems/count-vowels-permutation/) as an example:
```c++
class Solution {
public:
	const int MOD = 1e9 + 7;

	vector<vector<int>> multiply(vector<vector<int>> a, vector<vector<int>> b) {
		int n = a.size();
		int m = b[0].size();
		int k = a[0].size();
		vector<vector<int>> ans(n, vector<int>(m, 0));
		for (int i = 0; i < n; ++i) {
			for (int j = 0; j < m; ++j) {
				for (int c = 0; c < k; ++c) {
					ans[i][j] = ((long long)a[i][c] * b[c][j] + ans[i][j]) % MOD;
				}
			}
		}
		return ans;
	}

	vector<vector<int>> power(vector<vector<int>> a, int p) {
		int n = a.size();
		vector<vector<int>> ans(n, vector<int>(n, 0));
		for (int i = 0; i < n; ++i) {
			ans[i][i] = 1;
		}
		while (p > 0) {
			if ((p & 1) == 1) {
				ans = multiply(ans, a);
			}
			p >>= 1;
			a = multiply(a, a);
		}
		return ans;
	}
	int countVowelPermutation(int n) {
		vector<vector<int>> start = { {1, 1, 1, 1, 1} };  // 每个字母在最后都可以被算为一种结果
		vector<vector<int>> base = {
			{0, 1, 0, 0, 0},
			{1, 0, 1, 0, 0},
			{1, 1, 0, 1, 1},
			{0, 0, 1, 0, 1},
			{1, 0, 0, 0, 0}
		};
		vector<vector<int>> end = multiply(start, power(base, n - 1));
		int ans = 0;
		for (int e : end[0]) {
			ans = (e + ans) % MOD;
		}
		return ans;
	}
};
```
Its computation process is similar to 1-dimensional k-order matrix fast exponentiation, but its underlying idea is different. Below is the analysis of its underlying idea.
### Underlying Idea Analysis of k-dimensional 1-order Matrix Fast Exponentiation
#### How to Know It Is k-dimensional 1-order Matrix Fast Exponentiation
Its form must satisfy that each element satisfies a first-order formula like `F(n) = F(n - 1)`, but there are multiple first-order formulas simultaneously, each corresponding to a dimension. Finally, the results of all dimensions are summed.
### How to Construct the start and base Matrices
1. Construction of the start matrix \
   When the total number of elements is 1, it can be the corresponding element, so set the corresponding element to 1. In this problem, each element can be itself when the total number is 1, so start = {1, 1, 1, 1, 1}.
2. Construction of the base matrix \
   The horizontal axis can be each element, and the vertical axis is the previous element of the corresponding element. The base can be seen as: \
```test
n对应的值:
            a  0  1  0  0  0
            e  1  0  1  0  0
            i  1  1  0  1  1
            o  0  0  1  0  1
            u  1  0  0  0  0
n-1对应的值:    a  e  i  o  u
```
For a, the previous elements can be e, i, u, so set the intersection of a with e, i, u to 1. Similarly for others.
#### How to Obtain the Final Result
After matrix fast exponentiation, multiply the start matrix by the base matrix raised to the power of n - 1 (removing the known first step), then sum the values of each dimension in the resulting matrix to get the final result.

## FAQ
### How to Obtain the Expression for 1-dimensional k-order Matrix
The expression F(n) = F(n - 1) + F(n - 2) + ... + F(n - k) is the core of solving 1-dimensional k-order problems and needs to be given by the problem. However, some problems do not provide it, and you can use brute force methods to tabulate and find patterns.
