---
title: 不同语言在函数内部定义函数
categories:
    - python
tags:
    - C++
    - go
    - python
    - js
date: 2020-04-04 11:41:24
---

<Excerpt in index | 摘要>
在 LeetCode 刷题的时候，题解有的大佬给出的答案很优秀，是 python 的，想抄作业发现有的功能函数都定义在答案函数的内部，主要是闭包操作外部变量方便。不同语言在函数内部定义函数稍有不同，于是记录下自己可能用到的语言在函数中定义函数的方式。<!-- more -->
<The rest of contents | 余下全文>

## python

python 定义内部函数加个 def 就行，和使用变量差别不大，可以在参数列表后跟->指定返回值。

```python
def outerFunc() -> None:
    a: int = 1
    def innerFunc(b: int) -> int:
        nonlocal a  #不加的话不会修改外部变量的值
        a = 2
        return a + b
    print("ans:{ans}, a:{a}".format(ans=innerFunc(3), a=a)) #不加nonlocal ans:5, a:1; 加上nonlocal ans:5, a:2

if __name__ == '__main__':
    outerFunc()
```

## C++

C++主要是使用 Lambda 函数来实现，可以在参数列表后跟->指定返回值。在函数前有个`[]`的引出符，不同符号对于外部变量的处理方式不同：

-   [a]，表示以值传递方式捕捉变量 a
-   [=]，表示值传递捕捉所有父作用域变量，以值传递的形式捕获的变量如果要修改，需要在函数参数列表后面加上 mutable
-   [&a]，表示以引用传递方式捕捉变量 a
-   [&]，表示引用传递捕捉所有父作用域变量
-   [this]，表示值传递方式捕捉当前的 this 指针
-   [=,&a]，表示以引用传递方式捕捉 a,值传递方式捕捉其他变量

```C++
#include <iostream>
using namespace std;

void outerFunc() {
    int a = 1;
    auto innerFunc = [&a](int b)->int {
        a = 2;
        return a + b;
    };
    cout << "ans:" << innerFunc(3) << ", a:" << a;  //ans:5, a:2
}

int main(int argc, char* argv[]) {
    outerFunc();
    return 0;
}
```

## golang

golang 和 python 差不多，函数返回值直接写在参数后边。不过内部定义不能写函数名，调用可以通过变量来使用。

```go
package main
import "fmt"

func outerFunc() {
    a := 1
    // innerFunc := func innerFuncTest(b int) int { //error
    innerFunc := func (b int) int {
        a = 2
        return a + b
    }
    fmt.Printf("ans:%d, a:%d", innerFunc(3), a) //ans:5, a:2
}

func main(){
    outerFunc()
}
```

## js

js 参数和返回值不用指定，箭头函数连接参数和返回的对象。

```js
function outerFunc() {
    a = 1;
    innerFunc = (b) => {
        a = 2;
        return a + b;
    };
    console.log('ans:', innerFunc(3), ', a:', a); //ans: 5 , a: 2
}

outerFunc();
```

## 不同之处

-   只有 python 需要加个 nonlocal 才能修改外部函数的变量，其他语言默认就是可以修改外部函数的变量的
