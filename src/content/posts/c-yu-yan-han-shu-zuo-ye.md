---
title: "C语言函数作业"
published: 2025-01-01
description: "float sumfloat data, int N"
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

# 题目一：求和

## 程序如下：

```c
float sum(float data[], int N)
{
    float sum=0.0;
    for (int i=0;i<N;i++)
        sum += data[i];
    return sum;
}
```
简单的for循环即可实现功能

---
<!--more-->

# 题目二：求均方差

## 程序如下：

```c
#include <math.h>
double Avg(int N, int data[])
{
    double avg=0.0;
    for (int i=0;i<N;i++)
        avg += data[i];
    return avg/N;
}
double StdDev(int N, int data[])
{
    double dev=0.0,avg=0.0;
    for (int i=0;i<N;i++)
        avg += data[i];
    avg /= N;
    for (int i=0;i<N;i++)
        dev += pow(data[i]-avg,2);
    return pow(dev/N,0.5);
}
```
---
# 题目三：字符串加密

## 程序如下：

```c
#include <ctype.h>
void cryptograp(char ch[], int n)
{
    for (int i=0;i<n;i++)
    {
        if (islower(ch[i]))
            ch[i] = (ch[i]-'a'+5)%26+'a';
        else if (isupper(ch[i]))
            ch[i] = (ch[i]-'A'+5)%26+'A';
    }
}
```
类似于循环队列的操作方式

---
# 题目四：万年历

## 程序如下：

```c
void ShowDate(int y, int m)
{
	int daysInMonth = GetDaysofMonth(y, m);
	int firstDay = GetFirstDayInTable(y, m); 
	printf("***********************************\n");
	printf("Sun Mon Tue Wen Thur Fri Sta\n"); 
	for (int i = 0; i < firstDay; i++)
		printf("    ");
	for (int day = 1; day <= daysInMonth; day++)
	{
		printf("%2d ", day);
		if ((day + firstDay) % 7 == 0)
			printf("\n");
	}
	printf("\n***********************************\n");
}
```
