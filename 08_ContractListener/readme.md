---
title: 8. 监听合约事件
tags:
  - ethers
  - javascript
  - provider
  - event
  - listener
  - frontend
  - web
---

# Ethers极简入门: 8. 监听合约事件

我最近在重新学`ethers.js`，巩固一下细节，也写一个`WTF Ethers极简入门`，供小白们使用。

**推特**：[@0xAA_Science](https://twitter.com/0xAA_Science)

**WTF Academy社群：** [官网 wtf.academy](https://wtf.academy) | [WTF Solidity教程](https://github.com/AmazingAng/WTF-Solidity) | [discord](https://discord.gg/5akcruXrsk) | [微信群申请](https://docs.google.com/forms/d/e/1FAIpQLSe4KGT8Sh6sJ7hedQRuIYirOoZK_85miz3dw7vA1-YjodgJ-A/viewform?usp=sf_link)

所有代码和教程开源在github: [github.com/WTFAcademy/WTFEthers](https://github.com/WTFAcademy/WTF-Ethers)

-----

提示：本教程基于ethers.js 6.3.0 ，如果你使用的是v5，可以参考[ethers.js v5文档](https://docs.ethers.io/v5/)。

这一讲，我们将介绍如何监听合约，并实现监听USDT合约的`Transfer`事件。

具体可参考[ethers.js文档](https://docs.ethers.org/v6/api/contract/#ContractEvent)。

## 监听合约事件

### `contract.on`
在`ethersjs`中，合约对象有一个`contract.on`的监听方法，让我们持续监听合约的事件：

```js
contract.on("eventName", function)
```
`contract.on`有两个参数，一个是要监听的事件名称`"eventName"`，需要包含在合约`abi`中；另一个是我们在事件发生时调用的函数。

### contract.once

合约对象有一个`contract.once`的监听方法，让我们只监听一次合约释放事件，它的参数与`contract.on`一样：

```js
contract.once("eventName", function)
```

## 监听`USDT`合约

1. 声明`provider`：Alchemy是一个免费的ETH节点提供商。需要先申请一个，后续会用到。你可以参考这篇攻略来申请Alchemy API[Solidity极简入门-工具篇4：Alchemy](https://github.com/AmazingAng/WTFSolidity/blob/main/Topics/Tools/TOOL04_Alchemy/readme.md )

  ```js
  import { ethers } from "ethers";
  // 准备 alchemy API  
  // 可以参考https://github.com/AmazingAng/WTFSolidity/blob/main/Topics/Tools/TOOL04_Alchemy/readme.md 
  const ALCHEMY_MAINNET_URL = 'https://eth-mainnet.g.alchemy.com/v2/oKmOQKbneVkxgHZfibs-iFhIlIAl6HDN';
  // 连接主网 provider
  const provider = new ethers.JsonRpcProvider(ALCHEMY_MAINNET_URL);
  ```

2. 声明合约变量：我们只关心`USDT`合约的`Transfer`事件，把它填入到`abi`中就可以。如果你关心其他函数和事件的话，可以在[etherscan](https://etherscan.io/address/0xdac17f958d2ee523a2206206994597c13d831ec7#code)上找到。

  ```js
  // USDT的合约地址
  const contractAddress = '0xdac17f958d2ee523a2206206994597c13d831ec7'
  // 构建USDT的Transfer的ABI
  const abi = [
    "event Transfer(address indexed from, address indexed to, uint value)"
  ];
  // 生成USDT合约对象
  const contractUSDT = new ethers.Contract(contractAddress, abi, provider);
  ```

3. 利用`contract.once()`函数，监听一次`Transfer`事件，并打印结果。

  ```js
    // 只监听一次
    console.log("\n1. 利用contract.once()，监听一次Transfer事件");
    contractUSDT.once('Transfer', (from, to, value)=>{
      // 打印结果
      console.log(
        `${from} -> ${to} ${ethers.formatUnits(ethers.getBigInt(value),6)}`
      )
    })
  ```
  ![只监听一次](img/8-1.png)

4. 利用`contract.on()`函数，持续监听`Transfer`事件，并打印结果。
  ```js
    // 持续监听USDT合约
    console.log("\n2. 利用contract.on()，持续监听Transfer事件");
    contractUSDT.on('Transfer', (from, to, value)=>{
      console.log(
       // 打印结果
       `${from} -> ${to} ${ethers.formatUnits(ethers.getBigInt(value),6)}`
      )
    })
  ```
  ![持续监听](img/8-2.png)

## 总结
这一讲，我们介绍了`ethers`中最简单的链上监听功能，`contract.on()`和`contract.once()`。通过上述方法，可以你可以监听指定合约的指定事件。

在 DApp 的开发中，监听智能合约的事件是非常重要的，主要用于跟踪合约状态的变化和及时响应用户操作。以下是实际生产环境中监听合约事件的常见用途：

### 1. **更新前端 UI 和用户通知**
   - **实时反馈**：监听合约事件可以帮助 DApp 实时更新用户界面。例如，当用户成功执行转账或铸造代币的操作时，监听到相应的事件后，可以立即在界面上显示交易状态或更新用户的余额。
   - **通知和提醒**：事件触发时可以向用户发送通知，提醒操作成功、交易失败或其他重要状态。例如，在 NFT 市场中可以提醒用户他们的 NFT 被购买或拍卖成功。

### 2. **状态同步**
   - **前端与区块链数据同步**：在 DApp 中，前端页面的数据需要与区块链保持一致。通过监听事件，可以确保在合约状态发生变化时前端立即同步更新，避免显示过时的数据。
   - **高效查询**：事件数据可以简化前端查询。例如，可以通过监听合约的交易事件获取所有转账记录，而无需频繁地调用链上存储的历史数据。

### 3. **记录用户行为和生成历史记录**
   - **交易历史记录**：DApp 可以监听用户的交易事件（如转账、买卖）来生成交易历史记录，展示在用户界面上。这样用户就可以随时查看自己在 DApp 中的操作历史。
   - **活动日志**：对于治理型合约，可以记录用户的投票、提案等操作历史，从而帮助用户更好地了解参与治理的情况。

### 4. **后台任务自动执行**
   - **自动化流程**：在 DeFi 协议中，某些操作可能需要在用户执行交易后自动触发后台任务。例如，当用户质押代币、借贷或进行流动性挖矿时，监听相关事件可以自动执行必要的后台清算或奖励发放操作。
   - **调度任务**：对于需要自动调度的任务，比如在游戏中自动发放奖励、解锁成就等，也可以通过监听事件来触发这些操作。

### 5. **数据分析和监控**
   - **统计数据**：监听事件可以用于统计 DApp 的使用情况，例如计算交易量、代币转账频率、活跃用户数等，帮助开发者更好地分析用户行为。
   - **安全监控**：通过监听关键事件，DApp 可以检测到异常操作并发送警报，比如检测到代币大额转账、频繁操作等异常行为，从而及时采取措施。

### 6. **智能合约间的协作**
   - **跨合约通信**：在复杂的 DApp 系统中，不同的智能合约之间可能会互相调用。通过监听事件，某个合约可以在另一个合约完成某些操作后自动触发自身的逻辑。
   - **跨链桥应用**：在跨链桥应用中，通常需要监听链上资产的锁定或解锁事件，从而在另一条链上进行相应的资产映射或释放操作。

### 7. **优化用户体验**
   - **优化等待过程**：在用户发起交易后，DApp 可以监听交易确认事件，并在交易被确认时通知用户，避免用户长时间等待或不确定交易状态。
   - **减少链上调用**：通过事件，DApp 可以减少频繁的链上查询，降低 Gas 费用和用户的等待时间，提升整体使用体验。


### 小结

通过监听智能合约事件，DApp 可以实现实时性、交互性和自动化，大大提升了用户体验和系统的可靠性。在生产环境中，事件监听可以用于数据同步、用户通知、流程自动化、安全监控等多个场景，是开发高质量 DApp 的重要手段。
