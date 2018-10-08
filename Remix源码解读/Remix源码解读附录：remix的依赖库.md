# Remix源码解读附录：remix的依赖库

## 1. ethereumjs-vm

1. 建立简单对象及应用的方式：

   ```javascript
   var VM = require('ethereumjs-vm')
   
   //create a new VM instance
   var vm = new VM()
   var code = '7f4e616d65526567000000000000000000000000000000000000000000000000003055307f4e616d6552656700000000000000000000000000000000000000000000000000557f436f6e666967000000000000000000000000000000000000000000000000000073661005d2720d855f1d9976f88bb10c1a3398c77f5573661005d2720d855f1d9976f88bb10c1a3398c77f7f436f6e6669670000000000000000000000000000000000000000000000000000553360455560df806100c56000396000f3007f726567697374657200000000000000000000000000000000000000000000000060003514156053576020355415603257005b335415603e5760003354555b6020353360006000a233602035556020353355005b60007f756e72656769737465720000000000000000000000000000000000000000000060003514156082575033545b1560995733335460006000a2600033545560003355005b60007f6b696c6c00000000000000000000000000000000000000000000000000000000600035141560cb575060455433145b1560d25733ff5b6000355460005260206000f3'
   
   vm.runCode({
     code: Buffer.from(code, 'hex'), // code needs to be a Buffer
     gasLimit: Buffer.from('ffffffff', 'hex')
   }, function(err, results){
     console.log('returned: ' + results.return.toString('hex'));
   })
   ```

2. API

   - new VM([opts])新建一个vm对象

     | 项目                       | 注释                               |
     | -------------------------- | ---------------------------------- |
     | stateManager               | 一个`state manager`的实例          |
     | state                      | 一个状态树的merckle patricle树实例 |
     | blockchain                 | 一个用于存储/搜索的blockchain对象  |
     | chain                      | VM运行的链环境(默认是mainnet)      |
     | hardfork                   | 要使用的hardfork规则               |
     | activatePrecompiles        |                                    |
     | allowUnlimitedContractSize |                                    |

   - vm方法

     - `vm.runBlockchain(blockchain,cb)`

       运行里面包含的`blocks`（注意复数）并将它们对应的blockchain中

       - `blockchain`-需要运行的blockchain对象
       - `cb`- 回调函数，当出错时会返回一个err对象

     - `vm.runBlock(opts,cb)`(未完成)

       运行1.` block`对象中的所有事务并2. 更新矿工账户的余额（状态）

       - `opts.block`-需要运行的Block对象
       - `opts.generate`-一个布尔类型的变量；是否建立stateRoot树
       - `cb` - 回调函数，包含两个变量。
         - result:
           - receipts
           - results
         - error:

     - `vm.runTx(opts,cb)`

       运行一个事务

       - `opts.tx` - 需要运行的事务对象
       - `opts.block` - tx(事务)对象所属的block，默认情况下将会使用一个空的block
       - `cb` - 回调函数，包含两个变量。
         - result:
           - `amountSpent`：一个`bigNum`类型的数，表示这个事务花费的ether
           - `gasUsed`：这个事务花费的gas
           - `gasRefund`：最终退回的gas,`gasUsed = totalGasConsumed - gasRefund`
           - `vm` ： 进行vm.runCode之后的结果
         - error:返回错误

     - `vm.runCode(opts,cb)`

       运行EVM code

       - `opts.code` - 需要运行的EVM code，传送方式为Buffer
       - `opts.data` - 输入的数据
       - `opts.value` - 被传送到`opt.address`的ether数量，默认为0
       - `opts.block` - tx属于的Block
       - `opts.gasLimit` - 运行事务所设置的gas limit
       - `opts.account` - executing code所属的Account对象
       - `opts.address` - execute这个code的account地址，需要以byte Buffer的形式传送。
       - `opts.origin` - call操作的源头地址，地址需要以20bit的Buffer形式传送
       - `opts.caller` - 运行(ran)code的地址，需要以20bit Buffer的形式进行传送。
       - `cb` - 回调函数
         - `gas` - 剩余的gas
         - `gasUsed` - 使用的gas
         - `gasRefund` - 退回的gas
         - `selfdestruct` - 
         - `logs` - Array格式，合约返回的log
         - `exception` - 0：合约报错。1：合约运行正常
         - `exceptionError` - 如果有错误，就会返回一个string类型的exception解释文件
         - `return` - Buffer类型的数据，返回contract返回的信息。

   - vm事件

## 2. ethereumjs-tx

> A simple module for creating,manupulating and signing ethereum transactions
>
> 简单的用于**建立**，**操作**，**签名**EHT 事务的模块

官方实例：

