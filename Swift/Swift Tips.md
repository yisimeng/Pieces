# Swift Tips

**元组（Tuple）**的使用使用场景：方法有多个不同类型的返回值时，使用元组。
**nil：**只能复制给optional类型
**Extension：** 在extension中给系统的类扩充构造函数，只能扩充便利构造函数（关键字：convenience），参数有默认参数时，会自动生成两个函数（一个是完整参数函数，一个是使用的默认参数函数）。
**self.不能省略的情况**
> 1在方法中和其他的标识符有歧义    
> 2闭包中的‘self.’
> 3在属性懒加载中的’self.’

**required：**使用required关键字修饰初始化方法，表示指定的初始化方法，不能再使用‘（）’初始化

**protocol extension：** 可以where语法可以指定代理类型，不能当做delegate传值使用。
**@discardableResult：**在swift3.0之后，如果方法有返回值，但是没有用到的时候会报警告，可以使用关键字修饰去掉警告
