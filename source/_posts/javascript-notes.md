---
title: 你需要知道的 javascript 的细节
date: 2018-04-02 17:27:29
tags: [javascript]
---

### 对象

#### 变量可以当对象使用

javascript 中所有的变量都可以当做对象使用，除了undefined 和 null ，我们测试下

```
false.toString() // "false"

[1,2,3].toString() //"1,2,3"

1..toString() //"1"

({a:'33'}).toString() //"[object Object]"
```

```
undefined.toString() //Uncaught TypeError

null.toString()   //Uncaught TypeError
```



数值和对象虽然能调用 toString 方法，但是在写法上需要注意下

number调用时不能直接数值后面直接调用toString 方法，因为 js 会将点运算符解析为数值的小数点

```
1.toString() //Uncaught SyntaxError

1..toString() //"1"
```



对象直接调用toString 方法时，需要用小括号包裹起来，不然js 会将对象的花括号识别成块，从而报错

```
{a:'33'}.toString()  // Uncaught SyntaxError

({a:'33'}).toString() // "[object Object]"
```

#### 对象删除属性

> 删除对象的属性唯一的方法是使用 delete 操作符，设置元素属性为 undefined 或则 null 并不能真正删除，只是移除了属性和值的关联

```
var test = {
	name:'bbt',
	age:'18',
	love:'dog'
}

test.name = undefined
test.age = null
delete test.love

for (var i in test){
  console.log(i+':'+test[i])
}

```

运行结果

```
name:undefined
age:null
undefined
```

只有 love 被正则删除，name 和 age 还是能被遍历到

#### 构造函数

> 在 javascript 中，通过关键字 new 调用的函数就被认为是构造函数，我们可以通过构造函数创建对象实例

但是在使用过程中你一定发现了，每实例化一个对象，都会在实例对象上创造构造函数的方法和属性。倘若创建的实例比较多，重复创建同一个方法去开辟内存空间就会显得十分浪费，我们可以通过把被经常复用的方法放在原型链上。

#### 原型继承

> javascript 和一些我们所了解的面向对象编程的语言不太一样，在 es6 语法以前，我们是通过原型链来实现方法和属性的继承

```
function Child(){
  this.name = 'bbt'
}

Child.prototype = {
	title:'baba',
    method: function() {}
};

function Grandson(){}

//设置 Grandson 的 prototype 为 Child 的实例
Grandson.prototype = new Child()

//为 Grandson 的原型添加添加属性 age
Grandson.prototype.age = 40

// 修正 Grandson.prototype.constructor 为 Grandson 本身
Grandson.prototype.constructor = Grandson;

var xiaomin = new Grandson()

//原型链如下
xiaomin // Grandson的实例
    Grandson.prototype // Child的实例
         Grandson.prototype //{title:'baba',...}
            Object.prototype
                {toString: ... /* etc. */};
```

对象的属性查找，javascript 会在原型链上向上查找属性，直到查到 原型链顶部，所以，属性在原型链的越上端，查找的时间会越长，查找性能和复用属性方面需要开发者自己衡量下。

#### 获取自身对象属性

hasOwnProperty 方法能够判断一个对象是否包含自定义属性，而不是在原型链上的属性

```
var test = {hello:'123'}

Object.prototype.name = 'bbt'

test.name  //'bbt'
test.hasOwnProperty('hello') //true
test.hasOwnProperty('name') //false
```

for in 循环可以遍历对象原型链上的所有属性，如此我们将 hasOwnProperty 结合循环for in 能够获取到对象自定义属性

```
var test = {hello:'222'}
Object.prototype.name = 'bbt'

for(var i in test){
  console.log(i) // 输出两个属性，hello ，name
}


for(var i in test){
  if(test.hasOwnProperty(i)){
    console.log(i)//只输出 hello
  }
}
```

