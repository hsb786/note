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


### rest参数
ES6 引入 rest 参数（形式为...变量名），用于获取函数的多余参数，这样就不需要使用arguments对象了。rest 参数搭配的变量是一个数组，该变量将多余的参数放入数组中。
~~~
function add(...values) {
  let sum = 0;

  for (var val of values) {
    sum += val;
  }

  return sum;
}

add(2, 5, 3) // 10
~~~

**箭头函数体内的`this`对象，就是定义时所在的对象，而不是使用时所在的对象**
~~~
function Timer() {
  this.s1 = 0;
  this.s2 = 0;
  // 箭头函数     //绑定定义时所在的作用域（即Timer函数）
  setInterval(() => this.s1++, 1000);
  // 普通函数     //指向运行时所在的作用域（即全局对象）
  setInterval(function () {
    this.s2++;
  }, 1000);
}

var timer = new Timer();

setTimeout(() => console.log('s1: ', timer.s1), 3100);
setTimeout(() => console.log('s2: ', timer.s2), 3100);
// s1: 3
// s2: 0
~~~

`this`指向的固定化，并不是因为箭头函数内部有绑定`this`机制，实际原因是箭头函数根本没有自己的`this`，导致内部的`this`就是外层代码块的`this`。正是因为它没有`this`，所以也就不能用作构造函数


### Object.is()
相等运算符(`==`)和严格相等运算符(`===`)。它们都有缺点，前者会自动转换数据类型，后者的`NaN`不等于自身，以及`+0`等于`-0`。

ES6 提出“Same-value equality”（同值相等）算法，用来解决这个问题。Object.is就是部署这个算法的新方法。它用来比较两个值是否严格相等，与严格比较运算符（===）的行为基本一致。

~~~
Object.is('foo', 'foo')
// true
Object.is({}, {})
// false
~~~
不同之处只有两个：一是+0不等于-0，二是NaN等于自身。
~~~
+0 === -0 //true
NaN === NaN // false

Object.is(+0, -0) // false
Object.is(NaN, NaN) // true
~~~

### Object.assign()
`Object.assign`方法用于对象的合并，将源对象（source）的所有可枚举属性，复制到目标对象（target）。  
注意，如果目标对象与源对象有同名属性，或多个源对象有同名属性，则后面的属性会覆盖前面的属性。
~~~
const target = { a: 1, b: 1 };

const source1 = { b: 2, c: 2 };
const source2 = { c: 3 };

Object.assign(target, source1, source2);
target // {a:1, b:2, c:3}
~~~

- - -
## Promise
Promise是异步编程的一种解决方案，比传统的解决方案--回调函数和事件--更合理和更强大。

所谓`Promise`，简单说就是一个容器，里面保存着某个未来才会结束的事件（通常是一个异步操作）的结果。从语法上说，Promise 是一个对象，从它可以获取异步操作的消息。Promise 提供统一的 API，各种异步操作都可以用同样的方法进行处理。

`Promise`对象有以下两个特点
1. 对象的状态不受外界影响。`Promise`对象代表一个异步操作，有三种状态:`pending`(进行中)、`fulfilled`(已成功)和`rejected`(已失败)。只有异步操作的结果，可以决定当前是哪一种状态，任何其它操作都无法改变这个状态
2. 一旦状态改变，就不会再变，任何时候都可以得到这个结果


`Promise`对象是一个构造函数，用来生成`Promise`实例
~~~
const promise = new Promise(function(resolve, reject) {
  // ... some code

  if (/* 异步操作成功 */){
    resolve(value);
  } else {
    reject(error);
  }
});
~~~
`resolve`函数的作用是，将`Promise`对象的状态从“未完成”变为“成功”（即从 pending 变为 resolved），在异步操作成功时调用，并将异步操作的结果，作为参数传递出去

