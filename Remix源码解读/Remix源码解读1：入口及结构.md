# Remix源码解读1：入口及结构

>  Remix作为ETH官方的智能合约调试及开发环境，是了解ethereum开发全流程最好的方法。

Remix主要的包管理工具为lerna,持续集成服务(continuous integration,CI)工具为travis。

从lerna.json中：

```json
{
  "lerna": "2.10.2",
  "packages": [
    "remix-debug",
    "remix-lib",
    "remix-solidity",
    "remix-analyzer",
    "remix-tests",
    "remix-simulator"
  ],
  "version": "independent"
}
```

可以看出remix的主要的模块及文件结构。

.travis.yml的内容：

```javascript
language: node_js
node_js:
  - stable
//环境变量
env:
  - TEST_DIR=remix-lib
  - TEST_DIR=remix-solidity
  - TEST_DIR=remix-debug
  - TEST_DIR=remix-tests
  - TEST_DIR=remix-simulator
script: 
  - cd $TEST_DIR && npm install && npm test
deploy:
  provider: script
  script: remix-debugger/ci/deploy_from_travis.sh
  skip_cleanup: true
  on:
        branch: master
        condition: $TEST_DIR = remix-debugger
cache: false
```

## 1. Remix的重要模块

包含:

- remix-lib : 几乎所有模块都会require remix-lib中的公共方法
- remix-solidity : 对solidity的解析以及函数的操作

