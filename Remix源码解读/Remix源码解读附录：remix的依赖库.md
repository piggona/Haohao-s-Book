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

- Wallet and Signers(钱包及签名)：

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

- Providers(供应商/节点)：

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

- Contracts(合约)：

- Utilities(公用模块)：

## 4. solc