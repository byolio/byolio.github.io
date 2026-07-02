---
layout: post
title: Extensions of C++ References
subtitle: c++左值和右值引用的区别
date: 2024-11-30T00:00:00.000Z
author: Byolio
header-img: img/CppReference.jpg
catalog: true
tags:
  - c++
lang: en
---

> 🌐 [中文](/2024-11-30-CppReference/) | [English](/en/en/_posts/2024-11-30-CppReference/)

> What are lvalues and rvalues, and the difference between lvalue and rvalue references in C++

## Difference Between Lvalues and Rvalues
An lvalue is an expression that can appear on the left side of an assignment statement, while an rvalue is an expression that can only appear on the right side of an assignment statement.
```c++
a = 10; // 10 is an rvalue, a is an lvalue
vector<int> v;
v = vector<int>(10, 0); // vector<int>(10, 0) is an rvalue, v is an lvalue
```
Simply put, an lvalue can be assigned a value, while an rvalue cannot (i.e., the left side of = is an lvalue, and the right side of = is an rvalue).

## Difference Between Lvalue References and Rvalue References (Rvalue References Introduced in C++11)
Having understood the difference between lvalues and rvalues, we can now explore the difference between lvalue references and rvalue references. As the names suggest, an lvalue reference is a reference bound to an lvalue, and an rvalue reference is a reference bound to an rvalue. \
The syntax for lvalue references is `类型&`, and for rvalue references is `类型&&`. Examples are as follows:
```c++
int a = 10;      // Assignment
int &b = a;      // Lvalue reference
int &&c = 10;    // Rvalue reference

vector<int> v = vector<int>(10, 0);    // Assignment
vector<int> &v1 = v;                   // Lvalue reference
vector<int> &&v2 = vector<int>(10, 0); // Rvalue reference
```

## Difference Between const Lvalue References and const Rvalue References
The difference between const lvalue references and rvalue references is that const lvalue references can bind to rvalues, while const rvalue references cannot bind to lvalues.
```c++
int a = 10;
const int&& b = a;  // Error
const int& c = a;   // Lvalue
const int& d = 10;  // Rvalue
```

## Summary
1. The difference between lvalue references and rvalue references is that lvalue references can bind to lvalues, while rvalue references can bind to rvalues.
2. The difference between const lvalue references and rvalue references is that const lvalue references can bind to rvalues, while const rvalue references cannot bind to lvalues.

