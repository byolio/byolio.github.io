---
layout:     post
title:      "algorihtm--fastPower"
subtitle:   "如何实现乘法快速幂和矩阵快速幂"
date:       2025-10-31
author:     "Byolio"
header-img: "img/fastPower.jpg"
catalog: true
tags:
    - c++
    - algorithm
---
> 本文旨在讲解如何实现乘法快速幂和矩阵快速幂
> 本章测试连接为 : 
> 乘法快速幂: [https://www.luogu.com.cn/problem/P1226](https://www.luogu.com.cn/problem/P1226)
> 1维k阶矩阵快速幂: [https://leetcode.cn/problems/n-th-tribonacci-number/](https://leetcode.cn/problems/n-th-tribonacci-number/)
> k维1阶矩阵快速幂: [https://leetcode.cn/problems/count-vowels-permutation/](https://leetcode.cn/problems/count-vowels-permutation/)

## 快速幂简介
快速幂是一种用于快速计算幂的算法，它利用了指数的二进制表示来减少乘法操作的次数。主要分为两种类型:
1. 乘法快速幂: 用于计算`a^b`等形式的幂运算。
2. 矩阵快速幂: 用于计算矩阵的幂运算, 矩阵快速幂又具体分为1维k阶矩阵快速幂和k维1阶矩阵快速幂。


## 快速幂的时间复杂度
快速幂的时间复杂度为O(log n)，其中n为指数。传统的幂运算需要进行n次乘法操作，而快速幂通过将指数不断除以2，显著减少了乘法操作的次数，从而提高了计算效率, 因此快速幂都存在将数不断平方相加的操作。

## 乘法快速幂
我们先来看乘法快速幂: [https://www.luogu.com.cn/problem/P1226](https://www.luogu.com.cn/problem/P1226) \
解法如下: 
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
1. 在乘法快速幂中, 我们通过将指数b不断除以2, 并根据指数的二进制位是否为1来判断是否需要将当前的底数a乘到结果ans中。 \
例如, 计算`2^10`时, 指数10的二进制表示为1010, 我们从右往左看, 当遇到第一个1时, 我们将当前的底数2乘到结果ans中, 然后将底数平方, 指数右移一位, 继续判断下一个二进制位。
2. 当遇到下一个1时, 我们将当前的底数2乘到结果ans中, 然后将底数平方, 指数右移一位, 继续判断下一个二进制位。\
我们可以发现, 每次将指数右移一位, 底数就会平方一次, 因此我们只需要进行log n次乘法操作, 就可以得到结果。

## 矩阵快速幂
矩阵快速幂可以看作是乘法快速幂的推广, 乘法快速幂用于1维1阶的计算, 而矩阵快速幂则用于k维1阶的计算和1维k阶的计算。
### 1维k阶矩阵快速幂例子例子总览
这里使用泰波那契数列作为例子, 题目链接为: [https://leetcode.cn/problems/n-th-tribonacci-number/](https://leetcode.cn/problems/n-th-tribonacci-number/)
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
其思路和乘法快速幂类似, 只是将底数从1维1阶推广到了1维k阶。乘法快速幂中的乘法在1维k阶矩阵快速幂变成了矩阵乘法函数multiply, 用于计算两个矩阵的乘积。然后定义了一个矩阵快速幂函数power, 用于进行矩阵的幂运算。该函数的实现也与乘法快速幂类似, 只不过更换了底数为矩阵, 我们通过将指数不断除以2, 并根据指数的二进制位是否为1来判断是否需要将当前的底数矩阵a乘到结果ans中。底数矩阵a则不断对自己进行平方, 和乘法快速幂中底数处理方式相同。
### 1维k阶矩阵快速幂底层思路解析
#### 如何得知其是1维k阶矩阵快速幂
其形式需满足 F(n + k) = F(n + k - 1) + F(n + k - 2) + ... + F(n + k - k) \
其中k就是k阶的含义, 满足该式子就是满足可以使用1维k阶矩阵快速幂的条件, 我之前写过的[斐波那契数列blog](https://byolio.top/2024/10/06/fib/)中就提到过其就是1维k阶矩阵快速幂, 这里就不重复了。
#### 如何构造矩阵start和base
1. start矩阵中的值为F(n + k - 1), F(n + k - 2), ... , F(n + k - k)中最左边覆盖参数范围的值 \
例子:  \
T0 = 0, T1 = 1, T2 = 1  \
Tn+3 = Tn + Tn+1 + Tn+2 \
则start矩阵为 {1, 1, 0} , 覆盖了T2, T1, T0, 从右面的往左赋值, 因为最左边的是最后要输出的答案, 最右边是n输入范围内的第一个值 \
但需要注意 : \
如果为 Tn+4 = Tn + Tn+3 \
T0 = 0, T1 = 1, T2 = 1, T3 = 2  \
start矩阵为 {2, 1, 1, 0} , 需要覆盖T4, T3, T2, T1, 整个等式右边范围
2. base矩阵的构造也是根据等式右边的关系来构造的 \
矩阵第一列是等式右边的系数, 例如Tn+3 = Tn + Tn+1 + Tn+2, 第一列是{1, 1, 1}
如果为 Tn+4 = Tn + Tn+3  \
可以视为 : Tn+4 = 1 * Tn + 0 * Tn+1 + 0 * Tn+2 + 1 * Tn+3, 则base矩阵第一列为 {1, 0, 0, 1}  \
其他列则是设未知数计算的:  \
例如: Tn+3 = Tn + Tn+1 + Tn+2, 设base矩阵为:
{1, a, d}
{1, b, e}
{1, c, f} \
然后通过向后计算赋值求出a, b, c, d, e, f的值, 比方说泰波那契数列的例子, 我们可以得到:
{1, 1, 0}
{1, 0, 1}
{1, 0, 0}
#### 最后的结果如何获取
以泰波那契数列为例, start数组乘以base经过矩阵快速幂处理后, 最左边的值就是需要的结果, 其中为啥是n - 2不是n - 1, n, 因为start数组有3为, 但有用的只有最左边这1位, 3 - 1 = 2, 所以是n - 2, 也因此n == 0和n == 1时会被start求值时漏掉, 直接做预处理返回0和1即可。

### k维1阶矩阵快速幂例子总览
这里使用[https://leetcode.cn/problems/count-vowels-permutation/](https://leetcode.cn/problems/count-vowels-permutation/) 作为例子: 
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
其运算过程和1维k阶矩阵快速幂类似, 但是其底层思路和1维k阶矩阵快速幂不同, 下面就是其底层思路解析
### k维1阶矩阵快速幂底层思路解析
#### 如何得知其是k维1阶矩阵快速幂
其形式需满足每个元素都满足`F(n) = F(n - 1)`这种一阶公式, 但其同时存在多个一阶公式, 每个一阶公式对应一个维度, 最后将所有维度的结果相加即可。 
### 如何构造矩阵start和base
1. start矩阵的构造 \
   当总元素数为1时, 其可以为对应元素, 就将对应元素设置为1, 该题中每个元素在总元素为1的情况下, 可以是该元素就设置为1, 因此start = {1, 1, 1, 1, 1}
2. base矩阵的构造 \
   横坐标可以为每个元素, 纵坐标则是对应元素的前一个元素, base可以看作: \
```test
n对应的值:
            a  0  1  0  0  0
            e  1  0  1  0  0
            i  1  1  0  1  1
            o  0  0  1  0  1
            u  1  0  0  0  0
n-1对应的值:    a  e  i  o  u
```
a前可以为e, i, u就将a和e, i, u交界处设置为1, 其他同理
#### 最后的结果如何获取
经过矩阵快速幂处理后, start矩阵乘以base矩阵的n - 1次方(去掉已知的第一次), 然后将结果矩阵中的每个维度计算后值相加即可得到最终结果。

## FAQ
### 如何得到1维k阶矩阵的表达式
F(n) = F(n - 1) + F(n - 2) + ... + F(n - k)表达式作为1维k阶的解题核心是需要题目给出的, 但有的题目又不会给出, 可以使用暴力方法打表找规律解决
