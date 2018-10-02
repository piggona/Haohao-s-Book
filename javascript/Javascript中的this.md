# JavaScript中的this

[参考](http://www.ruanyifeng.com/blog/2018/06/javascript-this.html)

## 1. 重要的例子：

```javascript
var obj = {
  foo: function () { console.log(this.bar) },
  bar: 1
};

var foo = obj.foo;
var bar = 2;

obj.foo() // 1
foo() // 2
```

这种差异的原因，就在于函数体内部使用了`this`关键字。很多教科书会告诉你，`this`指的是函数运行时所在的环境。对于`obj.foo()`来说，`foo`运行在`obj`环境，所以`this`指向`obj`；对于`foo()`来说，`foo`运行在全局环境，所以`this`指向全局环境。所以，两者的运行结果不一样。

这种解释没错，但是教科书往往不告诉你，为什么会这样？也就是说，函数的运行环境到底是怎么决定的？举例来说，为什么`obj.foo()`就是在`obj`环境执行，而一旦`var foo = obj.foo`，`foo()`就变成在全局环境执行？

## 2. 内存的数据结构



## 3. 对于函数的this

[参考](http://www.cnblogs.com/penghuwan/p/7356210.html)

对于函数来说，只要它作为独立函数来使用，它的作用域就是window。

如果这个函数被一个对象所<font color=#FF0033>**直接包含**</font>，那么this就被隐式绑定到这个对象中。

<font color=#FF0033>判断这种情况的最好方式就是看函数的上一层是什么</font>

如果是函数，那么this作用域就是window:

```javascript
function fire () {
  // 我是被定义在函数内部的函数哦！
     function innerFire() {
  console.log(this === window)
      }
     innerFire(); // 独立函数调用
}
fire(); // 输出true
```

如果被对象直接包含，那作用域就是对象：

```javascript
var obj = {
     a: 1,
      fire: function () {
           console.log(this.a)
        }
}
obj.fire(); // 输出1
```



```javascript
var obj = {
    birth: 1990,
    getAge: function () {
        var b = this.birth; // 1990
        var fn = function () {
            return new Date().getFullYear() - this.birth; // this指向window或undefined
        };
        return fn();
    }
};
obj.getAge();
```

上面的例子，调用getAge时,它本身的上层是obj对象，所以作用域在obj中，所以this.birth=1990。

但当它调用fn时，fn函数的上层是getAge这个函数，所以作用域在window上，使用this.birth会报错。

## 4. this的显式绑定（call&bind)

[参考](http://www.cnblogs.com/penghuwan/p/7356210.html)

call的基本使用方式：fn.call(object)

fn是调用的函数，object是你希望函数的this绑定的对象。

```javascript
var obj = {
      a: 1,    // a是定义在对象obj中的属性
      fire: function () {
         console.log(this.a)
      }
}
 
var a = 2;  // a是定义在全局环境中的变量  
var fireInGrobal = obj.fire;
fireInGrobal();   // 输出2
fireInGrobal.call(obj); // 输出1
```

当使用call的时候，函数会立即执行，并且其中的this会指向obj这个对象。

当我们想永久的绑定一个this到对象中时，就使用bind函数。它不立即执行，返回的就是一个可供执行的函数。

```javascript
var fireInGrobal = fn.bind(obj);
```

