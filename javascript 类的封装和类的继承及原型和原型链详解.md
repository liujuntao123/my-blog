---
title: javascript 类的封装和类的继承及原型和原型链详解
date: 2018-05-17 09:52:19
tags:
- ES6 
- 继承
- 原型
- 原型链
categories: 深入JS
comments: true
---

本文主要讲ES5、ES6中类的定义、封装和类的继承，以及一些注意事项，文中除了参考引用一些资料外，也加入了很多自己的理解，如有错误，欢迎读者纠正。

## javascript 类的定义

### ES5 中的类

在 ES5 中，其实是没有类的概念的，原因是因为 js 的作者当初在设计这门语言的时候认为，一旦像 C++和 Java 一样引入了类，js 就是一种完整的面向对象编程语言了，这好像有点太正式了，而其增加了初学者的入门难度。他当时认为，没必要将 js 设计得很复杂，这种语言只要能够完成一些简单操作就够了，比如判断用户有没有填写表单。因此，有了一些模拟定义类的方法出现。

#### 构造函数法

这个方法相信稍微有些 js 基础的人都能知道。

```javascript
function People(name) {
  this.name = name
}
var jack = new People('jack')
People.prototype.constructor === People // true
jack.constructor === People // true
jack.__proto__ === People.prototype // true
```

但是这种方法的 protype 上的属性只能在外面定义，不够语义化，也挺麻烦。

```javascript
People.prototype.species = 'human'
```

#### Object.create()法

为了解决上述的缺点，ES5 有了 Object.create()的方法来直接传入 prototype。

```javascript
var People={
  species='human'
}
var jack=Object.create(People)
jack.name='jack'
```

显而易见，这种方法的问题是定义实例属性比较麻烦，而且也不能实现私有属性和私有方法。

#### 极简主义法

这种方法用一个对象模拟类，在对象里定义一个函数 createNew()模拟构造函数的功能，来生成实例。

```javascript
var Creature = {
  blood: 'red', //瞎写的，有些生物的血不是红色的:)
}
var People = {
  species: 'human',
  createNew: function(name) {
    var people = {}
    people.name = name
    return people
  },
}
```

这个方法可以既可以有私有属性和方法，而且也能比较语义化的定义实例的各种属性。但是这个方法有一个缺点，那就是在继承的时候没有用到 prototype 的概念，也就是没有用到 js 自己的原型链的概念。会让人觉得不够直观。

总而言之，ES5 没有类，哪怕是模拟实现了类，也不够完美。

### ES6 中的类

ES6 提供了更接近传统语言的写法，引入了 Class（类）这个概念，作为对象的模板。通过 class 关键字，可以定义类。基本上，ES6 的 class 可以看作只是一个语法糖，它的绝大部分功能，ES5 都可以做到，新的 class 写法只是让对象原型的写法更加清晰、更像面向对象编程的语法而已。

```javascript
//定义类
let methodName = 'getArea'
class Foo {
  constructor(x, y) {
    this.x = x
    this.y = y
  }

  toString() {
    return '(' + this.x + ', ' + this.y + ')'
  }
  [methodName]() {
    return 'area_' + this.x + '_' + this.y
  }
}
let foo = foo('a', 'b') //报错
let foo = new Foo('a', 'b')
typeof Foo // 'function'
Foo === Foo.prototype.constructor // true
foo.__proto__ === Foo.prototype // true
foo.toString === Foo.prototype.toString //true
```

可以看到，类的所有方法都定义在类的 prototype 属性上面。在类的实例上面调用方法，其实就是调用原型上的方法。类的属性名，可以采用表达式。类的内部所有定义的方法，都是不可枚举的（non-enumerable）。这一点与 ES5 的行为不一致。类必须使用 new 调用，否则会报错。这是它跟普通构造函数的一个主要区别，后者不用 new 也可以执行。类不存在变量提升（hoist），这一点与 ES5 完全不同。

#### new.targer 属性

new 是从构造函数生成实例对象的命令。ES6 为 new 命令引入了一个 new.target 属性，该属性一般用在构造函数之中，返回 new 命令作用于的那个构造函数。如果构造函数不是通过 new 命令调用的，new.target 会返回 undefined，因此这个属性可以用来确定构造函数是怎么调用的。

```javascript
function Person(name) {
  if (new.target !== undefined) {
    this.name = name
  } else {
    throw new Error('必须使用 new 命令生成实例')
  }
}

// 另一种写法
function Person(name) {
  if (new.target === Person) {
    this.name = name
  } else {
    throw new Error('必须使用 new 命令生成实例')
  }
}

var person = new Person('张三') // 正确
var notAPerson = Person.call(person, '张三') // 报错
```

