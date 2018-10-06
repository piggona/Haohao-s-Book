# Javascript的Promise

> **Promise是实现javascript异步实现的重要方法**
>
> 与callback函数的不同在于：
>
> 1.关心于事件是否发生（已经发生，尚未发生，已经结束）而不是callback的监听器模式（若事件发生完，设置监听也没有意义因为已经监听不到事件）
>
> 2.而且promise只支持一次触发，callback可以多次触发事件

## 1. 简单的Promise实例

- 首先建立一个实用函数:

  ```javascript
  function test(resolve,reject){
      log('start new Promise...');
      var timeOut = Math.random() * 2;
      log('set timeout to: ' + timeOut + ' seconds.');
      setTimeout(function () {
          if (timeOut < 1) {
              log('call resolve()...');
              resolve('200 OK');
          }
          else {
              log('call reject()...');
              reject('timeout in ' + timeOut + ' seconds.');
          }
      }, timeOut * 1000);
  }
  ```

  这个函数时执行的过程，当达到我们认为的执行成功时就调用resolve函数并在里面传输执行成功的消息。

  ```javascript
  resolve('200 OK');
  ```

  当到达我们确定执行失败的条件时就调用reject函数,并传输执行失败的消息。

  ```javascript
  reject('timeout in' + timeout + 'seconds.');
  ```

- 之后将函数注册到Promise中（新建一个Promise对象）

  ```
  var p1 = new Promise(test)
  ```

- 之后就可以使用Promise对象来异步监听对象了，它关心于事件是否已经发生或已经失败

  ```javascript
  //then监听事件的成功发生，其中的r就是resolve传来的参数
  //catch监听事件的失败，其中reason就是reject传来的参数
  p1.then(function(r){      
      log('Done:' + r);
  }).catch(function(reason){
      log('Failed:' + reason);
  })
  ```


## 2. 串行执行异步任务

```javascript
job1.then(job2).then(job3).catch(handleError);
```

其中job1,job2,job3都是Promise对象。这些对象顺序执行，其中一个对象出错都会报错并执行catch.

## 3. 并行执行异步任务

- 并行执行并获得所有任务的结果（一个列表）：Promise.all()

  ```javascript
  var p1 = new Promise(function (resolve, reject) {
      setTimeout(resolve, 500, 'P1');
  });
  var p2 = new Promise(function (resolve, reject) {
      setTimeout(resolve, 600, 'P2');
  });
  // 同时执行p1和p2，并在它们都完成后执行then:
  Promise.all([p1, p2]).then(function (results) {
      console.log(results); // 获得一个Array: ['P1', 'P2']
  });
  ```

- 并行执行并获得最快完成的任务的结果：Promise.race()

  ```javascript
  var p1 = new Promise(function (resolve, reject) {
      setTimeout(resolve, 500, 'P1');
  });
  var p2 = new Promise(function (resolve, reject) {
      setTimeout(resolve, 600, 'P2');
  });
  Promise.race([p1, p2]).then(function (result) {
      console.log(result); // 'P1'
  });
  ```


