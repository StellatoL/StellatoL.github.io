---
title: "C语言数据类型编程题目"
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
# 题目一：输出对应的数据类型<br>

## 程序如下：

```c
#include <stdio.h>
#include <string.h>
#include <ctype.h>
int main(void)
{
    char input[100];
    fgets(input, sizeof(input), stdin);
    input[strcspn(input, "\n")] = 0;
    if (strlen(input) == 1 && isprint(input[0]))
        printf("Data type: char, ASCII value: %d\n", (int)input[0]);
    else
    {
        char *endptr;
        long intValue = strtol(input, &endptr, 10);
        if (endptr != input && *endptr == '\0')
            printf("Data type: int\n");
        else
        {
            float floatValue = strtof(input, &endptr);
            if (endptr != input && *endptr == '\0')
                printf("Data type: float\n");
            else
            {
                double doubleValue = strtod(input, &endptr);
                if (endptr != input && *endptr == '\0')
                    printf("Data type: double\n");
                else
                    printf("Error\n");
            }
        }
    }
}
```

由于C语言把char类型视为特殊的int来处理，所以上述代码只能区分int，char和float。
对于数字的char类型（例如字符‘1’），以及double和float并不能很好地区分开来（目前也是没有想到什么好办法）

---
<!--more-->

# 题目二：显示身高

## 代码如下

```c
#include <stdio.h>
#define INCH 2.54
int main(void)
{
	int height;
	printf("请输入身高（英尺）：\n");
	scanf("%d",&height);
	printf("%.3f",INCH*height);
}
```

---
# 题目三：舍罕王的失算

## 代码如下

```c
#include <stdio.h>
#include <math.h>
#define SEED 25000.0
int main(void)
{
    double sum=0.0;
    for (int i=0;i<64;i++)
        sum += pow(2,i);
    printf("%.2lf Kg",sum/SEED);
}
```
算法效率最高的估计就是循环了，递归写法稍微简单一点但是空间复杂度会很大
