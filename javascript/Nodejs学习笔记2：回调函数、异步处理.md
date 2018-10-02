# Nodejs学习笔记2：回调函数、异步处理

## 1. 回调函数：

> 当异步操作完成时就会调用的函数

### 阻塞函数与非阻塞函数

- 阻塞函数：

  ```javascript
  var fs = require("fs");
  
  var data = fs.readFileSync('input.txt');
  
  console.log(data.toString());
  console.log("程序执行结束!");
  ```

  <font color=#FF0033>此函数执行时，一定会在执行fs.readFileSync之后再执行后面的指令。</font>

  本例中会先输出**input.txt中的内容->之后输出“程序执行结束”**

- 非阻塞函数：

  ```javascript
  var fs = require("fs");
  
  fs.readFile('input.txt', function (err, data) {
      if (err) return console.error(err);
      console.log(data.toString());
  });
  
  console.log("程序执行结束!");
  ```

  此函数执行时，会将任务请求先提交，执行后面的代码，如果任务得到结果则根据回调函数进行处理。

  本例中可能会先输出**“程序执行结束”->之后输出input.txt中的内容**

## 2. 无阻塞事件驱动服务：

![img](http://www.runoob.com/wp-content/uploads/2015/09/event_loop.jpg)

类似于观察者模式的服务方式：

- 用户（函数）发送一个事件处理的请求
- webserver收到请求就将其放到处理队列（Event Loop）中（相当于用户（函数）订阅了这个webserver）并暂时关闭这个请求，继续处理接下来的请求。
- 当处理队列中的任务完成，webserver就将结果放到event队列中，给用户返回处理好的值（回调函数）
- 在事件驱动模型中，会生成一个主循环来监听事件，当检测到事件时触发回调函数。

## 3. EventEmitter类

> 提交的异步处理在I/O完成之后都会发送一个事件到事件(event)队列，之后才会将得到的结果经回调函数给用户

**其中事件的触发与事件监听器的功能就在EventEmitter类中封装。**

1. 实例化EventEmitter类

   ```javascript
   // 引入 events 模块
   var events = require('events');
   // 创建 eventEmitter 对象
   var eventEmitter = new events.EventEmitter();
   ```

2. EventEmitter的基本方法

   - 得到eventEmitter对象之后，就可以自定义一个事件

   ```javascript
   eventEmitter.on('some_event', function() { 
       console.log('some_event 事件触发'); 
   }); 
   ```

   **在这里定义了事件的名以及当触发事件时会调用的回调函数。**

   - 之后需要触发事件时，使用：

   ```javascript
   setTimeout(function() { 
       eventEmitter.emit('some_event'); 
   }, 1000); 
   ```

   **使用eventEmitter的emit方法来触发事件完成。**

   - 当回调函数有参数时：

   ```javascript
   //event.js 文件
   var events = require('events'); 
   var emitter = new events.EventEmitter(); 
   emitter.on('someEvent', function(arg1, arg2) { 
       console.log('listener1', arg1, arg2); 
   }); 
   emitter.on('someEvent', function(arg1, arg2) { 
       console.log('listener2', arg1, arg2); 
   }); 
   emitter.emit('someEvent', 'arg1 参数', 'arg2 参数'); 
   ```

   **在emit函数中输入参数即可。**

3. EventEmitter的完全方法

   http://www.runoob.com/nodejs/nodejs-event.html

4. 大多数使用EventEmitter的方法都是创建继承EventEmitter的类。