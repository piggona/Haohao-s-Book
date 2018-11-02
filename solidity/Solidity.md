
# Solidity测试环境的安装配置

# 1. Atoms中的环境配置
### a) Atoms下载与安装
解压压缩包，将Atoms放入application
### b) 安装nodejs环境
正常安装即可
### c) 安装testrpc


```python
$ > npm install -g ethereumjs-testrpc truffle
```

执行testrpc命令即可建立拥有是个地址的虚拟测试链环境
### d) truffle框架的部署与标准化配置
#### i. 基本命令


```python
$> truffle init # 初始化truffle工程
$> truffle create contract HelloWorld # 建立一个新的智能合约sol文件
$> truffle compile # 编译sol文件 
$> truffle migrate #部署智能合约
```

#### ii. 框架文件结构
    主文件夹/ ____
    
        contracts/ ____ 
        #存放sol文件，也就是智能合约原始代码的文件夹
    
        build/contracts/ ____
        #存放compile之后的合约
    
        migrations/ ____ 
        #部署智能合约
    
        test/ ____ 
        # 测试智能合约的代码，js&sol测试
    
        truffle.js 
        # Truffle的基本配置设置

### e) truffle+testrpc环境调试流程
#### i. truffle工程的建立


```python
$> truffle init
$> truffle create contract HelloWorld
```

#### ii. 智能合约的编写


```python
# contracts/HelloWorld.sol

pragma solidity ^0.4.0;


contract HelloWorld {
  function sayHello() public pure returns (string) {
    return ("Hello World");
  }
}
```

编写完智能合约之后，要配置部署设置。


```python
# migrations/ 2_deploy_contracts.js

var HelloWorld = artifacts.require("HelloWorld");  #先声明包含一个合约的变量

module.exports = function(deployer) {
    deployer.deploy(HelloWorld);
};
```

#### iii. 智能合约的编译与部署
<em>其中要注意配置tuffle与testrpc的连接</em>

主要指定testrpc的端口:8545


```python
# truffle.js

module.exports = {
     networks: {
         development: {
             host: "localhost",
             port: 8545,
             network_id: "*" // 匹配任何network id
          }
     }
 };
```

之后即可进行编译部署


```python
$> truffle compile
$> truffle migrate
```

#### iv. 使用truffle console与合约互动
<em>首先获得合约的Instance(实例)<em>


```python
HelloWorld.deployed.then(instance => contract = instance)

相当于ES5的
HelloWorld.deployed.then(function(instance){
    contract = instance;
    });
```

之后使用得到的实例进行互动


```python
contract.sayHello()

'Hello World'
```

#### v. 加入新方法之后的重新编译运行
.sol文件中的合约加入新方法之后，需要重新编译运行时，有两种方法：

  1.    


```python
$> truffle compile
$> truffle migrate --reset
```

此时就相当于重新部署了链，会导致数据的丢失

2. 如果想只更新链上的程序，则需要改写migrations中的脚本。 

# 2. Solidity的继承与接口语法
主要的关键字为:


```python
is
interface
```

与JAVA非常类似，示例合约代码：

**结构**

可以参照JAVA设计模式的策略模式。但本例没有实现策略模式的结构。

按照策略模式，一个接口可以由多个实现，如本例中的Bank is Regulator.

在操作类中定义一个Regulator reg

之后就可以reg= new Bank()//或者new为别的接口实现

实现动态的改变实现类型


```python
pragma solidity ^0.4.4;

interface Regulator { //定义的接口
  function checkValue(uint amount) external returns (bool);
  function loan() external returns (bool);
}

contract Bank is Regulator  {
  uint private value;
  address private owner;

  modifier ownerFunc() { //Access Modification
    require(owner == msg.sender); // 设定合约执行的前提，
    _;

  }

  constructor (uint amount) public {
    value = amount;
    owner = msg.sender;
  }

  function deposit(uint amount) public ownerFunc{
    value += amount;
  }

  function withdraw(uint amount) public ownerFunc{
    if (checkValue(amount)) {
      value -= amount;
    }

  }

  function balance() public view returns (uint) {
    return value;
  }

  function checkValue(uint amount) public returns (bool) {
    return value >= amount;
  }

  function loan() public view returns (bool) {
    return value > 0;
  }

}

contract MyFirstContract is Bank(10){

  string private name;
  uint private age;


  function setName(string newName) public {
    name = newName;
  }

  function getName() public view returns (string) {
    return name;
  }

  function setAge(uint newAge) public {
    age = newAge;
  }

  function getAge() public view returns (uint) {
    return age;
  }

}
```

