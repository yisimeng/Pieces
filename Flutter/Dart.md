# Dart

1. 单行函数的简写：

   ```dart
   void main() => runApp(new MyApp());
   ```

2. _suggestions 以下划线为前缀标识符，会强制其变为私有。

3. Dart没有析构函数，通过垃圾回收机制回收内存。

4. 构造函数中，如果参数在“{}”中，表示参数是可选的。

   ```dart
   class People {
     String firstName;
     // 可选参数
     String secondName;
     People(this.firstName, {this.secondName})
   }
   ```

5. 匿名构造函数：

   ```dart
   class People {
     String firstName;
     String secondName;
     // 匿名构造函数（类初始化方法？）
     People.fromList(List<dyamic> list){
       firstName = list[0];
       secondName = list[1];
     }
   }
   ```

8. 重定向构造函数：

   ```dart
   class People {
     String firstName;
     String secondName;
     People({this.firstName}, {this.secondName})
     //重定向构造函数
     People.defaults(String first, String second) : this(); 
   }
   ```

9. 字符串中访问外部变量 `'${_var} -- 前面是变量'` 或者`'$_var -- 前面是变量'`。
10. 变量声明：var、dynamic、Object、final、const
    * **var**：可接受任何类型的变量，一旦赋值，类型将确定，后期不可修改。
    * **Object**：Dart所有对象的根基类（包含Function和Null），赋值以后可以在后期改变赋值类型。 
    * **dynamic**：与Object类似。不同点在于dynamic声明的对象编译器会提供所有可能的组合，Object声明的对象只能使用Object的属性和方法。
    * **final**：赋值之后不可改变，运行时常量。
    * **const**：编译时常量。

