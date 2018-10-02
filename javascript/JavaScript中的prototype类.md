# JavaScript中的prototype

## 1. Javascript函数作用域

- JavaScript的命名空间：

  [参考](http://www.cnblogs.com/dolphinX/p/3269145.html)

  在javascript中只有函数作用域，容易引起各种类型之间的冲突引起错误，如何定义恰当的命名空间就成了一个重要的问题。

  **简单的例子**

  ```javascript
  <input type="button" value="test" onclick="alert();"/>
          
          <script type="text/javascript">
              function alert(){
                  //.......
                  
                  test2();
                  //.......
              }
              
              function test2(){
                  alert('test2')
              }
  
  ```

  在个例子在不同的浏览器下有不同表现，IE会报Stack over flow, Firefox会死掉。。。反正都会报错，很简单的错误，代码中自定义了一个alert函数，在alert函数中调用了test2函数，test2函数中意图调用window的alert方法，这样循环调用了，也许看了你会说这么明显的错误谁会犯，但是如果自定义的方法叫close（这个经常会出现吧），然后内部调用了一个外部文件的函数，该函数调用了window的close方法，这样错误是不是隐蔽了很多呢。

  ### 1.  简单的命名空间

  由于JavaScript没有文件作用域，不同的函数分散在不同的文件中，甚至由不同的人编写，重名的概率大大增加。是不是足够小心就可以了呢？也不尽然，还有些意外情况，比如经常会用到继承，于是写了一个没出现过的函数名extend，不料在EcmaScript5中加入了extend函数，命名空间的必要性就体现出来了。

  **JavaScript有函数的作用域**，可以利用这点把自定义的函数写到一个函数体内，这样函数内的变量、对象、函数就像在一个命名空间内一样和外部隔离。

  ```javascript
  <input type="button" value="test" onclick="(new namespace()).alert();"/>
          
          <script type="text/javascript">
              function namespace(){
                  this.alert=function(){
                      console.log('test');
                  }
              }
          </script>
  ```

  <font color=#FF0033>这样自定义的alert方法就不会和window的alert冲突了。</font>使用(new 命名空间函数()).自定义函数()的方法使用。

  ### 2. 立即执行函数

  ```javascript
  //两种立即执行函数的形式
  //1. 
  (function xxx(){
  
         //function body 
  
   })();
  
  //2. 
  function xxx(){
  
         //function body 
  
   }
  
  xxx();
  ```

  xxx定义之后就可以立即执行，使用立即执行函数就可以在调用的时候不用再实例化(new)一个对象了：

  ```javascript
  <input type="button" value="test" onclick="NS.alert();"/>
          
          <script type="text/javascript">
              (function namespace(){
                  this.alert=function(){
                      console.log('test');
                  }
                  
                  window.NS=this;
              })();
          </script>
  ```

  因为通过windows.NS=this的执行就建立了一个可用的对象。

  ### 3. 使用prototype

  javascript中的prototype简而言之就是一个类实例化就有的方法（相当于构造函数）

  ```javascript
  <input type="button" value="test" onclick="NS.alert();"/>
  
  (function(){
                  var _NS=function(){
                  
                  }
                  _NS.prototype.alert=function(){
                      console.log('test');
                  }
                  window.NS=new _NS();
              })();
  ```


- 私有变量/函数

  里面的所有东西作用域只在函数内部,就是一个单纯的函数

  ```javascript
  function Obj(){
                  var a=0; //私有变量
                  var fn=function(){ //私有函数
                      
                  }
              }
  ```

  这样在函数对象Obj外部无法访问变量a和函数fn，它们就变成私有的，只能在Obj内部使用，即使是函数Obj的实例仍然无法访问这些变量和函数

  ```javascript
  var o=new Obj();
              console.log(o.a); //undefined
              console.log(o.fn); //undefined
  ```

- 静态变量/函数

  ```javascript
  function Obj(){
                  
              }
              
              Obj.a=0; //静态变量
              
              Obj.fn=function(){ //静态函数
                      
              }
              //这个函数就相当于var Obj=function Obj(){...}
              
              console.log(Obj.a); //0
              console.log(typeof Obj.fn); //function
              
              var o=new Obj();
              console.log(o.a); //undefined
              console.log(typeof o.fn); //undefined
  ```

  相当于实例化了一个独一无二的Obj对象，相当于static final对象，如果再建立一个Obj的话，那么就不是这个Obj了

- 实例变量/函数

  真正的类：

  ```javascript
  function Obj(){
                  this.a=[]; //实例变量
                  this.fn=function(){ //实例方法
                      
                  }
              }
              
              console.log(typeof Obj.a); //undefined
              console.log(typeof Obj.fn); //undefined
              
              var o=new Obj();
              console.log(typeof o.a); //object
              console.log(typeof o.fn); //function
  
  ```

  然而：

  ```javascript
  function Obj(){
                  this.a=[]; //实例变量
                  this.fn=function(){ //实例方法
                      
                  }
              }
              
              var o1=new Obj();
              o1.a.push(1);
              o1.fn={};
              console.log(o1.a); //[1]
              console.log(typeof o1.fn); //object
              var o2=new Obj();
              console.log(o2.a); //[]
              console.log(typeof o2.fn); //function
  ```


## 2. Prototype

[参考](https://blog.csdn.net/i10630226/article/details/47984729)

对于实例对象，如果需要<font color=#FF0033>**给所有实例化的类一些统一值或者函数**</font>。那么就需要prototype函数.

```javascript
function Person(name){
                this.name=name;
            }
            
            Person.prototype.share=[];
            
            Person.prototype.printName=function(){
                alert(this.name);
            }
            
            var person1=new Person('Byron');
            var person2=new Person('Frank');
            
            person1.share.push(1);
            person2.share.push(2);
            console.log(person2.share); //[1,2]

```

实际上当代码读取某个对象的某个属性的时候，都会执行一遍搜索，目标是具有给定名字的属性，搜索首先从对象实例开始，如果在实例中找到该属性则返回，如果没有则查找prototype，如果还是**没有找到则继续递归prototype的prototype对象，直到找到为止**，如果递归到object仍然没有则返回错误。同样道理**如果在实例中定义如prototype同名的属性或函数，则会覆盖prototype的属性或函数**

```javascript
function Person(name){
                this.name=name;
            }
            
            Person.prototype.share=[];

            var person=new Person('Byron');
            person.share=0;
            
            console.log(person.share); //0而不是prototype中的[]

```

