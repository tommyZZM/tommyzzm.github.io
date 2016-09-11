# ECMAScript 5.1 实用特性概览

**ECMAScript 5**发布于2009年12月。**ECMAscript 5.1**版(下文称ES5)发布于2011年6月，，并且成为ISO国际标准（ISO/IEC 16262:2011）

http://www.ecma-international.org/ecma-262/5.1/

> ECMAScript 5.1 是ECMAScript(基于JavaScript的规范)标准最新修正。 与HTML5规范进程本质类似，ES5通过对现有JavaScript方法添加语句和原生ECMAScript对象做合并实现标准化。

虽然如今再谈这些"老掉牙"的方法，在如今这个`言必及ES6`的时代略显落伍，但是ES5作为javascript发展中的重要一环，`ECMAScript2015`(下文称ES6)的新特性也是建立在ES5的基础之上的，通过回顾这些实用的ES5特性巩固基础，将有助于我们在今天和未来更好的使用javascript。


## 实用特性

### Object

#### **[`Object.defineProperty()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)**

>该方法直接在一个对象上定义一个新属性，或者修改一个已经存在的属性， 并返回这个对象。[`Object.defineProperties()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperties)与其一样,只是可以同时定义多个属性。

该方法允许精确添加或修改对象的属性。常用的场景
- 定义`setter`和`getter`。 (在ES6中已经有了更好的定义方法)
- 定义对象属性是否可枚举`enumerable`。
    

可枚举的属性键值能够被`for in`和`Object.keys`获得。
    
最常见的例子就是，`[]数组`中索引属性是可枚举的，而标准成员方法就是不可枚举的。

~! 这也是为什么我们不要使用`for in`遍历数组的原因，因为可能有一些拙劣的上下文代码，为数组添加了一个可枚举的方法，因此我们在扩展一个特殊对象属性时特别需要特别关注这一点


#### **[`Object.keys()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/keys)**

把对象的返回一个包括对象`可枚举键值`的数组。

`Object.keys`和语句 `for in` 的功能十分相似，经常是一些细心的javascript开发者的讨论热点。回想那ES5还不完全兼容的年代， `for in` 承担了遍历对象键值的任务。

**但是两者实际上是不一样的！**

`for in`会遍历对象原型链上所有的属性，包括继承下来的属性，而 `Object.keys` 只会遍历对象本身自己拥有的属性，因此在一些场景下 `Object.keys` 更快，所以在很多场景下我们都应该优先使用`Object.keys` 。

```
var a = {a1:1,a2:2,a3:3};
var b = {b1:1,b2:2,b3:3};
b.__proto__ = a;
for(var key in b) {
   console.log(key); //b1,b2,b3,a1,a2,a3
}

Object.keys(b); //b1,b2,b3
```

