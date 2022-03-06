---
title: smali 语法
toc: true
mathjax: false
date: 2022-03-05
updated: 2022-03-05
categories:
   - [杂货铺]
tags:
   - smali
   - Android
---

> 记录一下 smali 的基本语法，方便回顾

## 前言

smali 语法与汇编语法挺相似的，
简要做一些笔记，仅仅只是一些简单的语法，
更多信息请查看文末的链接

## 数据类型

以下是一些基本数据类型的表示
| smali | Java    | smali | Java     |
|:------|:--------|:------|:---------|
| I     | int     | C     | char     |
| S     | short   | Z     | boolean  |
| J     | long    | V     | void     |
| F     | float   | L     | object   |
| D     | double  | B     | byte     |
| [     | 数组    | [[    | 二维数组 |

例子

`[I` 表示 int[] ，整型数组  
`Ljava/lang/String` 表示 String 类型，在 Java 中，
String 为 java.lang 包中的一个对象

## 流程控制

### 条件判断

语法如下：  
`if-{cond} v1, v0, :cond_0`

`{cond}` 是判断条件，在后面会列出  
`:cond_0` 是跳转的位置，类似与 C 语言中的 `goto` 标签

比如，当 v0 >= v1 时，跳转到 `cond_0` 是这样写的：
`if-ge v0, v1, :cond_0`  

全部条件判断语句：  
`if-eq v0, v1`: `if (v0 == v1)`，eq 指 `eq`ual  
`if-ne v0, v1`: `if (v0 != v1)`，ne 指 `n`o `e`qula  
`if-ge v0, v1`: `if (v0 >= v1)`，ge 指 `g`reater than or `e`qual  
`if-le v0, v1`: `if (v0 <= v1)`，le 指 `l`ess than or `e`qual  
`if-gt v0, v1`: `if (v0 > v1)`，gt 指 `g`reater `t`han  
`if-lt v0, v1`: `if (v0 < v1)`，le 指 `l`ess `t`han

### 循环

直接代码解释比较方便，例子参见文末链接

```java
public void smaliWhile() {
    int a = 0;
    while(a <= 3) {
        a++;
    }

    int b = 0;
    for(int i = 0;i<3;i++){
        b++;
    }

    int c = 0;
    do{
        c++;
    }while (c <= 3);
}
```

对应的 smali 语法

```smali
.method public smaliWhile()V
    .locals 5

    .line 62
    // 4 字节常量，值为 0
    const/4 v0, 0x0

    // 赋值给 a
    move v1, v0

    .line 63
    .local v1, "a":I
    // goto 标签
    :goto_0
    const/4 v2, 0x3

    // if (a > 3), goto cond_0
    if-gt v1, v2, :cond_0

    .line 64
    // else v1 = v1 + 1
    add-int/lit8 v1, v1, 0x1

    goto :goto_0

    .line 68
    :cond_0
    const/4 v3, 0x0

    .line 69
    .local v3, "b":I
    move v4, v3

    move v3, v0

    .local v3, "i":I
    .local v4, "b":I
    :goto_1
    if-ge v3, v2, :cond_1

    .line 70
    add-int/lit8 v4, v4, 0x1

    .line 69
    add-int/lit8 v3, v3, 0x1

    goto :goto_1

    .line 74
    .end local v3
    :cond_1
    nop

    .line 76
    .local v0, "c":I
    :cond_2
    add-int/lit8 v0, v0, 0x1

    .line 77
    if-le v0, v2, :cond_2

    .line 78
    return-void
.end method
```

与汇编语法挺相似的，不多解释  
~~其实是懒得写笔记~~

## 函数定义

一般定义长这个样子:
`funcName (paraType1paraType2...)returnType`

- `funcName` 表示函数名
- `paraType1` 和 `paraType2` 表示传入参数的类型，他们之间**没有**空格
- `returnType` 表示返回值类型

例子

`foo (IF)Z` 是 `boolean foo (int, float)`

`bar([ILjava/lang/String)Z` 是 `boolean bar(int[], String)`

## 内部类

smali 使用 `$` 表示内部类，像这样子
`LpackageName/objectName$subObjName`

## 参考

1. [Smali coding guide for beginners](https://forum.xda-developers.com/t/guide-smali-coding-guide-for-beginners.2193735)
2. [Smali Introduction Manual](https://programmer.help/blogs/smali-introduction-manual.html)
