# Go 语言编写 CPython 扩展 goPy

goPy 是一个新的开源项目，实现了用 Go 语言来编写 CPython 扩展。

示例代码：

    
    
    package simple
     
    import (
    "fmt"
    "gopy"
    )
     
    func example(args *py.Tuple) (py.Object, error) {
    fmt.Printf("simple.example: %v\n", args)
    py.None.Incref()
    return py.None, nil
    }
     
    func init() {
    methods := []py.Method{
    {"example", example, "example function"},
    }
     
    _, err := py.InitModule("simple", methods)
    if err != nil {
    panic(err)
    }
    }

  

编译方法：

gopy pymodule.go

  

使用方法：

import simple

simple.example("hello", {123: True})

输出结果：

simple.example: [hello map[123:true]]

  

github开源项目地址：https://github.com/qur/gopy

  

