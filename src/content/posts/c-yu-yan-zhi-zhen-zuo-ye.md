---
title: "C语言指针作业"
published: 2025-01-01
description: "int main"
tags:
  - "基础语法"
  - "C"
category: "C语言"
categoryPath:
  - "RM_PRIME"
  - "RM算法"
  - "作业文件"
  - "C语言"
draft: false
---

---
# 题目一：输出Hello


```c
#include <stdio.h>
int main()
{
	char s[] = "Hello";
	char * p;
	for(p=s;*p!=‘\0’;p++)
	{
		printf("%c", *p);
	}
	return 0;
}
```

---

<!--more-->

# 题目二：输出Tesla

```c
#include <stdio.h>
void Print(const char * p1, const char * p2)
{
	for(;(p2-p1)>0;p1++)
	{
		printf("%c", *p1);
	}
}
int main()
{
	const char * s = "Tesla123";
	Print(s, s+5);
	printf("\n");
	Print(s, s+3);
	printf("\n");
	return 0;
}
```

---

# 题目三：ForEach

```c
#include <stdio.h>
#include <string.h>
void ForEach(void *a, int width, int num, void (*f)(void *)) 
{
	for (int i = 0; i < num; ++i)
	{
		f((char*)a + width * i);
	}
}
void PrintSquare(void *p)
{
	int *q = (int*)p;
	int n = *q;
	printf("%d,", n * n);
}
void PrintChar(void *p)
{
	char *q = (char*)p;
	printf("%c,", *q);
}
int main()
{
	int a[5] = {1, 2, 3, 4, 5};
	char s[] = "hello!";
	ForEach(a, sizeof(int), sizeof(a)/sizeof(int),PrintSquare);
	printf("\n");
	ForEach(s, sizeof(char), strlen(s), PrintChar);
	printf("\n");
	return 0;
}
```

---

# 题目四：Memcpy-1
```c
#include <stdio.h>
void Memcpy(char *src, char *dest, int n)
{
	for (int i=0;i<n;i++)
	{
		dest[i] = src[i];
	}
}
int Strlen(char *s)
{
	int i;
	for (i = 0; s[i]; ++i);
	return i+1;
}
int main()
{
	int a;
	char s1[30];
	char s2[30];
	int t;
	scanf("%d", &t);
	for (int i = 0; i < t; ++i)
	{
		scanf("%d", &a);
		int b = 99999999;
		Memcpy((char*)&a, (char*)&b, sizeof(int));
		printf("%d\n", b);
	}
	for (int i = 0; i < t; ++i)
	{
		scanf("%s", s1);
		Memcpy(s1, s2, Strlen(s1));
		printf("%s\n", s2);
	}
	return 0;
}
```

---

# 题目五：double
```c
#include <stdio.h>
void Double(int * p, int n)
{
	for(int i = 0; i < n; ++i)
	p[i] *= 2;
}
int main()
{
	int a[3][4] = {{1, 2, 3, 4}, {5, 6, 7, 8}, {9, 10, 11, 12}};
	Double(a[1],8);
	for(int i = 0; i < 3; ++i)
	{
		for(int j = 0; j < 4; ++j)
			printf("%d,", a[i][j]);
		printf("\n");
	}
	return 0;
}
```

---

# 题目六：Memcpy-2
```c
#include <stdio.h>
void Memcpy(void *src, void *dest, int size)
{
	char *csrc = *src;
	char *cdest = *dest;
	for (int i=0;i<size;i++)
	{
		cdest[i] = csrc[i];
	}
}
void Print(int *p, int size)
{
	for (int i = 0; i < size; ++i)
	{
		printf("%d", p[i]);
		if (i < size - 1)
		{
			printf(",");
		}
	}
	printf("\n");
}
int main()
{
	int a[10];
	int n;
	scanf("%d", &n);
	for (int i = 0; i < n; ++i)
	{
		scanf("%d", &a[i]);
	}
	int b[10] = {0};
	Memcpy(a, b, sizeof(a));
	Print(b, n);
	int c[10] = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
	Memcpy(c, c + 5, 5 * sizeof(int)); // 将c的前一半拷贝到后一半
	Print(c, 10);
	char s[10] = "123456789";
	Memcpy(s + 2, s + 4, 5); // 将s[2]开始的5个字符拷贝到s[4]开始的地方
	printf("%s\n", s);
	char s1[10] = "123456789";
	Memcpy(s1 + 5, s1 + 1, 4); // 将s1[5]开始的4个字符拷贝到s1[1]开始的地方
	printf("%s\n", s1);
	return 0;
}
```

