# Swift函数派发机制



> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [harryyan.github.io](https://harryyan.github.io/2021/08/27/Swift%E5%87%BD%E6%95%B0%E6%B4%BE%E5%8F%91%E6%9C%BA%E5%88%B6/)

> 一个令人困惑的小问题大家请看下面这段代码 (摘自 objc.io twitter 的 swift quiz) 1234567891011121314151617protocol Drawing { ......



大家请看下面这段代码 (_摘自 objc.io twitter 的 swift quiz_)

```
protocol Drawing {
  func render()
}

extension Drawing {
  func circle() { p rint("protocol")}
  func render() { circle()}
}

class SVG: Drawing {
  func circle(){ print("class") }
}

SVG().render()

// what's the output?
```

请给出你的答案😄

根据当时的统计，43% 选择了 protocol, 57% 选择了 class。但真理往往掌握在少数人手中，正确答案是 protocol。objc 给出的解释是: **circle 函数声明在 protocol 的 extension 里面，所以不是动态派发，并且类没有实现 render 函数，所以输出为 protocol.** 更为准确的说法应该是: **extension 中声明的函数是静态派发，编译的时候就已经确定了调用地址，类无法重写实现。**

在更深入研究 Swift 函数派发机制之前，我们有必要了解下函数派发的基本知识。函数派发就是 CPU 在内存中找到该函数地址并调用的过程。函数派发有三种类型: **静态派发，函数表派发和消息派发。**当我们在选择时，需要平衡程序的执行效率和动态性，选择最适合当下情景的派发方式。



直接派发
--------------------

直接派发是三种派发方式中最快的。CPU 直接按照函数地址调用，使用最少的指令集，办最快的事情。当编译器对程序进行优化的时候，也常常将函数内联，使之成为直接派发方式，优化执行速度。我们熟知的 C++ 默认使用直接派发方式，在 Swift 中给函数加上 **final** 关键字，该函数也会变成直接派发的方式。当然，有利就有弊，直接派发最大的弊病就是没有动态性，不支持继承。



函数表派发
-----------------------

这种方式是编译型语言最常见的派发方式，他既保证了动态性也兼顾了执行效率。函数所在的类会维护一个” 函数表”，也就是我们熟知的**虚函数表**。该函数表存取了每个函数实现的指针。每个类的 vtable 在编译时就会被构建，所以与直接派发相比只多出了两个读取的工作: **读取该类的 vtable** 和**该函数的指针**。理论上说，函数表派发也是一种高效的方式。不过和直接派发相比，编译器对某些含有副作用的函数却无法优化，也是导致函数表派发变慢的原因之一。而且 Swift 类扩展里面的方法无法动态加入该类的函数表中，只能使用静态派发的方式，这也是函数表派发的缺陷之一。

我们来看如下代码:

```
class Vehiche {
    func run() {}
    func brake() {}
}

class Car: Vehiche {
    override func brake() {}
    func speedUp() {}
}
```

当前情景下，编译器会创建两个函数表: 一个属于 **Vehiche** 类，另一个属于 **Car** 类，内存布局如下:

<img src="https://sylarimage.oss-cn-shenzhen.aliyuncs.com/uPic/Screen_Shot_2019-06-05_at_12.06.52_PM_n3inuw.png" style="zoom: 50%;" />

```
let car = Car()
car.brake()
```

当调用函数 **brake** 时，过程如下:

1.  读取该对象 (0XB00) 的 vtable.
2.  读取 **brake 函数指针** 0x222.
3.  跳转到地址 **0X222**，读取函数实现.



消息派发
--------------------

这种派发方式是三种里面最动态的一种方式。由于 Swfit 使用的依旧是 Objc 的运行时系统，所以这里的消息派发其实也就是 Objc 的 **Message Passing**。

```
id returnValue = [someObject messageName:parameter];
```

**someObject** 就是接收者，**messageName** 就是选择器，选择器和参数一起被称为 “**消息** “。  
当编译时，编译器会将该消息转换成一条标准的 C 语言调用：

```
id returnValue = objc_msgSend(someObject, @selector(messageName:), parameter);
```

objc_msgSend 函数回一句接收者和选择器的类型来调用适当的方法，它会去接收者所属类中搜索其方法列表，如果能找到，则跳转到对应实现；若找不到，则沿着继承体系继续向上查找，若能找到，则跳转；如果最终还是找不到，那就执行边界情况的操作，例如 **Message forwarding**。

<img src="https://sylarimage.oss-cn-shenzhen.aliyuncs.com/uPic/Screen_Shot_2019-06-05_at_8.33.29_AM_nwr7rv.png" style="zoom:50%;" />

这样做的好处在哪里呢？这种运作方式的关键在于开发者可以在运行时改变函数的行为，也就是我们常说的 **Swizzling**。Swizzling 经常用来配置服务以及 hook 某些测试 case。

KVO 就是使用 swizzling 实现的。

这种派发方式的流程步骤似乎很多，所幸的是 objc_msgSend 会将匹配的结果缓存到 **fast map** 中，而且每个类都有这样一块缓存；若是之后发送相同的消息，执行速率会很快。

了解了函数派发的基本知识，我们来看看 Swift 如何处理函数派发以及如何证明该种派发。我们先来看一张总结表:

<img src="https://sylarimage.oss-cn-shenzhen.aliyuncs.com/uPic/table_fbcuhz.png" style="zoom:50%;" />

从上表中我们可以直观的总结出：**函数的派发方式和以下两点相关联:**

1.  对象类型; 值类型总是使用直接派发 (静态派发，因为他们没有继承体系)
2.  函数声明的位置; 直接在定义中声明和在扩展中 (extension) 声明

除此之外，**显式的指定派发方式**也会改变函数其原有的派发方式，例如添加 **final 或者 @objc 关键字等等**；以及**编译器对特定函数的优化**，例如将从未被重写的私有函数优化成静态派发。

下面我们就这四个方面来分析和探讨 Swift 的派发方式，以及证明其派发方式。



对象类型
--------------------

如上文所述，值类型，也就是 struct 的对象总是使用静态派发; class 对象使用函数表派发 (非 extension)。请看如下示例:

```
class MyClass {
    func testOfClass() {}
}

struct myStruct {
    func testOfStruct() {}
}
```

现在我们使用如下命令将 swift 代码转换为 SIL(中间码) 以便查看其函数派发方式:

```
swiftc -emit-silgen -O test.swift
```

输出结果如下:、

```
...
sil_vtable MyClass {
  #MyClass.testOfClass!1: (MyClass) -> () -> () : @$s4test7MyClassC0a2OfC0yyF	// MyClass.testOfClass()
  #MyClass.init!allocator.1: (MyClass.Type) -> () -> MyClass : @$s4test7MyClassCACycfC	// MyClass.__allocating_init()
  #MyClass.deinit!deallocator.1: @$s4test7MyClassCfD	// MyClass.__deallocating_deinit
}
```

首先 swift 会为 class 添加 **init** 和 **@objc deinit** 方法，为 struct 添加 **init** 方法。在文件的结尾处就会显示如上代码，它展示了哪些函数是函数表派发的，以及它们的标识符。由于 struct 类型仅使用静态派发，所以不会显示 sil_vtable 字样。



函数声明位置
--------------------------

函数声明位置的不同也会导致派发方式的不同。在 Swift 中，我们常常在 extension 里面添加扩展方法。根据我们之前总结的表格，通常 extension 中声明的函数都默认使用静态派发。

```
protocol MyProtocol {
    func testOfProtocol()
}

extension MyProtocol {
    func testOfProtocolInExtension() {}
}

class MyClass: MyProtocol {
    func testOfClass() {}
    func testOfProtocol() {}
}

extension MyClass {
    func testOfClassInExtension() {}
}
```

我们分别在 **protocol** 和 **class** 中声明一个函数，再在其 extension 中声明一个函数; 最后让类实现协议的一个方法，转换成 SIL 代码后如下:

```
...
sil_vtable MyClass {
  #MyClass.testOfClass!1: (MyClass) -> () -> () : @$s4test7MyClassC0a2OfC0yyF	// MyClass.testOfClass()
  #MyClass.testOfProtocol!1: (MyClass) -> () -> () : @$s4test7MyClassC0A10OfProtocolyyF	// MyClass.testOfProtocol()
  #MyClass.init!allocator.1: (MyClass.Type) -> () -> MyClass : @$s4test7MyClassCACycfC	// MyClass.__allocating_init()
  #MyClass.deinit!deallocator.1: @$s4test7MyClassCfD	// MyClass.__deallocating_deinit
}

sil_witness_table hidden MyClass: MyProtocol module test {
  method #MyProtocol.testOfProtocol!1: <Self where Self : MyProtocol> (Self) -> () -> () : @$s4test7MyClassCAA0B8ProtocolA2aDP0a2OfD0yyFTW	// protocol witness for MyProtocol.testOfProtocol() in conformance MyClass
}
```

我们可以很直观的看到，声明在协议或者类主体中的函数是使用函数表派发的; 而声明在扩展中的函数则是静态派发。

值得注意的是: 当我们在 protocol 中声明一个函数，并且在 protocol 的 extension 中实现了它，而且没有其他类型重写该函数，那么在这种情况下，该函数就是直接派发，算是通用函数。



指定派发方式
--------------------------

给函数添加关键字的修饰也能改变其派发方式。



### final

添加了 final 关键字的函数无法被重写，使用直接派发，不会在 vtable 中出现。并且对 Objc runtime 不可见。



### dynamic

值类型和引用类型的函数均可添加 dynamic 关键字。在 Swift5 中，给函数添加 **dynamic** 的作用是为了赋予非 objc 类和值类型 (struct 和 enum) 动态性。我们来看如下代码:

```
struct Test {
    dynamic func test() {}
}
```

我们赋予了 test 函数动态性。将其转换成 SIL 中间码后如下:

```
// Test.test()
sil hidden [dynamically_replacable] @$s4test4TestVAAyyF : $@convention(method) (Test) -> () {
// %0                                             // user: %1
bb0(%0 : @trivial $Test):
  debug_value %0 : $Test, let, name "self", argno 1 // id: %1
  %2 = tuple ()                                   // user: %3
  return %2 : $()                                 // id: %3
} // end sil function '$s4test4TestVAAyyF'
```

我们在第二行可以看到 **test** 函数多了一个 “属性”: **dynamically_replacable**, 也就是说添加 dynamic 关键字就是赋予函数**动态替换**的能力。那什么是动态替换呢? 简而言之就是提供一种途径，比方说，可以将 Module A 中定义的方法，在 Module B 中动态替换，如下所示:

```
// Module A
struct Foo {
 dynamic func bar() {}
}
```

```
// Module B
extension Foo {
  @_dynamicReplacement(for: bar()0
  func barReplacement() {
    ...
    // Calls previously active implementation of bar()
    bar()
  }
```

添加 dynamic 关键字并不代表对 Objc 可见。



### @objc

该关键字可以将 Swift 函数暴露给 Objc 运行时，但并不会改变其派发方式，依旧是函数表派发。举例如下:

```
class Test {
    @objc func test() {}
}
```

SIL 代码如下:

```
...

// @objc Test.test()
sil hidden [thunk] @$s4test4TestCAAyyFTo : $@convention(objc_method) (Test) -> () {
// %0                                             // user: %1
bb0(%0 : @unowned $Test):
  %1 = copy_value %0 : $Test                      // users: %6, %2
  %2 = begin_borrow %1 : $Test                    // users: %5, %4
  // function_ref Test.test()
  %3 = function_ref @$s4test4TestCAAyyF : $@convention(method) (@guaranteed Test) -> () // user: %4
  %4 = apply %3(%2) : $@convention(method) (@guaranteed Test) -> () // user: %7
  end_borrow %2 : $Test                           // id: %5
  destroy_value %1 : $Test                        // id: %6
  return %4 : $()                                 // id: %7
} // end sil function '$s4test4TestCAAyyFTo'

...

sil_vtable Test {
  #Test.test!1: (Test) -> () -> () : @$s4test4TestCAAyyF	// Test.test()
  #Test.init!allocator.1: (Test.Type) -> () -> Test : @$s4test4TestCACycfC	// Test.__allocating_init()
  #Test.deinit!deallocator.1: @$s4test4TestCfD	// Test.__deallocating_deinit
}
```

我们可以看到 test 方法依旧在 “虚函数列表” 中，证明其实函数表派发。如果希望 test 函数使用消息派发，则需要额外添加 **dynamic** 关键字。

### @inline or static

@inline 关键字顾名思义是想告诉编译器将此函数直接派发，但将其转换成 SIL 代码后，依旧是 vtable 派发。Static 关键字会将函数变为直接派发。



编译器优化
-----------------------

Swift 会尽可能的去优化函数派发方式。我们上文提到，当一个类声明了一个私有函数时，该函数很可能会被优化为直接派发。这也就是为什么当我们在 Swift 中使用 **target-action** 模式时，私有 selector 会报错的原因 (_Objective-C 无法获取 #selector 指定的函数_)。



派发总结
--------------------

最后我们用一张图总结下 Swift 中的派发方式:

<img src="https://sylarimage.oss-cn-shenzhen.aliyuncs.com/uPic/summary_c0azo8.png" style="zoom:50%;" />

从上表可见，我们在类型的主体中声明的函数大都是函数表派发，这也是 Swift 中最为常见的派发方式；而扩展大都是直接派发；只有再添加了特定关键字后，如 **@objc, final, dynamic** 后，函数派发方式才会有所改变。除此之外，编译器可能将某些方法优化为直接派发。例如私有函数。

讲了这么多函数派发的方式，那对我们有什么用呢？或者说如何选择派发方式呢？

我总结了两点:

1.  帮助我们理解一些 “奇怪的” 行为，例如为何 extension 中的函数无法被子类继承，为何需要添加 @objc 甚至是 dynamic 后才能被重写。
2.  提供选择类型的条件，例如您的 app 对性能要求很苛刻，那尽量使用值类型；并且对引用类型方法添加关键字描述。

总的来说，如何选择还是取决于业务类型。首先要确定使用引用类型还是值类型，因为它们也部分决定了函数的派发方式；之后确定是否给函数添加关键字，例如 **@objc，final 或 dynamic**，以达到准确描述该函数的目的。