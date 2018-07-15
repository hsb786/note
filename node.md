## Node.js
使用Node.js时，我们不仅仅在实现一个应用，同时还实现了整个HTTP服务器

**Node.js组成部分**
1. 引入required模块：我们可以使用require指令来载入Node.js模块
2. 创建服务器：服务器可以监听客户端的请求，类似于Apache、Nginx等HTTP服务器
3. 接受请求与相应请求：服务器很容易创建，客户端可以使用浏览器或终端发送HTTP请求，服务器接收请求后返回响应数据

### 创建Node.js应用
~~~
//1. 使用require指令来载入http模块
var http = require('http');

//2. 使用http.createServer()方法创建服务器，并使用listen方法绑定8888端口。
http.createServer(function (request, response) {

    // 发送 HTTP 头部 
    // HTTP 状态值: 200 : OK
    // 内容类型: text/plain
    response.writeHead(200, {'Content-Type': 'text/plain'});

    // 发送响应数据 "Hello World"
    response.end('Hello World\n');
}).listen(8888);
~~~
- - -

## NPM
包管理工具，常见的使用场景：
1. 允许用户从NPM服务器下载别人编写的第三方包到本地使用。 
2. 允许用户从NPM服务器下载并安装别人编写的命令行程序到本地使用。
3. 允许用户将自己编写的包或命令行程序上传到NPM服务器供别人使用。

**使用npm命令安装模块**
    npm install <Module Name>

**全局安装与本地安装**
从敲的命令行来看，差别只是有没有-g而已

**本地安装**
1. 将安装包放在./node_modules下（运行npm命令时所在的目录），如果没有node_modules目录，会在当前执行npm命令的目录下生成node_modules目录
2. 可以通过require()来引入本地安装的包

**全局安装**
1. 将安装包放在/usr/local下或者你node的安装目录
2. 可以直接在命令行里使用

**查看安装信息**
    npm list -g     查看所有全局安装的模块
    npm list express    查看某个模块的版本号

**package.json**
package.json 位于模块的目录下，用于定义包的属性。属性说明：
+ name - 包名
+ version - 包的版本号
+ description - 包的描述。
+ homepage - 包的官网 url 。
+ author - 包的作者姓名。
+ contributors - 包的其他贡献者姓名。
+ dependencies - 依赖包列表。如果依赖包没有安装，npm 会自动将依赖包安装在 node_module 目录下。
+ repository - 包代码存放的地方的类型，可以是 git 或 svn，git 可在 Github 上。
+ main - main 字段指定了程序的主入口文件，require('moduleName') 就会加载这个文件。这个字段的默认值是模块根目录下面的 index.js。
+ keywords - 关键字

**卸载模块**
    npm uninstall express

**更新模块**
    npm update express

**搜索模块**
    npm search express

### Node.js REPL(Read Eval Print Loop:交互式解释器)
类似Window系统的终端或Unix/Linux shell，我们可以在终端中输入命令，并接受系统的响应

Node自带了交互式解释器，可以执行以下任务：
+ 读取 - 读取用户输入，解析输入了Javascript 数据结构并存储在内存中。
+ 执行 - 执行输入的数据结构
+ 打印 - 输出结果
+ 循环 - 循环操作以上步骤直到用户两次按下 ctrl-c 按钮退出。

**下划线(_)变量**
使用下划线(_)获取上一个表达式的运算结果：
~~~
var x = 10
var y = 20
x + y
var sum = _
console.log(sum)            //30
~~~

**REPL命令**
+ ctrl d    退出Node REPL

### Node.js 回调函数
Node.js 异步编程的直接体现就是回调。  
异步编程依托于回调来实现，但不能说使用了回调后程序就异步化了。  
回调函数在完成任务后就会被调用，Node 使用了大量的回调函数，Node 所有 API 都支持回调函数。  
 例如，我们可以一边读取文件，一边执行其他命令，在文件读取完成后，我们将文件内容作为回调函数的参数返回。这样在执行代码时就没有阻塞或等待文件 I/O 操作。这就大大提高了 Node.js 的性能，可以处理大量的并发请求。  
 回调函数一般作为参数的最后一个参数出现： 
 ~~~