`Promise`实例生成以后，可以用then方法分别制定`resolved`状态和`rejected`状态的回调函数
~~~
promise.then(function(value){
  //success
},function(error){
  //false
});
~~~

### Promise.prototype.then()
Promise实例具有`then`方法，也就是说，`then`方法是定义在原型对象`Promise.prototype`上的。它的作用是为Promise实例添加状态改变时的回调函数。`then`方法的第一个参数是`resolved`状态的回调函数，第二个参数（可选）是`rejected`状态的回调函数

`then`方法返回的是一个新的`Promise`实例（注意，不是原来那个`Promise`实例）。因此可以采用链式写法，即`then`方法后面再调用另一个`then`方法。

- - -


## async
`async`函数返回一个Promise对象，可以使用`then`方法添加回调函数。当函数执行的时候，一旦遇到`await`就会先返回，等到异步操作完成，再接着执行函数体内后面的语句
~~~
async function getStockPriceByName(name) {
  const symbol = await getStockSymbol(name);
  const stockPrice = await getStockPrice(symbol);
  return stockPrice;
}

getStockPriceByName('goog').then(function (result) {
  console.log(result);
});
~~~

`async`函数内部`return`语句返回的值，会成为`then`方法回调函数的参数

### Promise对象的状态变化
`async`函数返回的Promise对象，必须等到内部所有`await`命令后面的Promise对象执行完，才会发生状态改变，除非遇到`return`语句或者抛出错误。也就是说，只有`async`函数内部的异步操作执行完，才会执行`then`方法指定的回调函数

### await命令
正常情况下，`await`命令后面是一个Promise对象。如果不是，会被转成一个立即`resolve`的Promise对象

### 使用注意点
1. `await`命令后面的`Promise`对象，运行结果可能是`rejected`，所以最好把`await`命令放在`try...catch`代码块中。
~~~
async function myFunction() {
  try {
    await somethingThatReturnsAPromise();
  } catch (err) {
    console.log(err);
  }
}

// 另一种写法

async function myFunction() {
  await somethingThatReturnsAPromise()
  .catch(function (err) {
    console.log(err);
  });
}
~~~

2. 多个await命令后面的异步操作，如果不存在继发关系，最好让它们同时触发
3. `await`命令只能用在`async`函数之中，如果用在普通函数，就会报错


## Class
~~~
//定义类
class Point {
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }

  toString() {
    return '(' + this.x + ', ' + this.y + ')';
  }
}
~~~

ES6的类，完全可以看作构造函数的另一种写法，类的数据类型就是函数，类本身就指向构造函数
~~~
class Point {
  // ...
}

typeof Point // "function"
Point === Point.prototype.constructor // true
~~~

构造函数的`prototype`属性，在 ES6 的“类”上面继续存在。事实上，类的所有方法都定义在类的`prototype`属性上面。
~~~
class Point {
  constructor() {
    // ...
  }

  toString() {
    // ...
  }

  toValue() {
    // ...
  }
}

// 等同于

Point.prototype = {
  constructor() {},
  toString() {},
  toValue() {},
};
~~~

在类的实例上面调用方法，其实就是调用原型上的方法。
~~~
class B {}
let b = new B();

b.constructor === B.prototype.constructor // true
~~~


~~~
//定义类
class Point {

  constructor(x, y) {
    this.x = x;
    this.y = y;
  }

  toString() {
    return '(' + this.x + ', ' + this.y + ')';
  }

}

var point = new Point(2, 3);

point.toString() // (2, 3)

point.hasOwnProperty('x') // true
point.hasOwnProperty('y') // true
point.hasOwnProperty('toString') // false
point.__proto__.hasOwnProperty('toString') // true
~~~
`x`和`y`都是实例对象`point`自身的属性（因为定义在`this`变量上），所以`hasOwnProperty`方法返回`true`，而`toString`是原型对象的属性（因为定义在`Point`类上）

~~~
var p1 = new Point(2,3);
var p2 = new Point(3,2);

