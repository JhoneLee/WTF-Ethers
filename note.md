# 要点记录

* 不能生成实例的合约，要用[Interface 类](https://github.com/JhoneLee/WTF-Ethers/tree/main/13_EncodeCalldata)去 calldata
* 发送交易前为了防止浪费gas费，需要预执行一次交易判断是否可达，用[staticCall](https://github.com/JhoneLee/WTF-Ethers/blob/main/11_StaticCall/readme.md)
* 校验对方的合约是否符合它声称所支持的规范，用[基于ERC160的supportsInterface检测](https://github.com/JhoneLee/WTF-Ethers/blob/main/12_ERC721Check/readme.md)
* 只读合约用provider 读写合约用wallet
* 自己的代币想设置最小单位 以及相关的运算用 [units系列工具函数](https://github.com/JhoneLee/WTF-Ethers/blob/main/10_Units/readme.md)
* 进行货币交易的Transfer 要等待交易完成在进行后续操作 await tx.wait()
* 部署合约需要自己写js脚本，合约verify的时候要注意构造函数有参数的要把参数带上
   ```js
   // 假如我的构造函数里面有数值
   const { ethers } = require('hardhat');
   module.exports = [ethers.parseUnits('1000000', 18)];
   ```
   ```sh
   hardhat verify --network sepolia 0x72140Ec4BBEFE2A546A1eEcC7A30Efca4B2D967b  --constructor-args tokenArgs.js
   ```
* 通过[订阅合约的事件](https://github.com/JhoneLee/WTF-Ethers/blob/main/07_Event/readme.md)，来更新自己的界面反馈最新的用户数据
* 事件数据太多，还可以[过滤自己想要看的事件日志](https://github.com/JhoneLee/WTF-Ethers/blob/main/09_EventFilter/readme.md)