function foo1(name, age, callback) { }
function foo2(value, callback1, callback2) { }
 ~~~
 阻塞是按顺序执行的，而非阻塞是不需要按顺序的，所以如果需要处理回调函数的参数，我们就需要写在回调函数内

 ### Node.js 事件循环
 Node.js 是单进程单线程应用程序，但是因为 V8 引擎提供的异步执行回调接口，通过这些接口可以处理大量的并发，所以性能非常高。  
Node.js 几乎每一个 API 都是支持回调函数的。  
Node.js 基本上所有的事件机制都是用设计模式中观察者模式实现。  
Node.js 单线程类似进入一个while(true)的事件循环，直到没有事件观察者退出，每个异步事件都生成一个事件观察者，如果有事件发生就调用该回调函数.

### Node.js模块系统
为了让Node.js的文件可以相互调用，Node.js提供了一个简单的模块系统  
模块是Node.js应用程序的基本组成部分，文件和模块是一一对应的。换言之，一个Node.js文件就是一个模块，这个文件可能是JavaScript代码、JSON或者编译过的C/C++扩展

**创建模块**

hello.js
~~~
exports.world=function(){
    console.log('Hello');
}
~~~

main.js
~~~
//引入当前目录下的hello.js文件（./为当前目录，node.js默认后缀为js）
var hello = require('./hello');
hello.world();
~~~

Node.js提供了exports和require两个对象，其中exports是模块公开的接口，require用于从外部获取一个模块的接口，即所获取模块的exports对象

**把一个对象封装到模块中**
~~~
//hello.js 
function Hello() { 
    var name; 
    this.setName = function(thyName) { 
        name = thyName; 
    }; 
    this.sayHello = function() { 
        console.log('Hello ' + name); 
    }; 
}; 
module.exports = Hello;

//main.js 
var Hello = require('./hello'); 
hello = new Hello(); 
hello.setName('BYVoid'); 
hello.sayHello(); 
~~~

### Node.js函数
在JS中，一个函数可以作为另一个函数的参数。我们可以先定义一个函数，然后传递，也可以在传递函数的地方直接定义函数。

~~~
function say(word) {
  console.log(word);
}

function execute(someFunction, value) {
  someFunction(value);
}

execute(say, "Hello");
~~~
上述代码中，把say函数作为execute函数的第一个变量进行了传递。

**匿名函数**
我们可以直接在另一个函数的括号中定义和传递这个函数  
用这种方式，我们甚至不用给这个函数起名字，这也是为什么它被叫做匿名函数
~~~
function execute(someFunction, value) {
  someFunction(value);
}

execute(function(word){ console.log(word) }, "Hello");
~~~

### 全局对象
在浏览器js中，通常window是全局对象，而Node.js中的全局对象是global，所有全局变量（除了global本身以外）都是global对象的属性

**全局对象与全局变量**
 global 最根本的作用是作为全局变量的宿主。按照 ECMAScript 的定义，满足以下条 件的变量是全局变量：
 +  在最外层定义的变量
 + 全局对象的属性
 + 隐式定义的变量（未定义直接赋值的变量）

当你定义一个全局变量时，这个变量同时也会成为全局对象的属性，反之亦然。需要注 意的是，在 Node.js 中你不可能在最外层定义变量，因为所有用户代码都是属于当前模块的， 而模块本身不是最外层上下文。  
注意： 永远使用 var 定义变量以避免引入全局变量，因为全局变量会污染 命名空间，提高代码的耦合风险。

**__filename**
__filename 表示当前正在执行的脚本的文件名。它将输出文件所在位置的绝对路径

**__dirname**
 __dirname 表示当前执行脚本所在的目录。

**setTimeout(cb, ms)**
setTimeout(cb, ms) 全局函数在指定的毫秒(ms)数后执行指定函数(cb)。：setTimeout() 只执行一次指定函数。  
返回一个代表定时器的句柄值。