p1.__proto__ === p2.__proto__
~~~
上面代码中，p1和p2都是Point的实例，它们的原型都是Point.prototype，所以__proto__属性是相等的。
>__proto__ 并不是语言本身的特性，这是各大厂商具体实现时添加的私有属性，虽然目前很多现代浏览器的 JS 引擎中都提供了这个私有属性，但依旧不建议在生产中使用该属性，避免对环境产生依赖。生产环境中，我们可以使用 Object.getPrototypeOf 方法来获取实例对象的原型，然后再来为原型添加方法/属性。

~~~
var p1 = new Point(2,3);
var p2 = new Point(3,2);

p1.__proto__.printName = function () { return 'Oops' };

p1.printName() // "Oops"
p2.printName() // "Oops"

var p3 = new Point(4,2);
p3.printName() // "Oops"
~~~

## Module的语法
ES6模块的设计思想是尽量的静态化，使得编译时就能确定模块的依赖关系，以及输入和输出的变量  
通过export命令显式指定输出的代码，再通过import命令输入
~~~
// ES6模块
import { stat, exists, readFile } from 'fs';
~~~
上面代码的实质是从fs模块加载 3 个方法，其他方法不加载。这种加载称为“编译时加载”或者静态加载，即 ES6 可以在编译时就完成模块加载

ES6 模块之中，顶层的this指向undefined，即不应该在顶层代码使用this。

export输出的变量就是本来的名字，但是可以使用as关键字重命名。
~~~
function v1() { ... }
function v2() { ... }

export {
  v1 as streamV1,
  v2 as streamV2,
  v2 as streamLatestVersion
};
~~~

`export`命令规定的是对外的接口，必须与模块内部的变量建立一一对应的关系
~~~
// 报错
export 1;

// 报错
var m = 1;
export m;
~~~
上面两种写法都会报错，因为没有提供对外的接口，都只是直接输出1

~~~
// 写法一
export var m = 1;

// 写法二
var m = 1;
export {m};

// 写法三
var n = 1;
export {n as m};
~~~
上面三种写法都是正确的，规定了对外的接口m。其他脚本可以通过这个接口，取到值1。它们的实质是，在接口名与模块内部变量之间，建立了一一对应的关系。

### export default
为模块指定默认输出，其它模块加载该模块时，`import`命令可以为该匿名函数指定任意名字


## 编程风格

### 解构赋值
~~~
const arr=[1,2,3,4];
const[first,second]=arr; 
~~~

~~~
// bad
function getFullName(user) {
  const firstName = user.firstName;
  const lastName = user.lastName;
}

// good
function getFullName(obj) {
  const { firstName, lastName } = obj;
}

// best
function getFullName({ firstName, lastName }) {
}
~~~

~~~
// good
function processInput(input) {
  return { left, right, top, bottom };
}

const { left, right } = processInput(input);
~~~

### 对象
单行定义的对象，最后一个成员不以逗号结尾。多行定义的对象，最后一个成员以逗号结尾。
~~~
// good
const a = { k1: v1, k2: v2 };
const b = {
  k1: v1,
  k2: v2,
};
~~~

对象尽量静态化，一旦定义，就不得随意添加新的属性。如果添加属性不可避免，要使用Object.assign方法。
~~~
// if reshape unavoidable
const a = {};
Object.assign(a, { x: 3 });

// good
const a = { x: null };
a.x = 3;
~~~

### 数组
使用扩展运算符(...)拷贝数组
  const itemsCopy=[...items];

所有配置项都应该集中在一个对象，放在最后一个参数，布尔值不可以直接作为参数。
~~~
function divide(a, b, { option = false } = {}) {
}
~~~

### 模板字符串
反引号（`）标识。可以当作普通字符串使用，也可以用来定义多行字符串，或者在字符串中嵌入变量


>prop in object

`prop` 一个字符串类型或者symbol类型的属性名或者数组索引