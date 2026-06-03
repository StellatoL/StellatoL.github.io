---
title: "C语言数组作业"
published: 2025-01-01
description: "int mainvoid"
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
# 题目一：计算总和

## 程序如下：

```c
#include <stdio.h>
int main(void)
{
    int n,sum=0;
    scanf("%d",&n);
    int num[n];
    for (int i=0;i<n;i++)
    {
        scanf("%d",&num[i]);
        sum += num[i];
    }
    printf("%d",sum);
}
```
---
<!--more-->

# 题目二：反转字符串

## 程序如下：

```c
#include <stdio.h>
#include <string.h>
int main(void)
{
	char str[99];
	scanf("%s",str);
	int len = strlen(str);
	for (int i=0;i<len;i++)
		printf("%c",str[len-i-1]);
}
```
---
# 题目三：最大最小元素

## 程序如下：

```c
#include <stdio.h>
int main(void)
{
    int n,max;
    scanf("%d",&n);
    int num[n];
    for (int i=0;i<n;i++)
        scanf("%d",&num[i]);
    int min=max=num[0];
    for (int i=0;i<n;i++)
    {
        if (num[i]>max)
            max = num[i];
        if (num[i]<min)
            min = num[i];
    }
    printf("min=%d,max=%d",min,max);
}
```
