---
title: golang反射机制介绍
categories: golang
tags:
- golang
- reflect

---

#golang反射机制介绍

Go语言提供了一种机制，在编译时不知道类型的情况下，可更新变量、在运行时查看值、调用方法以及直接对他们的布局进行操作，这种机制称为反射(reflection)。

## reflect.Type 和 reflect.Value
反射功能由reflect包提供，它定义了两个重要的类型：Type 和 Value，分别表示变量的类型和值。反射包里面，提供了reflect.TypeOf 和 reflect.ValueOf，返回被检查对象的Type 和 Value:
>func TypeOf(i interface{}) Type
>func ValueOf(i interface{}) Value

可以看到，TypeOf和ValueOf的参数是interface{}类型，当把一个具体值赋给一个interface{}类型时，会发生一个隐式的类型转换，转换会生成一个包含两个部分内容的接口值：动态类型部分是操作数的类型，动态值部分是操作数的值。

reflect.TypeOf返回接口值对应的动态类型，注意返回的时具体值类型而不是接口类型。比如下面的输出的是"*os.File"而不是"io.Writer":
>var w io.Writer = os.Stdout
>fmt.Println(reflect.TypeOf(v)) // "*os.File"

同理，reflect.ValueOf返回接口动态值，以reflect.Value的形式返回，与reflect.TypeOf类似:
>v := reflect.ValueOf(3)
>fmt.Println(v) //"3"

调用Value的Type方法会把它的类型以reflect.Type方式返回：
>t := v.Type() //an reflect.Type
>fmt.Println(t.String()) //"int"

reflect.ValueOf的逆操作是reflect.Value.Interface方法，它返回一个interface{}接口值，与reflect.Value包含同一个具体值:
>v := reflect.ValueOf(3)
>x := v.Interface()
>i := x.(int)
>fmt.Printf("%d\n", i) //"3"

reflect.Value 与 interface{}都可以包含任意值。二者的区别是空接口(interface{})隐藏了值的布局信息、内置操作和相关方法，除非我们知道它的动态类型，并用一个类型断言来渗透进去，否则我们对所包含的值能做的事情很少。作为对比，Value有很多方法可以用来分析所包含的值，而不用知道它的类型。

代码示例:
>package main
>
>import (
>    "fmt"
>    "reflect"
>)
>
>func main() {
>    var x float64 = 3.4
>    fmt.Println("type:", reflect.TypeOf(x))
>    v := reflect.ValueOf(x)
>    fmt.Println("value:", v)
>    fmt.Println("type:", v.Type())
>    fmt.Println("kind:", v.Kind())
>    fmt.Println("value:", v.Float())
>    fmt.Println(v.Interface())
>    fmt.Printf("value is %5.2e\n",v.Interface())
>    y := v.Interface().(float64)
>    fmt.Println(y)
>}

输出:
>type: float64
value: 3.4
type: float64
kind: float64
value: 3.4
3.4
value is 3.40e+00
3.4

## 利用reflect.Value修改值
Golang中，反射不只用来解析变量值，还可以改变变量的值，接下来介绍如何用reflect.Value来设置变量的值。

我们需要知道的是，在我们传参给reflect.ValueOf获取reflect.Value时，传入的是参数的拷贝，所以reflect.ValueOf获得的是传入参数的拷贝的reflect.Value，对于它的修改对原来的数据是没有影响的。所以如果我们想要修改原来的值，就需要用指针，然后还需要利用Elem方法获取指针指向的变量，通过修改Elem()返回的变量，才能真正的修改原始变量的值，如下：
>x := 2
>v := reflect.ValueOf(&x)
>e := v.Elem()
>e.Set(reflect.ValueOf(3))
>fmt.Println(x) //"3"

这里，还有一些基本类型特化的Set变种：SetInt、SetUint、SetString、SetFloat等：
>e.SetInt(4)
>fmt.Println(e) //"4"

在更新变量前，我们可以用CanSet方法来判断是否可更改，如果更改一个不可更改的reflect.Value，将导致panic：
>fmt.Println(e.CanSet()) //"true"
>d := reflect.ValueOf(2)
>fmt.Println(d.CanSet()) //"false"
>d.Set(reflect.ValueOf(4)) // 这里将导致panic

## 注意事项
Golang的反射是i一个功能和表达能力都很强大的工具，但是使用它是要谨慎，具体有如下一些原因：
*基于反射的的代码都很脆弱。能导致编译器报告类型错误的每种写法，在反射中都有一个对应的的误用写法。编译器在编译时就能向你报告的这个错误，而反射错误则要等到执行时才以崩溃的方式来报告。并且反射还降低了自动重构和分析工具的安全性和准确度，因为它们无法检测到类型信息。
*我们要避免使用反射的原因是，反射的相关操作无法做静态类型检查，所以对于大量使用反射的代码是很难理解的。对于接受interfacef{}或者reflect.Value的函数，一定要写清楚期望的参数类型和其他限制条件
*基于反射的函数会比特定类型优化的函数慢一两个数量级。测试很适合用反射，但是对于关键路径上的函数，最好避免使用反射。



