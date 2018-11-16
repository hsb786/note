#### servlet初始化参数

servlet初始化参数只能读一次，就是在容器初始化servlet的时候

1. 容器为这个servlet读取部署描述文件，包括servlet初始化参数（<init-param>）
2. 容器为这个servlet创建一个新的ServletConfig实例
3. 容器为每个servlet初始化 参数创建一个String名/值对
4. 容器向ServletConfig提供名/值初始化参数的引用
5. 容器创建servlet类的一个实例
6. 容器调用servlet的init()方法，传入ServletConfig的引用



> 每个servlet有一个ServletConfig；每个Web应用有一个ServletContext