除了上面的方法，getOwnPropertyNames 和 Object.keys 方法，能够返回对象自身的所有属性名，也是接受一个对象作为参数，返回一个数组，包含了该对象自身的所有属性名。

```
var test = {hello:'222'}
Object.prototype.name = 'bbt'

Object.keys(test) //["hello"]
Object.getOwnPropertyNames(test) //["hello"]
```

那 getOwnPropertyNames 和 Object.keys 的用法有什么区别呢

Object.keys方法只返回可枚举的属性，Object.getOwnPropertyNames  方法还返回不可枚举的属性名。

```
var a = ['Hello', 'World'];

Object.keys(a) // ["0", "1"]

Object.getOwnPropertyNames(a) // ["0", "1", "length"]  // length 是不可枚举属性
```



### 函数

#### 函数声明的变量提升

我们通常会使用函数声明或函数赋值表达式来定义一个函数，函数声明和变量声明一样都存在提升的情况，函数可以在声明前调用，但是不可以在赋值前调用

函数声明

```
foo(); // 正常运行，因为foo在代码运行前已经被创建
function foo() {}
```

函数表达式

```
foo; // 'undefined'
foo(); // 出错：TypeError
var foo = function() {};
```

变量提升是在代码解析的时候进行的，foo() 方法调用的时候，已经在解析阶段将 foo 定义过了。赋值语句只在代码运行时才进行，所以在赋值前调用会报错

一种比较少用的函数赋值操作，将命名函数赋值给一个变量，此时的函数名只对函数内部可见

```
var test = function foo(){
  console.log(foo) //正常输出
}

console.log(foo) //Uncaught ReferenceError
```

#### this 的工作原理

> 在 javascript 中 ，this 是一个比较难理解的点，不同的调用环境会导致 this 的不同指向，但是唯一不变的是 this 总是指向一个对象

简单的说，this 就是属性和方法当前所在的对象（函数执行坐在的作用域），平时使用的 this  的情况可以大致分为5种

| 调用方式                                 | 指向             |
| ------------------------------------ | -------------- |
| 1. 全局范围调用                            | 指向 window 全局对象 |
| 2. 函数调用                              | 指向 window 全局变量 |
| 3. 对象的方法调用                           | 指向方法调用的对象      |
| 4. 构造函数调用                            | 指向构造函数创建的实例    |
| 5. 通过，call ，apply ，bind 显示的指定 this指向 | 和传参有关          |



Function.call

> 语法：function.call(thisArg, arg1, arg2, …)，thisArg 表示希望函数被调用的作用域，arg1, arg2, …表示希望被传入函数额参数 , 如果参数为空、`null`和`undefined`，则默认传入全局对象。

代码示例

```
var name = 'xiaomin'
var test = {name : 'bbt'}

function hello( _name ){
  _name ?console.log(this.name,_name): console.log(this.name)
}

hello() //xiaomin
hello.call(test) //bbt
hello.call(test,'xiaohong') //bbt xiaohong
hello.call() //xiaomin
hello.call(null) //xiaomin
hello.call(undefined) //xiaomin

```

Function.apply

> 语法和call 方法类似，不同的是，传入调用函数的参数变成以数组的形式传入，即 func.apply(thisArg, [argsArray])

改造上面的示例就是

```
hello.apply(test,['xiaomin'])
```

Function.bind

> `bind`方法用于将函数体内的`this`绑定到某个对象，然后返回一个新函数。

```
var d = new Date();
d.getTime()

var print = d.getTime; //赋值后 getTime 已经不指向 d 实例
print() // Uncaught TypeError
```

解决方法

```
var print = d.getTime.bind(d)
```

容易出错的地方

容易出错的地方，函数调用，this 总是指向 window 全局变量，所以在对象的方法里如果有函数的调用的话（闭包的情况），this 是会指向 全局对象的，不会指向调用的对象，具体示例如下