# 3. Solidity的Access Modifier与异常处理
目标：建立一个modifier:只允许合约的建立者添加或移除也可以说从我们建立的银行存款与取款。
1. 建立一个私有的变量->owner
address是以太坊中特有的地址，通过地址就可以识别消息的sender.
接下来探究sender中有什么样的data(以太坊链中的重要信息,当消息发送者使用contract，contract就可以访问owner的信息）
首先建立关于Bank的私有变量 ·


```python
      address private owner；
```

之后再构造函数constructor ()中建立一个address的实例


```python
    owner = msg.sender;
```

要使用这个owner变量，我们将建立一个modifier(即自定义一个修饰符），在里面定义一些错误处理(Error Handling)的算法


```python
    modifier ownerFunc {
        require(owner == msg.sender);
        _; # "_;"符号表示剩下的程序执行
      }
```

定义这个修饰符表示：用这个自定义modifier修饰的函数将先执行 require(owner == msg.sender); 之后才会执行剩下的程序

2. solidity的三种异常处理（检查条件）方式
    - 不用throw关键字的原因：之后版本会移除。
    - 当进行异常处理时要试图使用：require,revert或者assert

    a) **assert**：

    more of validating your <font color=#FF0033>input or functional input </font>at runtime
    它发现错误之后会执行并消耗gas,一般使用assert去<font color=#FF0033>标明/处理合约内部的错误</font>，如用户输入了编码者没有想象到的输入域，用户进行了合约约束范围外的操作。

    <em>例子：要保证在消息释放的时候要<font color=#FF0033>消耗gas</font>,此时可以使用assert（通常用于测试内部错误）</em>

    b) <font color=#FF0033>**require（使用频繁）**</font>:

    more of a parameter requirements(对于变量的限制）

    合约编写者有意识的限制（编写者预料到的不允许的用户操作）
    - 确认有效条件，例如输入，
    - 确认合约声明变量是一致的
    - 从调用到外部合约返回有效值

    c) **revert**:

    标记错误并回滚当前的调用



```python
if (...) 
{
    revert()
} #如果。。。则回滚事务
```

3. 使用modifier:直接在函数实现后面加上modifier的名字即可


```python
function deposit(uint amount) public ownerFunc{
    value += amount;
  }
```

    本例中加入ownerFunc就表明，只有持有msg.sender才可以使用deposit函数（通过执行require(owner == msg.sender))

# 4. Solidity的Imports and Libraries

## import功能
solidity中可以使用import关键词来引用其它文件或网络文件的contract或function


```python
import "文件相对路径"
#之后即可引用文件中的元素
```

## Library功能
Library功能可以给变量添加可以调用的方法
### 首先要定义Library:


```python
# libraries.sol

pragma solidity ^0.4.0;

library IntExtended {
  function increment(uint _self) public pure returns (uint) {
    return _self+1;
  }

  function decrement(uint _self) public pure returns (uint) {
    return _self-1;
  }

  function incrementByValue(uint _self,uint _value) public pure returns (uint){
    return _self + _value;
  }

  function decrementByValue(uint _self,uint _value) public pure returns (uint) {
    return _self + _value;
  }
}
```

### 之后在contract中使用Library:


```python
#testLibrary.sol

pragma solidity ^0.4.0;

import "Libraries.sol";

contract TestLibrary {
  constructor() public {

  }
  using IntExtended for uint; #为uint类型加上Library中定义的方法

  function testIncrement(uint _base) public pure returns (uint) {
    return _base.increment();
  }

  function testDecrement(uint _base) public pure returns (uint) {
    return _base.decrement();
  }

}
```

# 5. Solidity的事件监听与事务（Event logging and Transaction Information)
目标：建立一个合约 ，用户可以直接向它请求一个事务而不是需要自己设定一个函数来执行业务逻辑

## Fallback method
<em>interact with this function without actually having to specify the function</em>

如果在链上的人知道我们合约的地址，可以直接将以太币支付给我们的合约账户并执行这个function.


```python
function () payable {}
```

    那怎样执行这个function?
    需要使用事件监听的功能。

## 事件监听Event Logging


```python
# 定义一个event来对指定的类型进行监听

event SenderLogger(address)
event ValueLogger(uint)
```


```python
# 在Fallback method中，注册实例化监听器

function () public payable isOwner validValue{ #在其中也加入了限制传入消息的modifier
    emit SenderLogger(msg.sender);
    emit ValueLogger(msg.value);
  }
```

完整合约代码：


