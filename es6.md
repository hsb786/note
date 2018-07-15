**ECMAScript和JavaScript的关系**
JavaScript的创造者将JavaScript提交给标准化组织ECMA，希望这种语言能够成为国际标准。ECMA发布262号标准文件(ECMA-262)的第一版，规定了浏览器脚本语言的标准，并将这种语言称为ECMAScript。  
前者是后者的规格，后者是前者的一种实现

## let命令
用来声明变量，用法类似于var
+ 只在let命令所在的代码块内有效
+ 变量在声明语句之后才可以使用
+ 只要块级作用域内存在let命令，它所声明的变量就“绑定”(binding)这个区域，不再受外部的影响
+ 不允许在相同作用域内，重复声明同一个变量


**变量提升**
变量可以在声明之前使用，值为undefined

**暂时性死区**
ES6 明确规定，如果区块中存在let和const命令，这个区块对这些命令声明的变量，从一开始就形成了封闭作用域。凡是在声明之前就使用这些变量，就会报错。

总之，在代码块内，使用let命令声明变量之前，该变量都是不可用的。这在语法上，称为“暂时性死区”（temporal dead zone，简称 TDZ）。
~~~
if (true) {
  // TDZ开始
  tmp = 'abc'; // ReferenceError
  console.log(tmp); // ReferenceError

  let tmp; // TDZ结束
  console.log(tmp); // undefined

  tmp = 123;
  console.log(tmp); // 123
}

~~~

## const命令
const声明一个只读的常量，一旦声明变量，就必须立即初始化

**顶层对象的属性**
顶层对象，在浏览器环境指的是window对象，在 Node 指的是global对象。ES5 之中，顶层对象的属性与全局变量是等价的。
~~~
window.a = 1;
a // 1

a = 2;
window.a // 2
~~~

ES6 规定，为了保持兼容性，var命令和function命令声明的全局变量，依旧是顶层对象的属性；另一方面规定，let命令、const命令、class命令声明的全局变量，不属于顶层对象的属性。也就是说，从 ES6 开始，全局变量将逐步与顶层对象的属性脱钩。
~~~
var a = 1;
// 如果在 Node 的 REPL 环境，可以写成 global.a
// 或者采用通用方法，写成 this.a
window.a // 1

let b = 1;
window.b // undefined
~~~

## 变量的解构赋值
### 数组的解构赋值
ES6 允许按照一定模式，从数组和对象中提取值，对变量进行赋值，这被称为解构（Destructuring）。
    let [a, b, c] = [1, 2, 3];

**默认值**
~~~
let [foo = true] = [];
foo // true

let [x, y = 'b'] = ['a']; // x='a', y='b'
let [x, y = 'b'] = ['a', undefined]; // x='a', y='b'
~~~
注意，ES6 内部使用严格相等运算符（===），判断一个位置是否有值。所以，只有当一个数组成员严格等于undefined，默认值才会生效。

如果默认值是一个表达式，那么这个表达式是惰性求值的，即只有在用到的时候，才会求值。

### 对象的解构赋值 
对象的解构与数组有一个重要的不同。数组的元素是按次序排列的，变量的取值由它的位置决定；而对象的属性没有次序，变量必须与属性同名，才能取到正确的值。
~~~
let { bar, foo } = { foo: "aaa", bar: "bbb" };
foo // "aaa"
bar // "bbb"

let { baz } = { foo: "aaa", bar: "bbb" };
baz // undefined
~~~

如果变量名与属性名不一致，必须写成下面这样。
~~~
let obj = { first: 'hello', last: 'world' };
let { first: f, last: l } = obj;
f // 'hello'
l // 'world'
~~~

### 字符串的解构赋值 
~~~
const [a, b, c, d, e] = 'hello';
a // "h"
b // "e"
c // "l"
d // "l"
e // "o"
~~~

类似数组的对象都有一个length属性，因此还可以对这个属性解构赋值。
~~~
let {length : len} = 'hello';
len // 5
~~~
解构赋值的规则是，只要等号右边的值不是对象或数组，就先将其转为对象。

## 函数的扩展
###与解构赋值默认值结合使用
~~~
function foo({x, y = 5}) {
  console.log(x, y);
}

foo({}) // undefined 5
foo({x: 1}) // 1 5
foo({x: 1, y: 2}) // 1 2
foo() // TypeError: Cannot read property 'x' of undefined
~~~
上面代码只使用了对象的解构赋值默认值，没有使用函数参数的默认值。只有当函数foo的参数是一个对象时，变量x和y才会通过解构赋值生成。如果函数foo调用时没提供参数，变量x和y就不会生成，从而报错。通过提供函数参数的默认值，就可以避免这种情况。
~~~
function foo({x, y = 5} = {}) {
  console.log(x, y);
}

foo() // undefined 5
~~~
上面代码指定，如果没有提供参数，函数foo的参数默认为一个空对象。

## Module的语法
ES6模块的设计思想是尽量的静态化，使得编译时就能确定模块的依赖关系，以及输入和输出的变量  
通过export命令显式指定输出的代码，再通过import命令输入
~~~
// ES6模块
import { stat, exists, readFile } from 'fs';
~~~
上面代码的实质是从fs模块加载 3 个方法，其他方法不加载。这种加载称为“编译时加载”或者静态加载，即 ES6 可以在编译时就完成模块加载

ES6 模块之中，顶层的this指向undefined，即不应该在顶层代码使用this。