## javascript 类的继承

### ES5 中类的继承

#### 构造函数绑定

使用 call 或 apply 方法，将父对象的构造函数绑定在子对象上，即在子对象构造函数中加一行：

```javascript
function Animal() {
  this.species = 'animal'
}
function Cat(name, color) {
  Animal.apply(this, arguments)
  this.name = name
  this.color = color
}

var cat1 = new Cat('miao', 'black')

alert(cat1.species) // 动物
```

这样能够达到继承的目的，但是问题是，它会为所有的 Cat 实例都创建一个自己的 species 属性，这样会浪费内存，显然是不太优雅的做法

#### 继承 prototype

由于 Animal 对象中，不变的属性都可以直接写入 Animal.prototype。所以，我们也可以让 Cat()跳过 Animal()，直接继承 Animal.prototype。

```javascript
function Animal() {}
Animal.prototype.species = 'animal'
Cat.prototype = Animal.prototype
// 因为Cat的prototype直接指向了Animal的prototype,因此它的constructor属性也发生了变化，需要手动纠正Cat的构造函数
console.log(Cat.prototype.constructor === Animal) // true
Cat.prototype.constructor = Cat
console.log(Animal.prototype.constructor === Cat) // true
var cat1 = new Cat('miao', 'black')

alert(cat1.species) // animal
```

这里出现了一个问题，由于是直接赋值的，所以 Cat 的 prototype 和 Animal 的 prototype 指向了同一个对象，在修改 Cat 的 prototype.constructor 的时候，Animal.prototype.constructor 也随之被更改。因此可以采用一个中间量过渡。

```javascript
var F = function() {}
F.prototype = Animal.prototype
Cat.prototype = new F()
Cat.prototype.constructor = Cat
console.log(Cat.prototype === Animal.prototype) // false
```
这样，就能解决上述问题了。

### ES6 中类的继承
Class 可以通过extends关键字实现继承，这比 ES5 的通过修改原型链实现继承，要清晰和方便很多。

``` javascript
class Foo {
  constructor(x, y) {
    this.x = x
    this.y = y
  }
  static hello(){
    console.log('hello world')
  }

  toString() {
    return '(' + this.x + ', ' + this.y + ')'
  }
}
class ChildFoo extends Foo {
  constructor(x, y, z) {
    this.z=z //报错
    super(x, y); // 调用父类的constructor(x, y)
    this.z = z;
  }

  toString() {
    return this.z + ' ' + super.toString(); // 调用父类的toString()
  }
}
console.log(ChildFoo.hello===ChildFoo.__proto__.hello) // true
console.log(ChildFoo.hello===Foo.hello) // true
```
子类必须在constructor方法中调用super方法，否则新建实例时会报错。这是因为子类没有自己的this对象，而是继承父类的this对象，然后对其进行加工。如果不调用super方法，子类就得不到this对象。此时子类调用this对象，就会报错。
ES5 的继承，实质是先创造子类的实例对象this，然后再将父类的方法添加到this上面（Parent.apply(this)）。ES6 的继承机制完全不同，实质是先创造父类的实例对象this（所以必须先调用super方法），然后再用子类的构造函数修改this。
最后，父类的静态方法，也会被子类继承。



## javascript 原型和原型链，以及prototype和__proto__
### 什么是原型？
每个javascript对象都有一个私有属性,称之为``` Prototype```(注意这里只是一个称谓，非一个属性)，翻译过来就是原型，它被现今大多数浏览器实现为__proto__属性。这个__proto__属性，指向对象的类（构造函数）的```prototype```属性，这个属性也被称之为原型对象(注意和前面的原型不是一个概念)。每一个对象(null除外)在创建的时候就会从原型"继承"它所有的属性。
上述用代码表示就是：
``` javascript
function Person(name) {
  this.name=name
}
let person = new Person();

// 实例化之后的对象，这个对象的原型指向它的类的原型对象
console.log(person.__proto__===Person.prototype) // true
```
### 什么是原型链？
JavaScript 对象有一个指向一个原型对象的链。当试图访问一个对象的属性时，它不仅仅在该对象上搜寻，还会搜寻该对象的原型，以及该对象的原型的原型，依次层层向上搜索，直到找到一个名字匹配的属性或到达原型链的末尾。
上述用代码表示就是：
``` javascript
function Creature(){}
Creature.prototype.blood='red'
function Person(name) {
  this.name=name
}

// 将Person作为Creature的子类
Person.prototype=new Creature()

let person = new Person();

// 注意，下面是两段伪代码，实际运行后不会得到true，原因很简单，本文就不讨论了，请自行思考 :)

// 沿着等号往后走，这就是一条原型链。
console.log(person.blood===person.__proto__.blood=== Person.prototype.blood===Person.prototype.__proto__.blood===Creature.prototype.blood) // true

// 这是一个在原型链上没有找到指定属性的例子，原型链会一直延伸到最末端
console.log(person.tail===person.__proto__.tail=== Person.prototype.tail===Person.prototype.__proto__.tail===Creature.prototype.tail===Creature.prototype.__proto__.tail===Object.prototype.tail===undefined)
```
代码中可以看到，当访问person对象的blood属性的时候，显然本地是没有这个属性的，那么就会去寻找它的原型(即__proto__，也即Person.prototype)是否有这个属性，结果它的原型也没有这个属性。它的原型Person.prototype本身同样也是一个对象，那么它也有自己的原型，此时它会在自己的原型上继续寻找这个属性，也就是```Person.prototype.__proto__```,而```Person.prototype.__proto__```指向的是Creature.prototype，在Creature.prototype上找到了blood这个属性，原型链结束。

