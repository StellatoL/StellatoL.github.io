---
title: "泛型指针与通用性"
published: 2025-01-01
description: "int a = 10;"
tags:
  - "指针"
  - "C"
category: "C"
categoryPath:
  - "RM_PRIME"
  - "RM算法"
  - "学习记录"
  - "C"
draft: false
---

# 1.泛型指针的定义
## 类型声明：
### void * 是一个*无类型指针*，可以指向任意类型的数据的地址
### eg.
```c
int a = 10;
char c = 'x';
void *p = &a;  // 正确：指向 int
p = &c;        // 正确：指向 char
```

---
<!--more-->

# 2.核心特点

## (1)通用型
### 可以接受任何数据类型的地址
```c
void *p;
int num = 5;
p = &num;     // 指向 int
p = "hello";   // 指向字符串
```

---
## (2)不能直接解引用
### void* 指针没有类型信息，直接引用可能会导致编译错误
```c
int x = 10;
void *p = &x;
// *p = 20;    // 错误：无法解引用 void*
int *q = (int*)p; // 必须转为具体类型的指针
*q = 20;          // 正确
```

---
## (3)指针运算限制
### void* 指针不能直接进行算术运算（如 `p++`），因为编译器不知道步长：
  ```c
  int arr[3] = {1, 2, 3};
  void *p = arr;
  // p++;        // 错误：未知步长
  ```

---

# 3. 使用场景
## (1) 通用函数设计
### **示例**：实现一个遍历数组的通用函数（如问题中的 `ForEach`）：
```c
  void ForEach(void *arr, int elem_size, int num, void (*func)(void*))
  {
      for (int i = 0; i < num; i++)
      {
          // 通过 char* 指针按字节步进（因为 char 大小为 1 字节）
          func((char*)arr + i * elem_size);
      }
  }
```
---

## (2) 内存操作函数
### C 标准库中的 `memcpy` 和 `memset` 使用 `void*` 实现通用内存操作：
```c
  void* memcpy(void *dest, const void *src, size_t n);
```
---


## (3) 动态数据结构
### 在链表、树等数据结构中，`void*` 可存储任意类型的数据：
```c
struct Node{
	void *data;     // 泛型数据
	struct Node *next;
};
```
---