参考：[why-is-object-keys-faster-than-hasownproperty](http://stackoverflow.com/questions/30326452/why-is-object-keys-faster-than-hasownproperty)

而且由于该方法返回的是一个数组，因此我们也能很好的结合数组方法去处理对象中的数据

```
var obj = { a:'1', b:'2', c:'3' };
var values = Object.keys(obj).map(function(key){return obj[key]}); // 1,2,3
```

#### **[`Object.freeze()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/freeze) 与 [`Object.isFrozen`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/isFrozen)**

>方法可以冻结一个对象。冻结对象是指那些不能添加新的属性，不能修改已有属性的值，不能删除已有属性，以及不能修改已有属性的可枚举性、可配置性、可写性的对象。也就是说，这个对象永远是不可变的。该方法返回被冻结的对象。
    
注意的是这个方法只会冻结被传入的对象,而不会冻结对象key值所引用的对象。

```
var obj = {a: {b: 'c' }};

Object.freeze(obj);
obj.a = 123; //不能成功赋值,但也不会报错,只会静默失败
obj.a.b = 'd'; //能成功赋值 => {a: {b: 'd'}}
```
    
类似的方法还有`Object.seal(obj)`和`Object.preventExtensions(obj)`

`Object.freeze`引入不可变数据概念，但是由于实际开发中对性能的忧虑，实际很少被用到。

如果有场景需要避免对象使用者修改对象，`Object.freeze`将是一个很好的方法。而且由于`数组[]`也是一种对象，同样也可以为通过该方法冻结数组。

### Array

对于数组标准API的扩展可以说是ES5.1中的重头戏，这些核心API今天和未来都将为我们带来便利和启发, 这里列举和介绍一些常用的标准API。

#### **[`Array.isArray()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/isArray)**

> Array.isArray() 方法用来判断某个值是否为数组。如果是，则返回 true，否则返回 false。

javascript有六种原始数据类型`primitive`，包括`undefined`,`object`,`function`,`boolean`,`number`,`string`

其中`object`包括四种能通过语法糖构造的形态`{}`,`null`,`/\w+/`(正则表达式),`[]`(数组)

对于后三种特殊`object`，`null`可以使用`xxx === null`，正则表达式可以通过`xxx instanceof RegExp`判断，而数组大多数时候似乎可以通过`xxx instanceof Array`判断。

但是数组的情况仍然比较特殊,主要是数组在iframe中被传递时的场景,`xxx instanceof Array`会出现误判。这种情况较为罕见，相关资料参考: 
- [Difference between using Array.isArray and instanceof Array](http://stackoverflow.com/questions/22289727/difference-between-using-array-isarray-and-instanceof-array)
- [Determining with absolute accuracy whether or not a JavaScript object is an array](http://web.mit.edu/jwalden/www/isArray.html)

尽管`xxx instanceof Array`和`Array.isArray()`大多数时候表现是一致的，但是我们仍然应该使用更加完备健壮的后者。

**注意的是该方法并不能判断一些很像数组的对象`ArrayLike`，例如`querySelectorAll`返回的`ElementsList`，通过ES6(`ECMAScript 2015`)引入的`Array.from`我们可以将其转换成为标准数组**

#### **`[...].forEach(fn)`**

> 让数组的每一项都执行一次给定的函数，返回值为`undefined`

大多数时候该方法是语句`for(var i=0;i<length;i++){...}`的替代，在解决作用域变量提升的时候`forEach`是一个很好的方法。

对于大规模的数组处理，`forEach`性能比`for`语句要慢很多，而且有没有额外的产出，笔者觉得大多数时候比较鸡肋。但是`forEach`仍然可能算是读过的代码中使用频率较高的数组方法之一。

#### **[`[...].map(fn)`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/map)**

> 返回一个由原数组中的每个元素调用一个指定方法后的返回值组成的**新数组**。

`map`方法实际上是把一个数组**映射**成为另外一个数组

功能虽然简单，和forEach很像，但因为能返回新的数组而变得十分实用（失之毫厘，差之千里）。

对于任何一个数组集合，都可以使用map进行映射操作，实现很丰富的功能，而且保证代码的**可读性，该API也十分受开发者欢迎。

**

```
var arr = ["a","b","c"]
var arrToUpperCase = arr.map(function(ele,index){return ele.toUpperCase()});
// ["A","B","C"]
```

#### **[`[...].filter(fn)`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/filter)**

> 方法使用指定的函数测试所有元素，并返回一个包含所有通过测试的元素的**新数组**。

同样也是十分实用的方法，有时候会看到不熟悉的小伙伴会使用`for`和新数组`push`实现类似的功能。

```
<ul>
    <li><a href='http://a.com'></a></li>
    <li><a href='#invalid'></a></li>
    <li><a href='http://b.com'></a></li>
    <li><a href='http://c.com'></a></li>
</ul>
```

```
var alinks = docuement.querySelector("ul a");
var alinksValid = arrayFrom(alinks).filter(function(a){
    return a.getAttribute("href")[0]!=="#"
})

function arrayFrom(arrayLike){
    //... Array.from polyfill
}

/* 
[
   <a href='http://a.com'></a>
   ,<a href='http://b.com'></a>
   ,<a href='http://c.com'>
]
*/
```

#### **[`[...].some(fn)`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/some)**

> 方法测试数组中的某些元素是否通过了指定函数的测试，返回boolen值

非常实用的功能，判断数组中是否某元素符合特定条件。如果传入的方法校验为`true`则余下的元素都不会继续遍历，不会有冗余的元素访问。

```
var arr = [
   {name:"xiaoA"}
   ,{name:"xiaoB"}
   ,{haha:">_<"} 
   ,{name:"xiaoB"} //不会被访问,已经跳出
]

arr.some(function(ele){return !!ele.haha}) // true
```

该方法有很多用途
- 数组元素进行一些灵活的校验
- 选择数组中第一个符合条件的元素

和`for continue break`组合相比，`.some`方法提供更好的可读性和灵活性。

#### **[`[...].every(fn)`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/every)**

和`some`类似，但作用是检查所有元素是否符合条件，大多数场景下都可以用`some`实现相关的判断，因此出场频率极低。

#### **[`[...].reduce()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce) 与 `[...].reduceRight()`**

> 方法接收一个函数作为累加器（accumulator），数组中的每个值（从左到右）开始合并，最终为一个值。

最灵活的方法，能够实现非常丰富的功能。

- `reduce`和`reduceRight`作用一样，不同的是分别是从左右方向开始累加
- `reduce(fn,initialValue)`方法第二个参数为初始值`initialValue`，会传入累加方法的第一次调用时的第一个参数中，默认是数组的第一个元素

##### **累加运算**
作为累加器，数值计算可以说是最普通的使用方法。
```
[1,2,3].reduce(function(a,b){return a+b}); // 1+2+3 => 6
```

##### **数组去重** 
```
[1,2,2,3,3,3].reduce(function(arr,curr){
    if (arr.some(equal(curr))) return arr;
    arr.push(curr);
    return arr;
},[]); // [1,2,3]

function equal(value){
    return function(target){return value===target}
}
```

##### **对象K-V倒置**
```
var obj = { a:1, b:2, c:3 }
Object.keys(obj).reduce(function(_obj,key){
    _obj[obj[key]] = key;
    return _obj;
},{}) // {1:"a", 2:"b", 3:"c"}
```

#### 数组加工管道

`map`、`filter`和`reduce`都能返回一个数组,因此我们可以让其通过`链式`组合成为数组加工的管道。

```
[...].map(fn).filter(fn).reduce(fn)
```

### JSON

两个JSON方法都很常用了~

#### **[`JSON.parse()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/JSON/parse)**

> 根据[rfc4627](http://www.ietf.org/rfc/rfc4627.txt)标准解析JSON文本。

注意parse如果结果错误时会抛出异常，阻塞当次事件循环中接下来的代码。所以通常需要进行`try catch`进行防御性校验

```
function JSONparseSafe(str) {
    try {
       return JSON.parse(str);
    } catch(e) {
       console.warn(str,e);
    }
    return {};
}
```

#### **[`JSON.stringify()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify)**

> 把对象序列化成为JSON格式字符串

很常用了~,和`JSON.parse()`一样会抛出异常,笔者认为同样需要进行`try catch`进行防御性校验

注意该方法还有第二和第三个参数的

`JSON.stringify(value[, replacer [, space]])`

- `replacer`
如果该参数是一个函数，则在序列化过程中，被序列化的值的每个属性都会经过该函数的转换和处理；如果该参数是一个数组，则只有包含在这个数组中的属性名才会被序列化到最终的 JSON 字符串中。关于该参数更详细的解释和示例，请参考使用原生的 JSON 对象一文。

- `space`
指定缩进用的空白字符串，用于美化输出（pretty-print）。

##### **序列化循环嵌套对象(`circular structure`)**
```
var obj = { a:1, b:2 };
obj.self = obj;
// JSON.stringify(obj)
// Uncaught TypeError: Converting circular structure to JSON(…)

// 总所周知,JSON.stringify在序列化循环对象时会抛出异常
// 这时我们可以使用这样来解决循环对象的问题
JSON.stringify(obj ,function(key,value){
    if (key && value === obj) return "{[circular]}"
    return value
}) //"{"a":1,"b":2,"self":"{[circular]}"}"

```

### String

#### `"string".trim()`

> 方法会删除一个字符串两端的空白字符。在这个字符串里的空格包括所有的空格字符

```
("     aa     ").trim() //"aa"
```

方法虽然简单,但是那些还用正则替换实现同样的功能的同学好自为之吧<_<

### Date

- `Date.now`
- `Date.prototype.toISOString` 

### 保留关键字

虽然在ES5时代,ES6的具体标准特性还没有定稿,但是已经定义一些**"未来"会被使用的**`保留的关键字`,在完全实现ES5标准的浏览器/JS引擎中**关键字是不能作为字面量(literal)名的**,否则将会报错。这些关键字包括：

- `class`
- `extend`
- `super`
- `enum`
- `import`
- `export`

```
var class = "abc" // Uncaught SyntaxError: Unexpected token =
```

嗯，没错我们将（已经）在ES6中用上了他们。

## 兼容性与性能

#### 影响

随着Web的发展,前端开发者对javascript**数据处理能力**的诉求越来越强烈，以至于诞生了后来如`jQuery`、`underscore`等广泛使用的工具库。

而`ECMAScript 5`的到来也正是响应这一诉求，通过对js标准库的扩展，极大的增强了js的**处理数据的能力**，尤其是面对js中`[]`(数组)和`{}`(对象/哈希表)为核心的复合数据类型时。标准API抹平了不同工具库对同一个功能的实现差异，同时也提高了代码的可读性。

`ECMAScript 5`的API设计也吸收了其他类型编程语言中实用思想，例如`管道(pipeline)`、`无状态(no side effects)`、`数据不可变(immumable)`等概念，更后来引入的`Stream`(pipe(...).pipe(...))和`Promise`(then(...).then(...))也能看到这些影子。

还促进了实用工具库的完善，例如后来的[`lodash`](https://lodash.com/docs) 和 [`ramda`](http://ramdajs.com/0.21.0/docs/)等工具库。

ES6也是朝着同样的方向对js进行完善。

#### 兼容性

> http://kangax.github.io/compat-table/es5/

作为javascript标准库，ES5的特性已经被主流浏览器完全支持，包括`桌面端`、`移动端`和`Node.js`中都可以放心使用。
（即使你的客户还在使用IE5、IE7、IE8，大多数也能使用`es5-polyfill`进行兼容）

#### 性能

对某些标准API带来潜在性能损耗的猜疑确实使得相当多小伙伴望而却步，但是我们仍然应该去了解这些标准的API，学习他们的设计思想，即便在一些性能要求较高的场景中，我们仍然能够以标准作为参考，**设计出优雅的可读性高的方法**，而在大部分的场景中我认为应该**合理去使用**它们，享受标准完善带来的红利。

这样能以一个更好地准备，拥抱未来更多的新特性。

#### 相关链接
- [JavaScript 标准库 - MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects)
- [Optimization-killers - bluebird](https://github.com/petkaantonov/bluebird/wiki/Optimization-killers) (JavaScript性能优化技巧)
- [Composers and audiences](http://www.pocketjavascript.com/blog/2016/1/25/composers-and-audiences) ([译文](http://efe.baidu.com/blog/composers-and-audiences/))

## Enjoy!

~ 当我们为`ES6`的特性感到兴奋的同时, 我们已经知晓如何应用`ES5`了吗?