```javascript
const EthereumTx = require('ethereumjs-tx')
const privateKey = Buffer.from('e331b6d69882b4cb4ea581d88e0b604039a3de5967688d3dcffdd2270c0fd109', 'hex')

const txParams = {
  nonce: '0x00',
  gasPrice: '0x09184e72a000', 
  gasLimit: '0x2710',
  to: '0x0000000000000000000000000000000000000000', 
  value: '0x00', 
  data: '0x7f7465737432000000000000000000000000000000000000000000000000000000600057',
  // EIP 155 chainId - mainnet: 1, ropsten: 3
  chainId: 3
}

const tx = new EthereumTx(txParams)
tx.sign(privateKey)
const serializedTx = tx.serialize()
```

从官方实例中可以发现，该模块<font color=#FF0033>用于将已经编码的私钥(hex,16进制串)及事务（json)</font>进行**签名并序列化（serialize）**发送。

## 3. ethers

- ## Wallet and Signers(钱包及签名)：

  > Wallet用来管理用户的公/私钥对，并对transactions（事务）进行签名与证明在以太坊网络中用户的所有权

  - Wallet(钱包)

    - `new Wallet(privateKey[,provider])`:通过私钥与指定的provider(可选)来建立一个Wallet实例。

      ```javascript
      let privateKey = "0x0123456789012345678901234567890123456789012345678901234567890123";
      let wallet = new ethers.Wallet(privateKey);
      
      // Connect a wallet to mainnet
      let provider = ethers.getDefaultProvider();
      let walletWithProvider = new ethers.Wallet(privateKey, provider);
      ```

    - `Wallet.createRandom([options])`：随机创建一个钱包实例。

      - options可以选择extraEntropy：用于调整添加建立随机数的熵

      ```javascript
      let randomWallet = ethers.Wallet.createRandom();
      ```

    - `Wallet.fromEncryptedJson(json,password[,progressCallback])`:使用加密的Json Wallet(如从Geth,parity,Crowsale tools或由prototype.encrypt)建立钱包

      ```javascript
      let data = {
          id: "fb1280c0-d646-4e40-9550-7026b1be504a",
          address: "88a5c2d9919e46f883eb62f7b8dd9d0cc45bc290",
          Crypto: {
              kdfparams: {
                  dklen: 32,
                  p: 1,
                  salt: "bbfa53547e3e3bfcc9786a2cbef8504a5031d82734ecef02153e29daeed658fd",
                  r: 8,
                  n: 262144
              },
              kdf: "scrypt",
              ciphertext: "10adcc8bcaf49474c6710460e0dc974331f71ee4c7baa7314b4a23d25fd6c406",
              mac: "1cf53b5ae8d75f8c037b453e7c3c61b010225d916768a6b145adf5cf9cb3a703",
              cipher: "aes-128-ctr",
              cipherparams: {
                  iv: "1dcdf13e49cea706994ed38804f6d171"
               }
          },
          "version" : 3
      };
      
      let json = JSON.stringify(data);
      let password = "foo";
      
      ethers.Wallet.fromEncryptedWallet(json, password).then(function(wallet) {
          console.log("Address: " + wallet.address);
          // "Address: 0x88a5C2d9919e46F883EB62F7b8Dd9d0CC45bc290"
      });
      ```

    - `Wallet.fromMnemonic(mnemonic[,path="m/44'/60'/0'/0/0"[,wordlist]])`:通过用户提供的助记词与**mnemonic deriving path**建立一个满足BIP-039+BIP-044协议的钱包

      ```javascript
      let mnemonic = "radar blur cabbage chef fix engine embark joy scheme fiction master release";
      let mnemonicWallet = ethers.Wallet.fromMnemonic(mnemonic);
      
      // Load the second account from a mnemonic
      let path = "m/44'/60'/1'/0/0";
      let secondMnemonicWallet = ethers.Wallet.fromMnemonic(mnemonic, path);
      
      // Load using a non-english locale wordlist (the path "null" will use the default)
      let secondMnemonicWallet = ethers.Wallet.fromMnemonic(mnemonic, null, ethers.wordlists.ko);
      ```

      其中的wordlist:

      | Language             | node.js                  | Browser               |
      | -------------------- | ------------------------ | --------------------- |
      | English(US)          | `ethers.wordlists.en`    | included              |
      | Italian              | `ethers.wordlists.it`    | `dist/wordlist-it.js` |
      | Japanese             | `ethers.wordlists.ja`    | `dist/wordlist-ja.js` |
      | Korean               | `ethers.wordlists.ko`    | `dist/wordlist-ko.js` |
      | Chinese(simplified)  | `ethers.wordlists.zh_cn` | `dist/wordlist-zh.js` |
      | Chinese(traditional) | `ethers.wordlists.zh_tw` | `dist/wordlist-zh.js` |

    - `prototype.connect(provider)`:通过一个已经存在的实例建立一个Wallet实例，连接到一个新的provider中。

  - Prototype(Wallet类的原型方法：每个Wallet对象公有的方法)

    - `prototype.address`:钱包的公开地址
    - `prototype.privateKey`:钱包的私钥
    - `prototype.provider`:钱包所对应的provider
    - `prototype.mnemonic`:钱包对应的助记词
    - `prototype.path`:钱包对应的mnemonic path

  - Signing(钱包的签名操作)：

    - `prototype.sign(transaction)`：为一个事务对象进行签名，并且返回一个包含签名后16进制字符串的Promise对象。签名之后就可以使用sendTransaction方法进行事务发送到以太坊网络。

      其中transcation对象包含的字段为：

      ```javascript
      let transaction ={
          nonce : 0,//当前节点对该交易的版本号，为了避免交易重播，使用getTransactionCount()函数可以获取已知上一笔交易的nonce值
          gasLimit : 21000,//gas最多使用限制(抵押的gas)
          gasPrice : utils.bigNumberify("20000000000"),//gas的价格
          
          to : "0x88a5C2d9919e46F883EB62F7b8Dd9d0CC45bc290",
          data : "0x",
          value : utils.parseEther("1.0"),
          // 保证交易不能被多个链所执行
          chainId : ethers.utils.getNetwork('homestead').chainId
      }
      ```

      [nonce解析](https://mp.weixin.qq.com/s?__biz=MzI0NDAzMzIyNQ==&mid=2654065324&idx=1&sn=58ff35023fa61aea3ae2782ebac06fdc&chksm=f2a6810ac5d1081c1dc2cb2e6e1f1a5d697fe936861f3c1aebb69b84d43957b8afe4f055ebc0#rd)

      ```javascript
      let privateKey = "0x3141592653589793238462643383279502884197169399375105820974944592"
      let wallet = new ethers.Wallet(privateKey)
      
      console.log(wallet.address)
      // "0x7357589f8e367c2C31F51242fB77B350A11830F3"
      
      // All properties are optional
      let transaction = {
          nonce: 0,
          gasLimit: 21000,
          gasPrice: utils.bigNumberify("20000000000"),
      
          to: "0x88a5C2d9919e46F883EB62F7b8Dd9d0CC45bc290",
          // ... or supports ENS names
          // to: "ricmoo.firefly.eth",
      
          value: utils.parseEther("1.0"),
          data: "0x",
      
          // This ensures the transaction cannot be replayed on different networks
          chainId: ethers.utils.getNetwork('homestead').chainId
      }
      
      let signPromise = wallet.sign(transaction)
      
      signPromise.then((signedTransaction) => {
      
          console.log(signedTransaction);
          // "0xf86c808504a817c8008252089488a5c2d9919e46f883eb62f7b8dd9d0cc45bc2
          //    90880de0b6b3a76400008025a05e766fa4bbb395108dc250ec66c2f88355d240
          //    acdc47ab5dfaad46bcf63f2a34a05b2cb6290fd8ff801d07f6767df63c1c3da7
          //    a7b83b53cd6cea3d3075ef9597d5"
      
          // This can now be sent to the Ethereum network
          let provider = ethers.getDefaultProvider()
          provider.sendTransaction(signedTransaction).then((tx) => {
      
              console.log(tx);
              // {
              //    // These will match the above values (excluded properties are zero)
              //    "nonce", "gasLimit", "gasPrice", "to", "value", "data", "chainId"
              //
              //    // These will now be present
              //    "from", "hash", "r", "s", "v"
              //  }
              // Hash:
          });
      })
      ```

    - `prototype.signMessage(message)`：对一个消息进行签名，会返回一个包含签名后的`flat-format`串的Promise对象.

      > 对于签名后得到的signature，有两种格式，即flat-format与Expanded-format(r,s,v)：
      >
      > 对于交易/事务签名->[签名&验证](https://zhuanlan.zhihu.com/p/30481292)
      >
      > [对签名结果进行处理的工具函数](https://docs.ethers.io/ethers.js/html/api-utils.html#signature)

      signMessage方法可以对text类型及binary类型的数据进行签名：

      如果是text类型(即string类型)的消息，它就可以直接作为UTF8 bytes传送。

      如果是其它数据类型（如想传送数组类型的数据）,需要进行[arrayish](https://docs.ethers.io/ethers.js/html/api-utils.html#arrayish)操作才可以进行签名



~~~javascript
  //对text类型的数据进行签名
  //直接返回的是一个flat-format
  //也可以通过utils.splitSignature()函数转化为格式化输出
  
  let privateKey = "0x3141592653589793238462643383279502884197169399375105820974944592"
  let wallet = new ethers.Wallet(privateKey);
  
  // Sign a text message
  let signPromise = wallet.signMessage("Hello World!")
  
  signPromise.then((signature) => {
  
      // Flat-format
      console.log(signature);
      // "0xea09d6e94e52b48489bd66754c9c02a772f029d4a2f136bba9917ab3042a0474
      //    301198d8c2afb71351753436b7e5a420745fed77b6c3089bbcca64113575ec3c
      //    1c"
  
      // Expanded-format
      console.log(ethers.utils.splitSignature(signature));
      // {
      //   r: "0xea09d6e94e52b48489bd66754c9c02a772f029d4a2f136bba9917ab3042a0474",
      //   s: "0x301198d8c2afb71351753436b7e5a420745fed77b6c3089bbcca64113575ec3c",
      //   v: 28,
      //   recoveryParam: 1
      //  }
  });
  ```

  ```javascript
  //对binary类型的数据进行签名
  //需要先使用arrayify函数进行array化之后才可以进行签名
  
  let privateKey = "0x3141592653589793238462643383279502884197169399375105820974944592"
  let wallet = new ethers.Wallet(privateKey);
  
  // The 66 character hex string MUST be converted to a 32-byte array first!
  let hash = "0x3ea2f1d0abf3fc66cf29eebb70cbd4e7fe762ef8a09bcc06c8edf641230afec0";
  let binaryData = ethers.utils.arrayify(hash);
  
  let signPromise = wallet.signMessage(binaryData)
  
  signPromise.then((signature) => {
  
      console.log(signature);
      // "0x5e9b7a7bd77ac21372939d386342ae58081a33bf53479152c87c1e787c27d06b
      //    118d3eccff0ace49891e192049e16b5210047068384772ba1fdb33bbcba58039
      //    1c"
  });
~~~

  - Blockchain Operations(区块链操作)：需要连接provider的wallet操作

    - `prototype.getBalance([blockTag = "latest"])`：返回一个包含当前wallet的余额（表示形式->BigNumber,in wei）的Promise对象。

      ```javascript
      // We require a provider to query the network
      let provider = ethers.getDefaultProvider();
      
      let privateKey = "0x3141592653589793238462643383279502884197169399375105820974944592"
      let wallet = new ethers.Wallet(privateKey, provider);
      
      let balancePromise = wallet.getBalance();
      
      balancePromise.then((balance) => {
          console.log(balance);
      });
      
      let transactionCountPromise = wallet.getTransactionCount();
      
      transactionCountPromise.then((transactionCount) => {
          console.log(transactionCount);
      });
      
      ```

    - `prototype.getTransactionCount([blockTag = "latest"])`：

      > Returns a [Promise](https://docs.ethers.io/ethers.js/html/notes.html#promise) that resovles to the number of transactions this account has ever sent (also called the *nonce*) at the [blockTag](https://docs.ethers.io/ethers.js/html/api-providers.html#blocktag).

      返回包含当前钱包交易数量->nonce值(用于下一个交易验证)的Promise对象。

    - `prototype.estimateGas(transaction)`:传入一个交易对象，返回包含估算该交易的Gas值的Promise对象。

    - `prototype.sendTransaction(transaction)`:

      输入一个满足[Transaction Request](https://docs.ethers.io/ethers.js/html/api-providers.html#transaction-request)的transaction对象，之后返回一个包含[Transaction Response](https://docs.ethers.io/ethers.js/html/api-providers.html#transaction-response)的Promise对象。

      ```javascript
      // We require a provider to send transactions
      let provider = ethers.getDefaultProvider();
      
      let privateKey = "0x3141592653589793238462643383279502884197169399375105820974944592"
      let wallet = new ethers.Wallet(privateKey, provider);
      
      let amount = ethers.utils.parseEther('1.0');
      
      let tx = {
          to: "0x88a5c2d9919e46f883eb62f7b8dd9d0cc45bc290",
          // ... or supports ENS names
          // to: "ricmoo.firefly.eth",
      
          // We must pass in the amount as wei (1 ether = 1e18 wei), so we
          // use this convenience function to convert ether to wei.
          value: ethers.utils.parseEther('1.0')
      };
      
      let sendPromise = wallet.sendTransaction(tx);
      
      sendPromise.then((tx) => {
          console.log(tx);
          // {
          //    // All transaction fields will be present
          //    "nonce", "gasLimit", "pasPrice", "to", "value", "data",
          //    "from", "hash", "r", "s", "v"
          // }
      });
      ```

  - Encrypted Json Wallet(建立加密存储的Json钱包)：

    > 很多系统都提供了为本机的私钥进行加密存储的服务，它有多种不一样的加密方式及编码形式，当然它是可以供人以一定的规则进行异地导入的。

    `prototype.encrypt(password,options,progressCallBack)`：对本地的钱包对象进行加密处理。

    options字段选项：

    - salt：
    - iv：
    - uuid：
    - scrypt:
    - entropy(助记词加密的熵):
    - mnenomic(助记词加密的助记词本身):
    - path(助记词加密所使用的向量):

    其中的progressCallBack函数是一个加密过程监听器，在各阶段过程中返回0或1来显示加密过程状态。

- ## Providers(供应商/节点)：

  >  Provider封装了用户与区块链网络的连接，用于实现查询操作与发送签名后的状态变换事务/交易

  <font color=#FF0033>EtherscanProvider与InfuraProvider</font>提供了连接公用节点的能力而不需要自己本身建立节点。

  <font color=#FF0033>JsonRpcProvider与IpcProvider</font>用于手动连接已知的节点（自己的节点），支持的链类型有：mainnet,testnets,POA nodes或Ganache

  如果想连接web3 应用客户端(如Metamask客户端)，可以使用<font color=#FF0033>providers.Web3Provider()</font>来进行与Web3客户端的连接。

  一般情况下，我们使用<font color=#FF0033>providers.getDefaultProvider()</font>,这样就可以持续与一个EtherScan或Infura节点保持连接

  例子：

  - 默认情况：

    ```javascript
    // You can use any standard network name
    //  - "homestead"
    //  - "rinkeby"
    //  - "ropsten"
    //  - "kovan"
    
    let provider = ethers.getDefaultProvider('ropsten');
    ```

  - 连接Web3客户端：

    ```javascript
    // The network will be automatically detected; if the network is
    // changed in MetaMask, it causes a page refresh.
    
    let provider = new ethers.providers.Web3Provider(web3.currentProvider);
    ```

  Provider对象的用途：

  - 用于作为连接Wallet的参数对象:

    ```javascript
    let defaultProvider = ethers.getDefaultProvider('ropsten');
    let wallet = new Wallet(secretKey,defaultProvider);
    ```

  - 用于查询当前网络的状态：

    - `prototype.getNetwork()`:

      返回一个包含Network对象（包含网络信息）的Promise对象。

  - 用于向链上发送查询:

    - `prototype.getBalance(addressOrName,[blockTag = "latest"])`=>Promise<BigNumber>:

      查询某个地址的账户余额。

      例：

      ```javascript
      let address = "0x02F024e0882B310c6734703AB9066EdD3a10C6e0";
      
      provider.getBalance(address).then((balance) => {
      
          // balance is a BigNumber (in wei); format is as a sting (in ether)
          let etherString = ethers.utils.formatEther(balance);
      
          console.log("Balance: " + etherString);
      });
      ```

    - `prototype.getTransactionCount(addressOrName,[blockTag = "latest"])`=>Promise<number>:

      查询某个地址(账户)的交易计数值(nonce)

      例：

      ```javascript
      
      ```

    - `prototype.getBlockNumber()`=>Promise<number>:最近的block number

    - `prototype.getGasPrice()`=>Promise<BigNumber>:返回BigNumber类型的当前Gas Price

    - `prototype.getBlock(blockHashOrBlockNumber)`=>Promise<Block>:返回[Block Response](let address = "0x02F024e0882B310c6734703AB9066EdD3a10C6e0";provider.getTransactionCount(address).then((transactionCount) => {    console.log("Total Transactions Ever Sent: " + transactionCount);})对象;

    - `prototype.getTransaction(transactionHash)`=>Promise<Transaction Response>:返回[Transaction Response](https://docs.ethers.io/ethers.js/html/api-providers.html#transaction-response)对象；

    - `prototype.getTransactionReceipt(transactionHash)`=>Promise<TransactionReceipt>:返回[Transaction Receipts](https://docs.ethers.io/ethers.js/html/api-providers.html#transaction-receipt)对象；

  - 对链上合约的操作：

    - `prototype . call ( transaction ) `  =>   Promise<hex> :返回合约函数的返回值（16进制的值）<font color=#FF0033>==注意data的引入方式==</font>

      ```javascript
      // See: https://ropsten.etherscan.io/address/0x6fc21092da55b392b045ed78f4732bff3c580e2c
      
      // Setup a transaction to call the FireflyRegistrar.fee() function
      
      // FireflyRegistrar contract address
      let address = "0x6fC21092DA55B392b045eD78F4732bff3C580e2c";
      
      // First 4 bytes of the hash of "fee()" for the sighash selector
      let data = ethers.utils.hexDataSlice(ethers.utils.id('fee()'), 0, 4);
      
      let transaction = {
          to: ensName,
          data: data
      }
      
      let callPromise = defaultProvider.call(transaction);
      
      callPromise.then((result) => {
          console.log(result);
          // "0x000000000000000000000000000000000000000000000000016345785d8a0000"
      
          console.log(ethers.utils.formatEther(result));
          // "0.1"
      });
      ```

    - `prototype . estimateGas ( transaction ) `  =>   Promise<BigNumber> :返回输入transaction对象估计的Gas值

    - `prototype . sendTransaction ( signedTransaction ) `  =>   Promise<TransactionResponse> :将已经签名的transaction传播到Ethereum network中并返回transaction的结果对象

      ```javascript
      let privateKey = '0x0123456789012345678901234567890123456789012345678901234567890123';
      let wallet = new ethers.Wallet(privateKey, provider);
      
      let transaction = {
          to: "ricmoo.firefly.eth",
          value: ethers.utils.parseEther("0.1")
      };
      
      // Send the transaction
      let sendTransactionPromise = wallet.sendTransaction(transaction);
      
      sendTransactionPromise.then((tx) => {
         console.log(tx);
      });
      ```

  - 监听链上的事件(Events)

- ## Contracts(合约)：

  Contract(合约)是运行在以太坊区块链上的可执行程序。一个Contract(合约)包含代码本身(被称为byte code)还有长期存储的数据(被称为storage)。每一个被部署的合约有一个地址，用于外部与该合约连接。

  与合约交互可以通过==call方式,sendTransaction方式,也可以通过监听合约emit的Event的方式（Event无法由Contract所监听)==

  > **call合约函数的方式**：
  >
  > Constant-method：
  >
  > - 特征：无法进行add,remove,change data in the storage,log any events.只能对合约中的Constant-method进行call操作
  > - 花费：free
  >
  > Non-Constant-method：
  >
  > - 特征：需要以transaction的方式发送，只有EOA账户（外部账户，与合约账户对应）才可以发送，需要进行挖矿确认之后才会生效，生效（操作)时间由gas price,网络状态以及矿工的优先状态决定
  > - 花费：transaction gas

  例子使用的合约：

  ```javascript
  pragma solidity ^0.4.24;
  
  contract SimpleStorage {
  
      event ValueChanged(address indexed author, string oldValue, string newValue);
  
      string _value;
  
      constructor(string value) public {
          emit ValueChanged(msg.sender, _value, value);
          _value = value;
      }
  
      function getValue() view public returns (string) {
          return _value;
      }
  
      function setValue(string value) public {
          emit ValueChanged(msg.sender, _value, value);
          _value = value;
      }
  }
  ```

  ### 部署合约：

  为了在以太坊网络中部署一个合约，可以通过合约的bytecode与Application Binary Interface(ABI)建立一个ContractFactory对象。

  #### 建立一个ContractFactory对象(Creating a Contract Factory)：

  - `new ethers.ContractFactory(abi,bytecode[,signer])`:

    建立一个新的ContractFactory对象（可以部署的合约对象），在对象的参数中的signer表示用于部署时发送transaction的Signer对象（Wallet对象）

  - `ethers.ContractFacrory.fromSolidity(complilerOutput[,signer])`:

    通过Truffle根据Solidity代码编译的Truffle Json文件建立ContractFactory对象

  - `prototype.connect(signer)` => ContractFactory:

    根据一个已经存在的ContractFactory对象建立一个新的ContractFactory对象，但换了一个新的Signer对象。

  #### Prototype:

  - `prototype.bytecode`
  - `prototype.interface`
  - `prototype.signer`:如果返回为null,就不能调用deploy方法。

  #### 连接建立对象(Connecting):

  - `prototype.attach(address)` => Contract

    根据指定的地址连接到指定的Contract实例，具有特定的Contract Interface及Signer(即已经部署的Contract)

  #### 部署(Deployment)：

  - `prototype.deploy(...)` => Promise<Contract>

    发送建立当前ContractFactory指定的合约的Transaction,返回一个Contract对象。

    需要注意的是，合约有可能不能立即部署完成(mined)，可以通过`contract.deployed`函数来监控部署情况，这个函数会返回一个Promise对象，在合约部署成功时返回一个resolve，失败时就会返回reject

  - `prototype.getDeployTransaction(...)` => UnsignedTransaction

    返回建立当前ContractFactory指定合约需要的Transaction(没有进行签名的)，一般用于进行离线签名 建立合约的transaction。

  部署合约的例子：

  ```javascript
  const ethers = require('ethers');
  
  // The Contract interface
  let abi = [
      "event ValueChanged(address indexed author, string oldValue, string newValue)",
      "constructor(string value)",
      "function getValue() view returns (string value)",
      "function setValue(string value)"
  ];
  
  // The bytecode from Solidity, compiling the above source
  let bytecode = "0x608060405234801561001057600080fd5b506040516105bd3803806105bd8339" +
                   "8101604081815282518183526000805460026000196101006001841615020190" +
                   "91160492840183905293019233927fe826f71647b8486f2bae59832124c70792" +
                   "fba044036720a54ec8dacdd5df4fcb9285919081906020820190606083019086" +
                   "9080156100cd5780601f106100a2576101008083540402835291602001916100" +
                   "cd565b820191906000526020600020905b815481529060010190602001808311" +
                   "6100b057829003601f168201915b505083810382528451815284516020918201" +
                   "9186019080838360005b838110156101015781810151838201526020016100e9" +
                   "565b50505050905090810190601f16801561012e578082038051600183602003" +
                   "6101000a031916815260200191505b5094505050505060405180910390a28051" +
                   "610150906000906020840190610157565b50506101f2565b8280546001816001" +
                   "16156101000203166002900490600052602060002090601f0160209004810192" +
                   "82601f1061019857805160ff19168380011785556101c5565b82800160010185" +
                   "5582156101c5579182015b828111156101c55782518255916020019190600101" +
                   "906101aa565b506101d19291506101d5565b5090565b6101ef91905b80821115" +
                   "6101d157600081556001016101db565b90565b6103bc806102016000396000f3" +
                   "0060806040526004361061004b5763ffffffff7c010000000000000000000000" +
                   "0000000000000000000000000000000000600035041663209652558114610050" +
                   "57806393a09352146100da575b600080fd5b34801561005c57600080fd5b5061" +
                   "0065610135565b60408051602080825283518183015283519192839290830191" +
                   "85019080838360005b8381101561009f57818101518382015260200161008756" +
                   "5b50505050905090810190601f1680156100cc57808203805160018360200361" +
                   "01000a031916815260200191505b509250505060405180910390f35b34801561" +
                   "00e657600080fd5b506040805160206004803580820135601f81018490048402" +
                   "8501840190955284845261013394369492936024939284019190819084018382" +
                   "80828437509497506101cc9650505050505050565b005b600080546040805160" +
                   "20601f6002600019610100600188161502019095169490940493840181900481" +
                   "0282018101909252828152606093909290918301828280156101c15780601f10" +
                   "610196576101008083540402835291602001916101c1565b8201919060005260" +
                   "20600020905b8154815290600101906020018083116101a457829003601f1682" +
                   "01915b505050505090505b90565b604080518181526000805460026000196101" +
                   "00600184161502019091160492820183905233927fe826f71647b8486f2bae59" +
                   "832124c70792fba044036720a54ec8dacdd5df4fcb9285918190602082019060" +
                   "60830190869080156102715780601f1061024657610100808354040283529160" +
                   "200191610271565b820191906000526020600020905b81548152906001019060" +
                   "200180831161025457829003601f168201915b50508381038252845181528451" +
                   "60209182019186019080838360005b838110156102a557818101518382015260" +
                   "200161028d565b50505050905090810190601f1680156102d257808203805160" +
                   "01836020036101000a031916815260200191505b509450505050506040518091" +
                   "0390a280516102f49060009060208401906102f8565b5050565b828054600181" +
                   "600116156101000203166002900490600052602060002090601f016020900481" +
                   "019282601f1061033957805160ff1916838001178555610366565b8280016001" +
                   "0185558215610366579182015b82811115610366578251825591602001919060" +
                   "01019061034b565b50610372929150610376565b5090565b6101c991905b8082" +
                   "1115610372576000815560010161037c5600a165627a7a723058202225a35c50" +
                   "7b31ac6df494f4be31057c7202b5084c592bdb9b29f232407abeac0029";
  
  
  // Connect to the network
  let provider = ethers.getDefaultProvider('ropsten');
  
  // Load the wallet to deploy the contract with
  let privateKey = '0x0123456789012345678901234567890123456789012345678901234567890123';
  let wallet = new ethers.Wallet(privateKey, provider);
  
  // Create an instance of a Contract Factory
  let factory = new ethers.ContractFactory(abi, bytecode, wallet);
  
  // Notice we pass in "Hello World" as the parameter to the constructor
  let contract = await factory.deploy("Hello World");
  
  // The address the Contract WILL have once mined
  // See: https://ropsten.etherscan.io/address/0x2bd9aaa2953f988153c8629926d22a6a5f69b14e
  console.log(contract.address);
  // "0x2bD9aAa2953F988153c8629926D22A6a5F69b14E"
  
  // The transaction that was sent to the network to deploy the Contract
  // See: https://ropsten.etherscan.io/tx/0x159b76843662a15bd67e482dcfbee55e8e44efad26c5a614245e12a00d4b1a51
  console.log(contract.deployTransaction.hash);
  // "0x159b76843662a15bd67e482dcfbee55e8e44efad26c5a614245e12a00d4b1a51"
  
  // The contract is NOT deployed yet; we must wait until it is mined
  await contract.deployed()
  
  // Done! The contract is deployed.
  ```



  ### 连接已经部署的合约：

  #### 连接合约(Connecting to a Contract)：

  `new ethers.Contract(addressOrName,abi,providerOrSigner)`:

  通过地址或名(addressOrName)来进行与合约的连接，交互接口通过abi设置，连接方式通过provider或Signer设置。

  接口设置：[Contract ABI](https://docs.ethers.io/ethers.js/html/api-contract.html#contract-abi)

  不同连接方式对合约操作权限与限制的影响：[Providers&Signers](https://docs.ethers.io/ethers.js/html/api-contract.html#providers-vs-signers)



  连接一个存在的合约：

  ```javascript
  const ethers = require('ethers');
  
  // The Contract interface
  let abi = [
      "event ValueChanged(address indexed author, string oldValue, string newValue)",
      "constructor(string value)",
      "function getValue() view returns (string value)",
      "function setValue(string value)"
  ];
  
  // Connect to the network
  let provider = ethers.getDefaultProvider();
  
  // The address from the above deployment example
  let contractAddress = "0x2bD9aAa2953F988153c8629926D22A6a5F69b14E";
  
  // We connect to the Contract using a Provider, so we will only
  // have read-only access to the Contract
  let contract = new ethers.Contract(contractAddress, abi, provider);
  ```

  Call一个只读的Constant-Method:

  ```javascript
  // Get the current value
  let currentValue = await contract.getValue();
  
  console.log(currentValue);
  // "Hello World"
  ```

  Call一个Non-Constant Method:

  ```javascript
  // A Signer from a private key
  let privateKey = '0x0123456789012345678901234567890123456789012345678901234567890123';
  let wallet = new ethers.Wallet(privateKey, provider);
  
  // Create a new instance of the Contract with a Signer, which allows
  // update methods
  let contractWithSigner = contract.connect(wallet);
  // ... OR ...
  // let contractWithSigner = new Contract(contractAddress, abi, wallet)
  
  // Set a new Value, which returns the transaction
  let tx = await contractWithSigner.setValue("I like turtles.");
  
  // See: https://ropsten.etherscan.io/tx/0xaf0068dcf728afa5accd02172867627da4e6f946dfb8174a7be31f01b11d5364
  console.log(tx.hash);
  // "0xaf0068dcf728afa5accd02172867627da4e6f946dfb8174a7be31f01b11d5364"
  
  // The operation is NOT complete yet; we must wait until it is mined
  await tx.wait();
  
  // Call the Contract's getValue() method again
  let newValue = await contract.getValue();
  
  console.log(currentValue);
  // "I like turtles."
  ```

  监听事件：

  ```javascript
  contract.on("ValueChanged", (author, oldValue, newValue, event) => {
      // Called when anyone changes the value
  
      console.log(author);
      // "0x14791697260E4c9A71f18484C9f997B308e59325"
  
      console.log(oldValue);
      // "Hello World"
  
      console.log(newValue);
      // "Ilike turtles."
  
      // See Event Emitter below for all properties on Event
      console.log(event.blockNumber);
      // 4115004
  });
  ```

  持续监听（Filtering an Event）：

  ```javascript
  // A filter that matches my Signer as teh author
  let filter = contract.filters.ValueChanged(wallet.address);
  
  contract.on(filter, (author, oldValue, newValue, event) => {
      // Called ONLY when your account changes the value
  });
  ```



  #### Prototype:

  `prototype.address`:返回合约部署的地址

  `prototype.deployTransaction`:如果已经通过ContractFactory对合约进行了部署，那么返回的就是部署时的transaction对象，如果没有部署返回的就是null

  `prototype.interface`:返回一个格式化的ABI对象

  #### 合约部署状态：

  `prototype.deployed()` => Promise<Contract>

  如果一个合约通过deploy部署成功，并成功进行挖矿确认。那就会返回一个包含Contract对象的Promise。

  ### 元类属性：

  > 因为Contract对象是动态的并且是在运行时加载(loaded at runtime)，许多对象中的属性是动态改变的，由Contract ABI来决定。

- Utilities(公用模块)：

## 4. solc