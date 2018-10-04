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

     - vm.runTx(opts,cb)

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

     - vm.runCode(opts,cb)

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
    - `prototype.mnemonic`:

  - Signers(签名)

- Providers(供应商/节点)：

- Contracts(合约)：

- Utilities(公用模块)：

## 4. solc