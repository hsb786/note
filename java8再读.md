设计匿名内部类的目的，就是为了方便Java程序员将代码作为数据传递



> Lambda表达式中引用既成事实上的final变量

~~~
String name="a";
// 编译报错，因为lambda表达式中引用了name，就相当于在第一条语句中name已经被定义为final了，再给name赋值就会编译报错
name="b";
Runnable runnable=()->system.out.println(name);
~~~