```
var name = 'xiaomin'
var test = {
  name : 'bbt'
}
test.method = function(){
  function hello(){
      console.log(this.name)
    }
    hello()
}

// 调用
test.method() // 输出 xiaomin
```

如果需要将 this 指向调用的对象，可以将对象的 this 指向存储起来，通常我们使用 that 变量来做这个存储。改进之后的代码

```
var name = 'xiaomin'
var test = {
  name : 'bbt'
}
test.method = function(){
  var that = this
  function hello(){
      console.log(that.name)
    }
    hello()
}

// 调用
test.method() // 输出 bbt
```

####闭包和引用

> 闭包我们可以理解成是在函数内部定义的函数

在 javascript 中，内部作用域可以访问到外部作用域的变量，但是外部作用域不能访问内部作用域，需要访问的时候，我们需要通过创建闭包，来操作内部变量

```
function test(_count){
  var count = _count

  return {
    inc:function(){
      count++
    },
    get:function(){
      return count
    }
  }
}

var a = test(4)
a.get()//4
a.inc()
a.get()//5
```

闭包中常会出错的面试题

```
for(var i = 0; i < 10; i++) {
    setTimeout(function() {
        console.log(i);
    }, 0);
}
```

很多同学会觉得，上面的代码会正常输出0到9，但是实际是输出十次10。遇到这个题目，除了闭包的概念要理解清楚，你还需要知道，setTimeout 内的代码会被异步执行，代码会先执行所有的同步代码，即上面的这段代码会先将 for 循环执行，此时 i 的值为 10，console.log(i) 一直引用着全局变量的 i  所以会输出十次 10

 改进代码，我们在 for 循环里创建一个闭包，把循环自增的 i 作为参数传入

```
for(var i = 0; i < 10; i++) {
    (function(e) {
        setTimeout(function() {
            console.log(e);
        }, 1000);
    })(i);
}
```

#### setTimeout && setInterval

> javascript 是异步的单线程运行语言，其他代码运行的时候可能会阻塞 setTimeout && setInterval 的运行

```
console.log(1)
setTimeout(function(){
  console.log(2)
}, 0);
console.log(3)

输出结果： 1，3，2  //setTimeout 被阻塞
```

处理阻塞的方法是将setTimeout 和 setInterval放在回调函数里执行

```
function test(){
  	setTimeout(function(){
  		console.log(2)
	}, 0);
}
```

setTimeout 和 setInterval 被调用时会返回一个 ID 用来清除定时器

手工清除某个定时器

```
var id = setTimeout(foo, 1000);
clearTimeout(id);
```

清楚所有的定时器

```
var lastId = setTimeout(function(){
  console.log('11')
}, 0);

for(var i=0;i<lastId;i++;){
  clearTimeout(i);
}

```

获取最后一个定时器的id，遍历清除定时器，可以清除所有的定时器。

### 类型

####包装对象

> 数值、字符串、布尔值——在一定条件下，也会自动转为对象，也就是原始类型的“包装对象”。

我们可以通过构造函数，将原始类型转化为对应的对象即包装对象，从而是原始类型能够方便的调用某些方法

数值，字符串，布尔值的类型转换函数分别是 Number，String，Boolean，在调用的时候在函数前面加上New 就变成了构造函数，能够蒋对应的原始类型转化为“包装对象”

```
var v1 = new Number(123);
var v2 = new String('abc');
var v3 = new Boolean(true);

typeof v1 // "object"
typeof v2 // "object"
typeof v3 // "object"

v1 === 123 // false
v2 === 'abc' // false
v3 === true // false
```

#### 类型转换

类型转换分为强制类型转换和自动转换，javascript 是动态类型语言，在到吗解析运行时，需要的数据类型和传入的数据类型不一致的时候，javascript 会进行自动类型转化。当然，你也可以通过类型转换方法进行强制类型装换。

日常开发中，我们最常用的数据类型自动转换不过就下面三种情况

不同数据类型之间相互运算

```
'2'+4 // '24'
```

