---
title: Go语言中的defer
categories:
    - go
tags:
    - go defer
date: 2020-02-29 21:02:06
---

<Excerpt in index | 摘要>
Go 语言中的 defer 语句，可以在函数结束或者是出现异常时保证执行相关释放资源的操作，和 C++中的析构函数有点类似。像是文件的关闭，socket 连接的关闭这些操作都可以使用 defer 语句来延迟执行。<!-- more -->
<The rest of contents | 余下全文>

# 特性

## 执行时机

-   函数返回时（碰到的第一个 return 的位置）
-   函数执行完毕（没有 return）
-   函数碰到异常，抛出 panic 时
-   调用 os.Exit 时 defer 不会被执行

## 执行顺序

1. 返回值赋值给返回值变量，函数没有写返回值变量名则赋值给匿名变量
2. 声明了 defer 语句的话就执行，有多个 defer 语句则按照声明顺序反过来分别执行
3. 函数返回返回值 ret

## 延迟执行

-   defer 要先声明后才会延迟执行，如果函数已经走到 return，return 后面声明的 defer 语句是不会执行的
-   如果有参数传递给 defer，值传递是在声明 defer 语句时就已经传值了，并不是在执行函数时去获取值

# 代码示例

## 返回结果判断

```
func ret1() int {
	x := 1
	defer func() {
		x++     //后执行，修改的是局部变量x
	}()
	return x    //先执行，匿名变量赋值为1
}

func ret2() (x int) {
	x = 1
	defer func() {
		x++     //修改了返回值
	}()
	return x    //返回值就是x
}

func TestRet(t *testing.T) {
	t.Log("ret1(): ", ret1())
	t.Log("ret2(): ", ret2())
}

// 执行结果：
// ret1():  1
// ret2():  2
```

## defer 传入参数、多个 defer 执行顺序

```
func calc(index, a, b int) int {
	ret := a + b
	fmt.Println(index, a, b, ret)
	return ret
}

func TestParam(t *testing.T) {
	a := 1
	b := 2
	defer calc(1, a, calc(2, a, b)) // defer calc(1, 1, 3)
	a = 0
	defer calc(3, b, calc(4, a, b)) // defer calc(3, 2, 2)
	b = 1
}

// 执行结果：
// 2 1 2 3
// 4 0 2 2
// 3 2 2 4
// 1 1 3 4
```

## 执行时遇到 panic

```
func TestPanic(t *testing.T) {
	defer func() {
		t.Log("Clear resources")
	}()
	t.Log("Started")
	panic("Fatal error")
	t.Log("End")            //panic后不会执行
}

// 执行结果：
// Started
// Clear resources
// panic: Fatal error [recovered]
```
