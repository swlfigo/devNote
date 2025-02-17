# Access Control



# Private

 private所修饰的属性或者方法只能在当前类里访问

 private所修饰类只能在当前.swift文件里访问



# fileprivate 

fileprivate访问级别所修饰的属性或者方法在当前的Swift源文件里可以访问。

# internal（默认访问级别，internal修饰符可写可不写）

internal访问级别所修饰的属性或方法在源代码所在的整个模块都可以访问、被继承、被重写

如果是框架或者库代码，则在整个框架内部都可以访问，框架由外部代码所引用时，则不可以访问。即使使用import，也会提示错误：

> No such module '...'

如果是App代码，也是在整个App代码，也是在整个App内部可以访问。

# public

可以被任何类访问。但其他module中不可以被override和继承，而在module内可以被override和继承。

> Cannot inherit from non-open class '...' outside of its defining module

# open

可以被任何类使用，包括override和继承。

访问权限排序从高到低排序：open>public>interal > fileprivate >private







Open 和 Public 级别可以让实体被同一模块源文件中的所有实体访问，在模块外也可以通过导入该模块来访问源文件里的所有实体。通常情况下，你会使用 Open 或 Public 级别来指定框架的外部接口。Open 和 Public 的区别在后面会提到。

Internal 级别让实体被同一模块源文件中的任何实体访问，但是不能被模块外的实体访问。通常情况下，如果某个接口只在应用程序或框架内部使用，就可以将其设置为 Internal 级别。

File-private 限制实体只能在其定义的文件内部访问。如果功能的部分细节只需要在文件内使用时，可以使用 File-private 来将其隐藏。

Private 限制实体只能在其定义的作用域，以及同一文件内的 extension 访问。如果功能的部分细节只需要在当前作用域内使用时，可以使用 Private 来将其隐藏。
 Open 只能作用于类和类的成员，它和 Public 的区别如下：

Public 或者其它更严访问级别的类，只能在其定义的模块内部被继承。

Public 或者其它更严访问级别的类成员，只能在其定义的模块内部的子类中重写。

Open 的类，可以在其定义的模块中被继承，也可以在引用它的模块中被继承。

Open 的类成员，可以在其定义的模块中子类中重写，也可以在引用它的模块中的子类重写。