### 有关 prototype 的方法
使用__proto__是有争议的，官方也不鼓励使用它。因为它从来没有被包括在EcmaScript语言规范中，但是现代浏览器都实现了它。
不过__proto__属性已在ECMAScript 6语言规范中标准化，用于确保Web浏览器的兼容性，因此它未来将被支持。
但是官方依然不推荐使用它，而是推出了一系列的有关原型的方法。

#### Object.getPrototypeOf(object)
此方法放回给定对象的原型，注意这里的原型是指__proto__，而非prototype属性。
``` javascript
var proto = {};
var obj = Object.create(proto);
Object.getPrototypeOf(obj) === proto; // true
```
#### Object.setPrototypeOf(obj,prototype)
此方法可以重新设置一个对象的原型，同样，这里的原型依然指的是__proto__，其实就基本等同于Object.create()方法。
```javascript
var proto={}
var obj={}
Object.setPrototypeOf(obj,proto)
obj.__proto__===proto
```

#### prototypeObj.prototype.isPrototypeOf(object)
此方法用来检查一个对象是否是另一个对象的原型，可以理解为等同于
```javascript
prototypeObj.prototype===object.__proto__
```

## 私有方法/属性、静态方法/属性、公共方法/属性、实例方法/属性

经常会听到体积一些私有属性公有属性，静态属性静态方法之类的词汇，趁此机会，也对这些概念做一个系统的梳理。
### 私有方法/属性
指在类的内部运算使用的方法和属性，而其他人无论是通过这个类还是通过这个类的实例都无法访问。
其实很好理解为什么ES6的类没有私有属性和私有方法，因为它没有目前提供这个语法。
给一个变量赋值，无非两种方法，一种是等号，一种是声明式的，而ES6的类内部的语法很有限，并不能通过任何方法来进行一个变量赋值，等号的话不能识别，而直接声明的函数又会被解析为公有方法。
所以那些模拟的私有方法和属性也就容易理解了，其实就是要达到在类的内部能够全局使用，而脱离了这个类就无法访问的目的。
### 静态方法/属性
是指这个类自己的属性和方法，不会被继承，而是通过类本身来调用。比如Creature这个类，那么他的静态方法可以通过Creature.someProperty这样的方式访问到
### 公共方法/属性
其实就可以简单的理解为原型对象(prototype)上的方法和属性，所有的实例化的对象都是能继承到的和共享的。
### 实例方法/属性
就是constructor函数里this后面的一些赋值的方法和属性，是没有实例自己私有的。






## 参考文档

http://www.ruanyifeng.com/blog/2011/06/designing_ideas_of_inheritance_mechanism_in_javascript.html
http://www.ruanyifeng.com/blog/2010/05/object-oriented_javascript_encapsulation.html
http://www.ruanyifeng.com/blog/2010/05/object-oriented_javascript_inheritance.html
http://www.ruanyifeng.com/blog/2010/05/object-oriented_javascript_inheritance_continued.html
http://www.ruanyifeng.com/blog/2012/07/three_ways_to_define_a_javascript_class.html
http://es6.ruanyifeng.com/#docs/class
http://es6.ruanyifeng.com/#docs/class-extends
https://developer.mozilla.org/zh-cn/docs/web/javascript/reference/global_objects/object
https://stackoverflow.com/questions/41189190/how-and-why-would-i-write-a-class-that-extends-null
https://github.com/mqyqingfeng/Blog/issues/2
https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Inheritance_and_the_prototype_chain
