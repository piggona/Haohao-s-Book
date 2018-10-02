# Remix源码解读2：remix-lib的解析

## 1. 对package.json文件的解读

- 依赖（dependencies):
  - async(2.1.2)：为项目提供异步功能
  - ethereumjs-block(1.6.0)：
  - ethereumjs-tx(1.3.3)：用于创建、操作和签名ethereum事务的简单模块
  - ethereumjs-util(5.1.2)：以太坊实用工具
  - ethereumjs-vm(2.3.3)：在Javascript环境中的以太坊虚拟机
  - ethers(3.0.15)：完整的以太坊链交互库
  - solc(0.4.13)：对solidity进行编译（输出为可以运行在以太坊虚拟机的代码）及优化
  - web3(0.20.6)：
- 入口文件：

## 2. 模块解读：

- 以太坊合约工作流程：

  [solidity代码]->solc编译->[EVM CODE]->ethereumjs-vm运行代码

- 