# Dart

1. 单行函数的简写：

   ```dart
   void main() => runApp(new MyApp());
   ```

2. _suggestions 以下划线为前缀标识符，会强制其变为私有。

3. Dart没有析构函数，通过垃圾回收机制回收内存。

4. final修饰，赋值之后不可改变，运行时常量。

5. const修饰构造函数，节约性能，参数必须是final。编译时常量。

6. 构造函数中，如果参数在“{}”中，表示参数是可选的。

   ```dart
   class People {
     String firstName;
     // 可选参数
     String secondName;
     People(this.firstName, {this.secondName})
   }
   ```

7. 匿名构造函数：

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

9. 字符串中访问外部变量 `'${_var} -- 前面是变量'`

