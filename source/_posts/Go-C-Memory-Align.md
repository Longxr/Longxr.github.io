---
title: golang 和 C++ 的内存对齐
categories:
  - go
tags:
  - go
  - C++
date: 2020-05-20 11:38:44
---

<Excerpt in index | 摘要>
golang 和 C++的内存对齐，基本一致，记住规则和对应类型的 size 即可 <!-- more -->
<The rest of contents | 余下全文>

## 内存对齐规则

1. 有效对齐值是固定值和结构体中最长数据类型长度中较小的那个。固定值系统默认为 32 位是 4, 64 位是 8，`#pragma pack(n)`设置了则是对应的 n。
2. 结构体第一个成员的 offset 为 0，以后每个成员相对于结构体首地址的 offset 都是 `min{该成员大小, 有效对齐值}`的整数倍，如有需要编译器会在成员之间加上填充字节。
3. 结构体的总大小为 有效对齐值 的整数倍，如有需要编译器会在最末一个成员之后加上填充字节。

## C++内存对齐

### 常见类型占用内存大小

| 类型/编译器 | 16 位编译器 | 32 位编译器 | 64 位编译器 |
| ----------- | ----------- | ----------- | ----------- |
| bool        | 1           | 1           | 1           |
| char        | 1           | 1           | 1           |
| char\*      | 2           | 4           | 8           |
| int         | 2           | 4           | 4           |
| float       | 4           | 4           | 4           |
| double      | 8           | 8           | 8           |
| long long   | 8           | 8           | 8           |

### 例子

```C++
#include <iostream>
#include <stdint.h>
using namespace std;
// #pragma pack(4)
class Part1 {
    bool a;
    int32_t b;
    int8_t c;
    int64_t d;
    char e;
};

class Part2 {
    char e;
    int8_t c;
    bool a;
    int32_t b;
    int64_t d;
};

int main(void){
    Part1 part1;
    Part2 part2;

    printf("part1 size: %zu\n", sizeof(part1));
    printf("part2 size: %zu\n", sizeof(part2));
}
// output:
// part1 size: 32,   if pack(4): part1 size: 24
// part2 size: 16
```

## golang 内存对齐

### 常见类型占用内存大小

| 类型/编译器 | 64 位编译器 |
| ----------- | ----------- |
| bool        | 1           |
| byte        | 1           |
| uintptr     | 8           |
| int         | 8           |
| float64     | 8           |
| string      | 16          |

### 例子

```go
package main
import (
    "fmt"
    "unsafe"
)

type Part1 struct {
    a bool
    b int32
    c int8
    d int64
    e byte
}

type Part2 struct {
    e byte
    c int8
    a bool
    b int32
    d int64
}

func main() {
    part1 := Part1{}
    part2 := Part2{}

    fmt.Printf("part1 size: %d, align: %d\n", unsafe.Sizeof(part1), unsafe.Alignof(part1))
    fmt.Printf("part2 size: %d, align: %d\n", unsafe.Sizeof(part2), unsafe.Alignof(part2))
}
// part1 size: 32, align: 8
// part2 size: 16, align: 8
```

## 例子分析

仅讨对齐固定值为 8 时 Part1 的情况，例子中的 a,b,c,d,e 占用大小分别为 1,4,1,8,1。填充字节用 0 表示，|之间表示 8 字节空间：

1. a 偏移为 0，|a...
2. b 偏移为 4 的整数倍，|a000b|...
3. c 偏移为 1 的整数倍，|a000b|c...
4. d 偏移为 8 的整数倍，|a000b|c0000000|d|...
5. e 偏移为 1 的整数倍，|a000b|c0000000|d|e...
6. 整个结构体对齐，补齐为 8 的整数倍，|a000b|c0000000|d|e0000000|
7. 得出总大小为 32