---

# 题目七：MyMax
```c
#include <stdio.h>
#include <stdlib.h>
/* TODO */
int Compare1(void * n1, void * n2)
{
	int * p1 = (int * )n1;
	int * p2 = (int * )n2;
	return ((*p1)%10) - ((*p2)%10);
}
int Compare2(void * n1, void * n2)
{
	int * p1 = (int * )n1;
	int * p2 = (int * )n2;
	return *p1 - *p2;
}
#define eps 1e-6
int Compare3(void * n1, void * n2)
{
	float * p1 = (float * )n1;float * p2 = (float * )n2;
	if( * p1 - * p2 > eps)
		return 1;
	else if(* p2 - * p1 > eps)
		return -1;
	else
		return 0;
}
// MyMax函数，根据比较函数找到数组中的最大元素
void *MyMax(void *array, size_t size,int n, int (*compare)(const void *, const void *))
{
	void *max_element = array;
	for (int i = 1; i < n; ++i)
	{
		if (compare((char *)array + i * size, max_element) > 0)
		{
			max_element = (char *)array + i * size;
		}
	}
	return max_element;
}
int main()
{
	int t;
	scanf("%d", &t);
	while (t--)
	{
		int n;
		scanf("%d", &n);
		int a[10];
		float d[10];
		for (int i = 0; i < n; ++i)
		{
			scanf("%d", &a[i]);
		}
		for (int i = 0; i < n; ++i)
		{
			scanf("%f", &d[i]);
		}
		int * p = (int *)MyMax(a, sizeof(int), n, Compare1);
		printf("%d\n", *p);
		p = (int *)MyMax(a, sizeof(int), n, Compare2);
		printf("%d\n", *p);
		float * pd = (float *)MyMax(d, sizeof(float), n, Compare3);
		printf("%f\n", *pd);
	}
	return 0;
}
```
##### 这段代码找不到任何缺漏的地方，感觉这个/*TODO*/更像是一个注释什么的
##### 硬要说不足的地方，就是三个Compare函数中参数应该带const，整个函数的功能性并没有缺失

---

# 题目八：指向指针的指针
```c
#include <stdio.h>
int main()
{
	int x,y,z;
	x = 10;
	y = 20;
	z = 30;
	int * a[3] = {&x, &y, &z};
	for(int **p=a; p < a + 3; ++p)
	{
		printf("%d\n",*(*p));
	}
	return 0;
}
```

---

# 题目九：SwapMemory
```c
#include <stdio.h>
void SwapMemory(void * m1, void * m2, int size)
{
	char temp;
	char *cm1 = (char *)m1;
	char *cm2 = (char *)m2;
	for (int i=0;i<size;i++)
	{
		temp = cm1[i];
		cm1[i] = cm2[i];
		cm2[i] = temp;
	}
}
void PrintIntArray(int * a, int n)
{
	for(int i = 0;i < n; ++i)
	{
		printf("%d,", a[i]);
	}
	printf("\n");
}
int main()
{
	int a[5] = {1, 2, 3, 4, 5};int b[5] = {10, 20, 30, 40, 50};
	SwapMemory(a, b, 5 * sizeof(int));
	PrintIntArray(a, 5);
	PrintIntArray(b, 5);
	char s1[] = "12345";
	char s2[] = "abcde";
	SwapMemory(s1, s2, 5);
	printf("%s\n", s1);
	printf("%s\n", s2);
	return 0;
}
```
---
