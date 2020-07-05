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

9. await：之后的操作必须是异步，且当前函数必须是异步函数。 

   async方法中，Future以及Future的then函数里的方法是异步，其他仍然为同步。

10. **then** 之后的方法优先级比 Feature 默认的队列优先级高。

11. isolate：隔离，Dart中的多线程，更像是一个进程，有独立内存控件，不需要担心多线程资源抢夺，不需要锁，使用port进行通信。compute方法就是封装的 isolate。



## Dart 异步

### Dart 的事件循环

在Dart 中有两种队列：

1. 事件队列（Event Queue），包含所有的外来事件：I/O，mouse events，drawing events，timers，isolate之间的信息传递。
2. 微任务队列（Microtask Queue）表示一个在极短时间内会完成的异步任务。优先级最高，高于Event Queue，只要队列中有任务，就可以一直霸占时间循环。MicroTast Queue添加的任务主要是由Dart内部产生。

> Future是事件队列

