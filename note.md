# 要点记录

* 不能生成实例的合约，要用[Interface 类](https://github.com/JhoneLee/WTF-Ethers/tree/main/13_EncodeCalldata)去 calldata， 除此之外[Interface类还可以解码交易信息](https://github.com/JhoneLee/WTF-Ethers/blob/main/20_DecodeTx/img/20-3.png)
* 发送交易前为了防止浪费gas费，需要预执行一次交易判断是否可达，用[staticCall](https://github.com/JhoneLee/WTF-Ethers/blob/main/11_StaticCall/readme.md)
* 校验对方的合约是否符合它声称所支持的规范，用[基于ERC165的supportsInterface检测](https://github.com/JhoneLee/WTF-Ethers/blob/main/12_ERC721Check/readme.md)  ERC20因为出现的早所以不能用上面的方法检测，[推荐方式如下](https://github.com/JhoneLee/WTF-Ethers/blob/main/24_ERC20Check/readme.md) 用于搞链上分析和识别貔貅
* 只读合约用provider 读写合约用wallet
* 可以[监听交易池](https://github.com/JhoneLee/WTF-Ethers/blob/main/19_Mempool/readme.md)，查看pending的全部交易的 to gas费率 滑点等等， 但是由于交易订单巨量但免费节点的请求数量有限，这个东西知道就行
* 自己的代币想设置最小单位 以及相关的运算用 [units系列工具函数](https://github.com/JhoneLee/WTF-Ethers/blob/main/10_Units/readme.md)
* 涉及进行货币交易的合约 Transfer方法要等待交易完成在进行后续操作 await tx.wait(), 钱包的转账 sendTransaction方法也一样
* [有多笔**合约转账**可以利用 Airdrop 合约一次性完成](https://github.com/JhoneLee/WTF-Ethers/blob/main/15_MultiTransfer/MultiTransfer.js)，节约Gas费
* ETH 是以太坊网络的原生代币，直接用于支付交易费用和转账；WETH 是 ERC-20 版本的 ETH，用于与 ERC-20 代币和智能合约交互， 所以一个钱包地址的资产，需要分别查看eth余额和weth余额, **互相转换要收取gas费**
   ```js
   console.log("\n3. 读取一个地址的ETH和WETH余额")
   //读取WETH余额
   const balanceWETH = await contractWETH.balanceOf(wallets[19])
   console.log(`WETH持仓: ${ethers.formatEther(balanceWETH)}`)
   //读取ETH余额
   const balanceETH = await provider.getBalance(wallets[19])
   console.log(`ETH持仓: ${ethers.formatEther(balanceETH)}\n`)
   // eth转换weth
   const txWeth = await contractWETH.deposit({
    value: ethers.parseEther('0.001'),
  });
   // weth 转换eth
   const txEth = await contractWETH.withdraw(ethers.parseEther('0.001'));
   ```
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
* 用ethers.js [生成自己的助记词和钱包](https://github.com/JhoneLee/WTF-Ethers/tree/main/14_HDwallet)，并且可以把钱包搞成加密json，解密json后得到钱包的信息
* [没有多个钱包、weth批量转账到一个账户方法，只能通过遍历](https://github.com/JhoneLee/WTF-Ethers/blob/main/16_MultiCollect/readme.md)，对每个钱包和WETH合约执行sendTransaction\ transfer , 还要预留gas费，不能一下全转完
* [Merkle Tree 默克尔树算法生成用户白名单](https://github.com/JhoneLee/WTF-Ethers/blob/main/17_MerkleTree/readme.md)主要用来校验用户的地址时候有权限铸造NFT， 同时还有[数字签名白名单法](https://github.com/JhoneLee/WTF-Ethers/blob/main/18_Signature/readme.md)也可以提供用户白名单鉴权
* hardhat 简化了合约的部署和编译，如果没有hardhat, 我们需要取到abi后用一下方法生成合约
  ```js
   // 3. 创建合约工厂
   // NFT的abi
   const abiNFT = [
       "constructor(string memory name, string memory symbol, bytes32 merkleroot)",
       "function name() view returns (string)",
       "function symbol() view returns (string)",
       "function mint(address account, uint256 tokenId, bytes32[] calldata proof) external",
       "function ownerOf(uint256) view returns (address)",
       "function balanceOf(address) view returns (uint256)",
   ];
   // 合约字节码，在remix中，你可以在两个地方找到Bytecode
   // i. 部署面板的Bytecode按钮
   // ii. 文件面板artifact文件夹下与合约同名的json文件中
   // 里面"object"字段对应的数据就是Bytecode，挺长的，608060起始
   // "object": "608060405260646000553480156100...
   const bytecodeNFT = contractJson.default.object;
   const factoryNFT = new ethers.ContractFactory(abiNFT, bytecodeNFT, wallet);
   const contractNFT = await factoryNFT.deploy("WTF Merkle Tree", "WTF", root)
  ```
* 钱包地址[可以搞成靓号](https://github.com/JhoneLee/WTF-Ethers/blob/main/21_VanityAddress/readme.md) 但是靓号也没多靓，有点鸡肋这个东西
* 智能合约在以太坊中是公开透明的，不要写私密数据到智能合约中，任何人都可以[解析智能合约数据](https://github.com/JhoneLee/WTF-Ethers/blob/main/22_ReadAnyData/readme.md)
* [三明治攻击简单例子](https://github.com/JhoneLee/WTF-Ethers/blob/main/23_Frontrun/readme.md) 不劳而获 人类之耻
