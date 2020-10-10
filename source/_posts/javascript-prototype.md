---
title: JAVASCRIPT 原型链污染
date: 2018-12-12 04:09:27
tags: 
- javascript
---

### 原型和原型链

当查找一个对象的时候，Javascript会向上遍历原型链(`prototype chain`)，直到找到给定名称的属性为止，查找到原型链的顶部，也就是`Object.prototype`，如果这个时候还是没有找到，那么就会返回`undefined`。例子如下：
```
function foo() {
    }
}

foo.prototype.add = function (x, y) {
    return x + y + 10;
}

Object.prototype.sub = function(x, y) {
    return x - y;
}

var f = new foo();
console.log(f.add(1, 2));   //3
console.log(f.sub(1, 2));   //-1
```
从代码运行的结果来看，`sub`是通过向上层查找来执行得到的结果，而`add`是先查找自身属性，如果没有再查找原型，再没有，继续往上走，一直查到`Object`的原型上（`foo`->`foo.prototype`->`Object.prototype`），这就造成了一些安全问题，子类可以随意修改父类的原型的值；在某种层面上说，用`for in`语句遍历属性的时候，效率也是一个问题。

### hasOwnProperty函数
`hasOwnProperty`函数是`Object.prototype`的一个方法，它能够判断一个对象是否包含自定义属性，而不是原型上的属性，因为`hasOwnProperty`是`Javascript`中唯一一个处理属性但是不查找原型链的函数。
```
Object.prototype.bar = 1;
var foo = { goo: undefined };

foo.bar;    //1

'bar' in foo;   //true

console.log(foo.hasOwnProperty('bar'));      //false
console.log(foo.hasOwnProperty('goo'));      //true
```
从上面代码看得出`hasOwnProperty`能够给出正确的结果，这在遍历对象的时候会很有用。但是有一个需要注意的地方，`hasOwnProperty`可能会被其他外部对象非法占用，即碰巧某一个对象，有一个属性名也刚好叫做`hasOwnProperty`，这时候就需要使用外部的`hasOwnProperty`来获取正确的结果：
```
var foo = {
    hasOwnProperty: function () {
        return false;
    },
    bar: 'bingo'
};

console.log(foo.hasOwnProperty('bar'));      // false

console.log(Object.hasOwnProperty.call(foo, 'bar'));    //true
```

在循环遍历某个对象的时候，也可以使用`hasOwnProperty`,这样可以避免原型对象带来的干扰：
```
Object.prototype.bar = 1;
var foo = {moo: 2};
for (var i in foo) {
    console.log(i);     //moo, bar
}
```
`foo`在自己的原型上没有找到`bar`，所以向上查找，使用`hasOwnProperty`保证输出结果正确：
```
Object.prototype.bar = 1;
var foo = {moo: 2};
for (var i in foo) {
    if (foo.hasOwnProperty(i)) {
        console.log(i)
    }
}
```