```python
pragma solidity ^0.4.22;

/*功能在于：当一个用户部署transaction合约，那他可以在任何时候发送以太币到合约中来（而不是调用函数的命令）
之后便开始执行fallback函数
*/
contract transaction {
  event SenderLogger(address);
  event ValueLogger(uint);

  address private owner;

  modifier isOwner {
    require(owner == msg.sender);
    _;
  }

  modifier validValue {
    assert(msg.value >= 1 ether);
    _;
  }

  constructor() public {
    owner = msg.sender;
  }

  function () public payable isOwner validValue{
    emit SenderLogger(msg.sender);
    emit ValueLogger(msg.value);
  }

}
```

# 6. Solidity的合约结构
<em> Each contract can contain declarations of <font color=#F57C00>State Variables,Functions,Function Modifiers,Event,Struct Types and Enum Types</font></em>

## State Variables(基本变量类型）
- bool：布尔型

- int/uint：整形与无符号整形

其中可以使用如int8,int256去设定变量占用的位

- fixed/ufixed：定点数，相当于demical

通常使用 $$ fixedM*N (M为此类型占用的bit数，N为此类型的定点） $$
- address:地址类型，20字节

其中address类型可以被使用与查询余额及发送ether到这个账户的操作。

address的方法：


```python
# address.balance方法就是查询这个账户余额
# address.transfer(n)方法就是将n个wei发送到address中

address x = 0x123;
address myAddress this;

if (x.balance < 10 && myAddress.balance >= 10) x.transfer(10)

# 使用call方法可以向指定的地址发送信息
# 但要注意若第一个参数为四个字节，则会默认被认为是函数签名的序号值

address.call("register", 256) #这个就表示向address传送了"register"+256这样的数据

# 可以使用call来控制合约的gas提供量与Ether提供量

address.call.gas(10000).value(1 ether)("register",256)

```

- Fixed-size byte arrays:定长的比特串

使用bytes1,bytes2,...,bytes32来定义。

其中的操作可以类比于java中的字符串操作

- Dynamically-sized byte array:变长的比特串

string

bytes

- enum：枚举类型


```python
enum ActionChoices { Goleft,GoRight, GoStraight, SitStill } # 定义

function setGoStraight() public { # 使用
    choice = ActionChoices.GoStraight;
}
```

- Array:数组类型

类似于java的数组


```python
# 两种为Array申请空间的方法

contract c {
    function f(uint len) public pure {
        uint[] memory a = new uint[](7);
        bytes memory b = new bytes(len)
        # a.length = 7,b.length == len
        a[6] = 8;
    }
}

# 其中array的方法有：length,push与pop
# push可以用于在数组的最后加上一个元素
# pop则弹出数组最后一个元素
```

## Structs:自定义类型
<em>首先要介绍Mappings</em>

### Mappings(映射):

$$ mapping(\_KeyType => \_ValueType) $$

    其中_KeyType可以是除了Mapping类型之外的各种类型（相当于key)：dynamically sized array,contract, enum或struct
    _ValueType可以是任何类型(相当于value)，包括mapping


```python
#添加字典元素

mapping(uint = > uint) public intMapp;
mapping(uint => mapping(uint => string)) public mapMapp;

function set() {
    intMapp[1] = 100;
    mapMapp[2][2] = "aaa"
}
```

### Structs:

一个自定义类型，可以作为mapping与arrays的元素存在，也可以自身内部包含mapping与array。

<font color=#FF0033>不可以自己包含自己，但可以包含自己类型的Array或mapping类型。</font>

### 对于Variables的constant声明：
声明表示变量是静态的，不可改变。
如：


```python
contract C {
    uint constant x =32**22 + 8;
    string constant text = "abc";
    bytes32 constant myHash = keccak256("abc")
}
```

## Functions:合约方法

<font color=#1E88E5>function ( \< parameter types \> ) {public|private|internal|external} [pure|view|payable] [returns (< return types >) ] </font>

### Function的修饰符
<em>function的修饰符主要包括可见性的修饰及功能的修饰</em>
#### 可见性修饰
- external:
添加这个修饰符之后，在合约contract中如果需要调用这个方法(function)如f()，则需要将表达方式改为this.f()才可以进行调用。
- internal:
这个修饰符表示这个方法是合约contract内部的方法。可以直接在合约内部进行调用
- public:
这个修饰符表示方法可以被合约contract内部调用也可以使用messages来访问。
- private:
这个修饰符表示方法只能在合约内部被调用。

     虽然private不能被其它合约访问及修改，但依旧可见，以为它在链上。