**clearTimeout(t)**
clearTimeout( t ) 全局函数用于停止一个之前通过 setTimeout() 创建的定时器。 参数 t 是通过 setTimeout() 函数创建的定时器。
~~~
function printHello(){
   console.log( "Hello, World!");
}
// 两秒后执行以上函数
var t = setTimeout(printHello, 2000);

// 清除定时器
clearTimeout(t);
~~~

**setInterval(cb, ms)**
setInterval(cb, ms) 全局函数在指定的毫秒(ms)数后执行指定函数(cb)。

返回一个代表定时器的句柄值。可以使用 clearInterval(t) 函数来清除定时器。

setInterval() 方法会不停地调用函数，直到 clearInterval() 被调用或窗口被关闭。

### 常用工具
 util 是一个Node.js 核心模块，提供常用函数的集合，用于弥补核心JavaScript 的功能 过于精简的不足。 

 **util.inherits**
 util.inherits(constructor, superConstructor)是一个实现对象间原型继承 的函数。  
JavaScript 的面向对象特性是基于原型的，与常见的基于类的不同。JavaScript 没有 提供对象继承的语言级别特性，而是通过原型复制来实现的
~~~
var util = require('util'); 
function Base() { 
    this.name = 'base'; 
    this.base = 1991; 
    this.sayHello = function() { 
    console.log('Hello ' + this.name); 
    }; 
} 
Base.prototype.showName = function() { 
    console.log(this.name);
}; 
function Sub() { 
    this.name = 'sub'; 
} 
util.inherits(Sub, Base); 
var objBase = new Base(); 
objBase.showName(); 
objBase.sayHello(); 
console.log(objBase); 
var objSub = new Sub(); 
objSub.showName(); 
//objSub.sayHello(); 
console.log(objSub);
~~~

我们定义了一个基础对象Base 和一个继承自Base 的Sub，Base 有三个在构造函数 内定义的属性和一个原型中定义的函数，通过util.inherits 实现继承。运行结果如下：
~~~
base 
Hello base 
{ name: 'base', base: 1991, sayHello: [Function] } 
sub 
{ name: 'sub' }
~~~ 
注意：Sub 仅仅继承了Base 在原型中定义的函数，而构造函数内部创造的 base 属 性和 sayHello 函数都没有被 Sub 继承。

**util.inspect**
 util.inspect(object,[showHidden],[depth],[colors])是一个将任意对象转换 为字符串的方法，通常用于调试和错误输出。它至少接受一个参数 object，即要转换的对象。   
showHidden 是一个可选参数，如果值为 true，将会输出更多隐藏信息。  
 depth 表示最大递归的层数，如果对象很复杂，你可以指定层数以控制输出信息的多 少。如果不指定depth，默认会递归2层，指定为 null 表示将不限递归层数完整遍历对象。 如果color 值为 true，输出格式将会以ANSI 颜色编码，通常用于在终端显示更漂亮 的效果。  
~~~
var util = require('util'); 
function Person() { 
    this.name = 'byvoid'; 
    this.toString = function() { 
    return this.name; 
    }; 
} 
var obj = new Person(); 
console.log(util.inspect(obj)); 
console.log(util.inspect(obj, true)); 
~~~

运行结果
~~~
Person { name: 'byvoid', toString: [Function] }
Person {
  name: 'byvoid',
  toString: 
   { [Function]
     [length]: 0,
     [name]: '',
     [arguments]: null,
     [caller]: null,
     [prototype]: { [constructor]: [Circular] } } }
~~~

## Express框架
Express 是一个简洁而灵活的 node.js Web应用框架, 提供了一系列强大特性帮助你创建各种 Web 应用，和丰富的 HTTP 工具。  
Express 框架核心特性：
+ 可以设置中间件来响应 HTTP 请求。
+ 定义了路由表用于执行不同的 HTTP 请求动作。
+ 可以通过向模板传递参数来动态渲染 HTML 页面。


