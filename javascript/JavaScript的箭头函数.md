# JavaScript的箭头函数(Arrow Function)

>  箭头函数相当于匿名函数，而且解决了作用域的问题

## 1. 箭头函数的形式

- 简单箭头函数

  ```javascript
  ‘use strict'
  var fn = x => x * x;
  ```

  如果函数只有一个变量且只有一条return语句，则可以如上。

- 有多个参数的箭头函数

  ```javascript
  // 两个参数:
  (x, y) => x * x + y * y
  
  // 无参数:
  () => 3.14
  
  // 可变参数:
  (x, y, ...rest) => {
      var i, sum = x + y;
      for (i=0; i<rest.length; i++) {
          sum += rest[i];
      }
      return sum;
  }
  ```

- 返回对象的箭头函数

  ```javascript
  x => ({ foo: x })
  ```


## 2. 箭头函数与匿名函数的区别

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
```

在上面的例子中，函数fn的外层是getAge函数，所以这里面this的作用域是window.

如果像使用obj的作用域就要：

```javascript
var obj = {
    birth: 1990,
    getAge: function () {
        var b = this.birth; // 1990
        var that = this; //赋值this到that
        var fn = function () {
            return new Date().getFullYear() - that.birth; // this指向window或undefined
        };
        return fn();
    }
};
```

或者使用箭头函数，箭头函数内部的this指向的是词法作用域，取决于谁调用这个函数（函数运行的函数）

```javascript
var obj = {
    birth: 1990,
    getAge: function () {
        var b = this.birth; // 1990
        var fn = () => new Date().getFullYear() - this.birth; // this指向obj对象
        return fn();
    }
};
obj.getAge(); // 25
```