#### 功能修饰
- view:表示这个方法 不改变<font color=#FF0033>账户的状态</font>

      什么操作会改变账户的状态？
      1. Writing to state variables
      2. Emitting events
      3. Creating other contracts
      4. Using selfdestruct
      5. Sending Ether via calls.
      6. Calling any function not marked view or pure
      7. Using low-level calls
      8. Using inline assembly that contains certain opcodes(使用内联的汇编，其中有操作账户状态的操作）
- pure:表示这个方法不读也不改变账户的状态   
- payable:表示这个方法可以用来接收ether

### Fallback Function
<em>一个协议只能有一个fallback function,用来处理协议中其它方法无法处理的消息，如收到的Ether(with out data)</em>

但若将fallback function设计成接收Plain Ether，需要将该函数加上payable修饰符

## Event:事件
<em>主要关键词：event   emit</em>

在solidity中使用event来定义一个事件，使用emit来触发这个事件。

    在Dapp应用中，如果监听了某事件，当事件发生时，会进行回调。


```python
pragma solidity ^0.4.21;

contract ClientReceipt {
    event Deposit(
        address indexed _from,
        bytes32 indexed _id,
        uint _value
    );

    function deposit(bytes32 _id) public payable {
        // Events are emitted using `emit`, followed by
        // the name of the event and the arguments
        // (if any) in parentheses. Any such invocation
        // (even deeply nested) can be detected from
        // the JavaScript API by filtering for `Deposit`.
        emit Deposit(msg.sender, _id, msg.value);
    }
}
```

<font color=#0097A7>当调用deposit这个方法时，就会触发这个事件。</font>

日志与事件在合约内是无法访问的，即使是创建日志的合约

要使用事件触发方法，需要与web3.js交互，在web3.js中监听并操作：


```python
var abi = /* abi as generated by the compiler */;
var ClientReceipt = web3.eth.contract(abi);
var clientReceipt = ClientReceipt.at("0x1234...ab67" /* address */);

var event = clientReceipt.Deposit();

// watch for changes
event.watch(function(error, result){
    // result will contain various information
    // including the arguments given to the `Deposit`
    // call.
    if (!error)
        console.log(result);
});

// Or pass a callback to start watching immediately
var event = clientReceipt.Deposit(function(error, result) {
    if (!error)
        console.log(result);
});
```

## 继承与多态
- is关键词建立继承关系，可以使用父合约中的internal变量与方法
- 构造函数constructor
- 多继承


```python
contract X {}
contract A is X {}
contract C is A,X {}

# The reason for this is that C requests X to override A (by specifying A, X in this order), 
# but A itself requests to override X, which is a contradiction that cannot be resolved.
```

- Abstract Contracts:抽象合约
- 接口
- Libraries进阶了解

<em>Library的主要作用就是给某个类型赋予Library中扩展的方法。</em>

在LIbrary中定义所要拓展的方法，之后使用using...for...语法进行类型扩展绑定。（详情见前面Library的例子）

Library使用的注意事项：
1. Library不能定义接收Ether的方法
2. using A for *表示Library A中的方法拓展给了所有类型。

## Contract操作
### 使用new关键词,像创建类一样创建Contract实例

<em>在一个contract对象中可以使用<font color=#FF0033>new关键字</font>创建一个合约的实例。</em>

用这种方式可以实现策略模式(类似java)


```python
pragma solidity >0.4.24;

contract D {
    uint x;
    constructor(uint a) public payable {
        x = a;
    }
}

contract C {
    D d = new D(4); # will be executed as part of C's constructor

    function createD(uint arg) public {
        D newD = new D(arg);
    }

    function createAndEndowD(uint arg, uint amount) public payable {
        # Send ether along with the creation
        D newD = (new D).value(amount)(arg);
    }
}
```

### Function calls:在合约中/间调用方法
#### Internal Function Calls：内部调用
直接在方法中调用方法即可
#### External Function Calls：外部调用
先要在本contract中定义一个要调用方法所在contract的对象，之后再调用外部contract的方法。

<em><font color=#FF0033>要注意contract的限定词</font></em>


```python
pragma solidity ^0.4.0;

contract InfoFeed {
    function info() public payable returns (uint ret) { return 42; }
}

contract Consumer {
    InfoFeed feed;
    function setFeed(address addr) public { feed = InfoFeed(addr); }
    function callFeed() public { feed.info.value(10).gas(800)(); }
}
```

## Solidity中的特有元素及系统设定全局变量
### Ether Unit
### Time Unit
### 特殊变量及可以调用的方法：
#### 1. 区块及事务元素
#### 2. ABI编码方法
#### 3. 错误处理方法
#### 4. 数学加密方法
#### 5. 关于地址的方法
#### 6. 合约相关的