对非布尔值进行布尔运算

```
if('22'){
  console.log('hello')
}
```

对非数据类型使用一元运算符

```
+'12'  //12
```

我们也通过 Number ，String，Boolean 来进行强制数据类型转换。强制类型转化的规则有点复杂，我们来了解一下。

Number 转换  [引用阮老师的详细解释](http://javascript.ruanyifeng.com/grammar/conversion.html)

```
第一步，调用对象自身的valueOf方法。如果返回原始类型的值，则直接对该值使用Number函数，不再进行后续步骤。

第二步，如果valueOf方法返回的还是对象，则改为调用对象自身的toString方法。如果toString方法返回原始类型的值，则对该值使用Number函数，不再进行后续步骤。

第三步，如果toString方法返回的是对象，就报错。
```

String 转换方法同样也是通过调用原对象的 toString 方法和 valueOf 方法，但是不同的是 String 函数会先调用 toString 方法进行转换

Boolean 的转换规则会相对简单一些，除了几个特殊的值，都会被转化为 true

```
undefined
null
+0或-0
NaN
''（空字符串）
```

但是要注意

```
Boolean('false') //true
```

#### typeof

> typeof 操作符返回数据类型，但是由于 javascript 设计的历史原因，typeof 现已经不能满足我们现在对于类型判断的要求了


| Value                            | Class            |Type       |
| ------------------------------------ | -------------- |-------------- |
|"foo"               |String     |string|
|new String("foo")   |String     |object|
|1.2                 |Number     |number|
|new Number(1.2)     |Number     |object|
|true                |Boolean    |boolean|
|new Boolean(true)   |Boolean    |object|
|new Date()          |Date       |object|
|new Error()         |Error      |object|
|[1,2,3]             |Array      |object|
|new Array(1, 2, 3)  |Array      |object|
|new Function("")    |Function   |functio|
|/abc/g              |RegExp     |object (function in Nitro/V8)|
|new RegExp("meow")  |RegExp     |object (function in Nitro/V8)|
|{}                  |Object     |object|
|new Object()        |Object     |object|
|null                |null       |object|

我们可以看到，typeof 不能区分对象的数组和日期，还会把 null 判断成对象，那我们一般是什么时候用 typeof 呢。我们可以用来判断一个已经定义的变量是否被赋值。

```
var a
if(typeof a == 'undefined'){
  console.log('a 已经被定义')
}
```

#### instanceof

> instanceof 操作符通常用来判断，一个对象是否在另一个对象的原型链上，需要注意的是 instanceof的左值是对象，右值是构造函数

```
// defining constructors
function C() {}
function D() {}

var o = new C();

// true, because: Object.getPrototypeOf(o) === C.prototype
o instanceof C;

// false, because D.prototype is nowhere in o's prototype chain
o instanceof D;
```

 #### Object.prototype.toString

> 那么我们有没有可以用来区分变量数据类型的方法呢，有，Object.prototype.toString

一些原始数据类型也有 toString 方法，但是通常他们的 toString 方法都是改造过的，不能进行 数据类型判断，所以我们需要用 Object 原型链上的 toString 方法

```
var a = 1234
a.toString() // '1234'

Object.prototype.toString.call(a) // "[object Number]"
```

 不同类型返回的结果如下：

```

 1. 数值 [object Number]
 2. 字符串 [object String]
 3.布尔值 [object Boolean]
 4.undefined [object undefined]
 5.null  [object Null]
 6.数组 [object Array]
 7.arguments [object Arguments]
 8.函数 [object function]
 9.Error [object Error]
 10.Date [object Date]
 11.RegExp [object RegExp]
 12.其他对象 [object object]

```

那么我们就能够通过 Object.prototype.toString 方法，封装一个可以判断变量数据类型的函数了

```
function type(obj) {
    return Object.prototype.toString.call(obj).slice(8, -1);
}

type(function(){}) //"Function"
```









