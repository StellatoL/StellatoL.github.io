---
title: "循环与判断"
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
# 题目一：将三个整数从大到小输出<br>
## 程序如下：

```c
#include<stdio.h>
int main(void)
{
	int a,b,c,n;
	scanf("%d %d %d",&a,&b,&c);
	if (a<(b<c?b:c))//a是最小值
		printf("%d %d %d",(b>c?b:c),(b>c?c:b),a);
	else if (a>(b>c?b:c))
		printf("%d %d %d",a,(b>c?b:c),(b>c?c:b));
	else
		printf("%d %d %d",(b>c?b:c),a,(b>c?c:b));
	return 0;
}
```

用三元表达式替换了一下if else语句~~（单纯懒得写而已）~~

---
<!--more-->

# 题目2：判断100-200的素数，并输出所有素数

## 程序如下：

```c
#include<stdio.h>
#include<math.h>
int main(void)
{
    int N=200,i,flag,count=0;
    for (i=100;i <= N;i++)
    {
        flag = 1;
        int n = (int)sqrt(i);
        for (int k=2;k<=n;k++)
        {
            if (i%k == 0)
            {
                flag = 0;
                break;
            }
        }
        if (flag)
        {
            printf("%5d",i);
            count +=1;
            if (count==8)
            {
                printf("\n");
                count = 0;
            }
        }
    }
    return 0;
}
```

为了美观，八个为一行输出
（其实判断是否为素数完全可以写成一个函数，这样主程序就不会这么臃肿）

---
# 题目3：输入一个正整数，将其分解质因数

## 程序如下：

```c
#include<stdio.h>
#include<math.h>

int isprime(int);

int main(void)
{
    int n, i = 2;
    int N = 0, p = 0;
    int a[101];
    scanf("%d", &n);
    N = n;
    while (N > 1)
    {
        while (N % i == 0 && isprime(i))
        {
            N /= i;
            a[p++] = i;
        }
        i++;
    }
    printf("%d=", n);
    for (int j = 0; j < p; j++)
    {
        if (j<p-1)
            printf("%d*", a[j]);
        else
            printf("%d", a[j]);
    }
    if (p == 0)
        printf("%d", n);
    return 0;
}

int isprime(int n)
{
    if (n < 2)
        return 0;
    for (int k = 2; k <= (int)sqrt(n); k++)
    {
        if (n % k == 0)
        {
            return 0;
        }
    }
    return 1;
}
```

对上一题说的这题实现了：）

Date: 24.12.1
