# GO中的面向对象编程


<!--more-->

# Object Oriented Programming 面向对象程序设计

## 多态

Golang是一种静态类型的编程语言，没有像传统面向对象语言那样提供显式的多态特性。然而，Golang通过接口的使用可以实现多态的效果。

#### What

多态是面向对象编程中的一个重要概念，它允许使用不同的对象来执行相同的操作，从而实现了代码的灵活性和可扩展性。简单来说，多态意味着一个方法可以根据接收到的不同对象的类型来执行不同的行为。

#### Why

使用多态可以使代码更加灵活和可扩展。通过多态，我们可以在不改变原有代码的基础上，新增或修改对象的行为。这种灵活性使得代码更易于维护和扩展，并且提高了代码的可复用性。

#### How

在Golang中，多态可以通过接口来实现。接口是一种抽象类型，它定义了一组方法的集合。如果一个对象实现了接口所定义的所有方法，那么该对象就可以被视为该接口类型的实例。通过这种方式，我们可以将不同类型的对象视为相同的接口类型，从而实现多态的效果。

```
package main

import "fmt"

type Shape interface {
	Area() float64
}

type Rectangle struct {
	width  float64
	height float64
}

func (r Rectangle) Area() float64 {
	return r.width * r.height
}

type Circle struct {
	radius float64
}

func (c Circle) Area() float64 {
	return 3.14 * c.radius * c.radius
}

func GetArea(s Shape) float64 {
	return s.Area()
}

func main() {
	r := Rectangle{width: 5, height: 3}
	c := Circle{radius: 4}

	fmt.Println("Rectangle area:", GetArea(r))
	fmt.Println("Circle area:", GetArea(c))   
}
```

## 封装

封装是面向对象编程中的一种重要概念，它指的是将数据和对数据的操作封装在一起，形成一个独立的单元。在Golang中，封装通过访问控制来实现，即使用大小写字母来决定成员的可见性。

#### What

封装是将数据和对数据的操作封装在一起，形成一个单独的实体。在Golang中，封装通过使用大小写字母来决定成员的可见性。小写字母开头的标识符为私有成员，只能在当前包内访问；大写字母开头的标识符为公共成员，可以在任何包中访问。

#### Why

封装的主要目的是将数据隐藏起来，防止外部直接访问和修改。通过封装，可以实现对数据的保护，避免数据被意外修改或滥用。同时，封装还可以隐藏实现细节，提高代码的安全性和可维护性。

#### How

可以通过结构体和方法来实现封装。结构体是一种自定义数据类型，可以将不同类型的数据封装在一起。方法是与结构体关联的函数，用于对结构体的操作和行为进行封装。

```
package main

import (
	"fmt"
)

type Person struct {
	name   string
	age    int
	gender string
}

func (p *Person) SetName(name string) {
	p.name = name
}

func (p *Person) GetName() string {
	return p.name
}

func main() {
	p := Person{
		name:   "Alice",
		age:    20,
		gender: "Female",
	}

	fmt.Println(p.GetName()) // 输出：Alice

	p.SetName("Bob")
	fmt.Println(p.GetName()) // 输出：Bob
}
```

## 继承

### why

继承的特点是可以在在原来的类的属性基础上增加新的属性。
把通用型的代码同一编写在一个基类中 ，在子类中添加新的属性。
可以重用前期项目中经过验证和检查的bug少的类，不会破坏原有的代码架构。

### 继承规则

- 在派生类没有改写基类的成员方法时，相应的成员方法被继承。
- 派生类可以直接调用基类的成员方法，譬如基类有个成员方法为Base.Func()，那么Derived.Func()等同于Derived.Base.Func()
- 如果派生类的成员方法名与基类的成员方法名相同，那么基类方法将被覆盖或叫隐藏。比如基类和派生类都有成员方法Func()，那么Derived.Func()将只能调用派生类的Func()方法，如果要调用基类版本，可以通过Derived.Base.Func()来调用。

```
package main

import "fmt"

type Base struct {
    BaseName string
}

func (b *Base) PrintName() {
    fmt.Println(b.BaseName)
}

type Derived struct {
    DerivedName string
    Base
}

func (d *Derived) PrintName() {
    fmt.Println(d.DerivedName)
}

func main() {
    d := &Derived{}
    d.BaseName = "BaseStruct"
    d.DerivedName = "DerivedStruct"
    d.Base.PrintName() // BaseStruct
    d.PrintName()      // DerivedStruct
}
```

## 总结

- 在 go 语言中，type name struct{} 结构体，相当于其他语言中的 class 类的概念。
- 在其他语言中，方法是直接写在在 类 里面的，而在 go 语言中，我们对于该结构体，如果存在方法，是以 func (结构体名) 方法名{}，即 func(c Cat) sleep{} 的方式来声明方法。

