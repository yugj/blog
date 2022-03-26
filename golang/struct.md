# 结构体

## 样例
```shell script
type manager struct {

	SideBar string
}

func(manager) test1() {
    // 等同于入参带 manager值传递方法 
    // func test1(s manager, name string)
}

func(*manager) test2() {
    // 等同于入参带 manager引用传递的方法
    // func test1(s *manager, name string)
}

func(s *manager) test3() {
    // 访问manager成员变量SideBar需要使用s访问
    // 刚开始一致困惑 为啥我无法通过manager.SideBar 访问，这边SideBar不像java语言静态变量可通过类.方法访问
    // golang 没有静态成员变量说法，这边只是结构体，需要通过参数访问，若真需要使用静态方式，可通过定义const变量提到结构体外使用
    s.SideBar = "xxx"
}
```

## 小结
使用结构体还是结构体指针，本质上与函数参数应该是值还是指针传递一致  
使用结构体扩展结构体方法，实现面向对象编程  

## LINK
https://go.dev/doc/faq#methods_on_values_or_pointers

