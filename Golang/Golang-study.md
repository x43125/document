# Golang基础学习

## 1 特性：静态编程语言

- 自动回收垃圾
- 更丰富的内置类型: map
- 函数多返回值: 不感兴趣的用 “_” 占位
- 错误处理: 无需层层 try-catch
- 匿名函数和闭包
- 类型和接口：非侵入式接口
- 并发编程：goroutine
- 反射:官方不建议使用
- 语言交互性：Cgo

## 2 第一个Go程序

```go
package main

import "fmt"

func main() {
   fmt.Println("Hello World")
}
```

