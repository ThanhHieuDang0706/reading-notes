
- [Bitcoin-区块链技术的起源](#bitcoin-区块链技术的起源)
  - [非对称加密](#非对称加密)
  - [时间戳服务器](#时间戳服务器)
  - [Merkle Tree](#merkle-tree)
  - [工作量证明(POW)](#工作量证明pow)
    - [拜占庭将军问题](#拜占庭将军问题)
    - [工作量证明谜题](#工作量证明谜题)
    - [矿机](#矿机)
    - [权益证明(POS)](#权益证明pos)
- [Ethereum - 迈入区块链2.0](#ethereum---迈入区块链20)
  - [Gas费用](#gas费用)
  - [ERC标准](#erc标准)
    - [ERC-20 同质化代币标准](#erc-20-同质化代币标准)
    - [ERC-721 非同质化代币标准](#erc-721-非同质化代币标准)
    - [ERC-1155](#erc-1155)
    - [ERC-3525 半同质化代币标准](#erc-3525-半同质化代币标准)
  - [Etherscan](#etherscan)
  - [区块链扩容方案](#区块链扩容方案)
    - [L0、L1、L2](#l0l1l2)
    - [Ethereum2.0](#ethereum20)
    - [信标链（ Beacon Chain )](#信标链-beacon-chain-)
    - [L1扩容方案](#l1扩容方案)
      - [Sharding1.0设计思路](#sharding10设计思路)
      - [Danksharding设计思路](#danksharding设计思路)
    - [L2扩容方案](#l2扩容方案)
      - [状态通道 (State Channels) / 支付通道 (Payment Channels)](#状态通道-state-channels--支付通道-payment-channels)
- [Web3.0应用架构](#web30应用架构)


# Bitcoin-区块链技术的起源
2008 年 10 月 31 日，中本聪（Satoshi Nakamoto）发布了[比特币白皮书][1]。

## 非对称加密
使用非对称加密的公私钥体系设计账户，公钥就是账户地址，持有私钥的人可以通过签名交易来支配账户里的资产。签名后的交易被广播到网络中，等待矿工打包。（保证了签名不可伪造也不可抵赖）

<img src="bc01.svg" width="600" />

## 时间戳服务器
每次打包一些交易（最多 1MB 大小）会形成一个`区块(block)`，根据区块头的内容加上`时间戳(timestamp)`可以算出这个区块的哈希值，同时区块头里有一个指向前一区块的哈希值，从而这些不断增长的区块可以串接起来形成单向有序链表（区块链）。所有历史交易记录都保存在区块链中。

全网所有节点地位相同，都持有这个账本的全量数据，账本内容完全公开可追溯。这样就形成了一个全局账本，确保同一枚比特币不会被`双重支付`。

<img src="bc02.svg" width="600" />

## Merkle Tree
区块体里存放了这次打包的 N 笔交易，所有交易按规则生成一个Merkle Tree，每个区块头里有个 merkle root 字段记录默克尔树的根，来保证区块体交易不可篡改。

<img src="bc03.jpeg" width="600" />

Merkle Tree的特点：任何一个叶子结点被篡改，会被立刻发现

<img src="bc04.jpg" width="600" />

如果一枚硬币最近发生的交易发生在足够多的区块之前，那么，这笔交易之前该硬币的花销交易记录可以被丢弃 —— 目的是为了节省磁盘空间。

<img src="bc05.svg" width="600" />

## 工作量证明(POW)
有些节点是矿工，负责把网络中的交易打包到账本中，并赚取打包的奖励和交易手续费。矿工打包出的区块除了满足数据的合法性要求（交易内容都是可以通过验证的），还需要给出一个`工作量证明`。

### 拜占庭将军问题
`拜占庭将军问题`是Leslie Lamport（2013年的图灵讲得住）用来为描述分布式系统一致性问题（Distributed Consensus）在[论文][2]中抽象出来一个的例子。

>拜占庭帝国想要进攻一个强大的敌人，为此派出了10支军队去包围这个敌人。这个敌人虽不比拜占庭帝国，但也足以抵御5支常规拜占庭军队的同时袭击。这10支军队在分开的包围状态下同时攻击。他们任一支军队单独进攻都毫无胜算，除非有至少6支军队（一半以上）同时袭击才能攻下敌国。他们分散在敌国的四周，依靠通信兵骑马相互通信来协商进攻意向及进攻时间。困扰这些将军的问题是，他们不确定他们中是否有叛徒，叛徒可能擅自变更进攻意向或者进攻时间。在这种状态下，拜占庭将军们才能保证有多于6支军队在同一时间一起发起进攻，从而赢取战斗？

说明：
1. 拜占庭将军问题中并不去考虑通信兵是否会被截获或无法传达信息等问题，即消息传递的信道绝无问题。Lamport已经证明了在消息可能丢失的不可靠信道上试图通过消息传递的方式达到一致性是不可能的。（计算机网络通讯中TCP协议保障）
2. Lamport提出的Paxos算法或其衍生算法(ZooKeeper/Chubby等系统支持)仅适用于中心化的分布式系统，这样的系统的没有不诚实的节点（不会发送虚假错误消息，但允许出现网络不通或宕机出现的消息延迟）。

中本聪通过引入`工作量证明(POW:Proof of Work)`来解决不诚实节点的问题，使得在P2P网络中可以保证一致性。

### 工作量证明谜题
>找到一个`nonce值`，使得新区块头的哈希值小于某个指定的值，即区块头结构中的“难度目标”。在节点成功找到满足的Hash值 之后，会马上对全网进行广播打包区块，网络的节点收到广播打包区块，会立刻对其进行验证。

如果验证通过，则表明已经有节点成功解迷，自己就不再竞争当前区块打包，而是选择接受这个区块，记录到自己的账本中，然后进行下一个区块的竞争猜谜。

计算过程类似：

```py
for nonce in range(0, 2**32):
    block_header = version + previous_block_hash + merkle_root + timestamp + target_bits + nonce
    if HASH(block_header) < target_bits: 
        break
    else: 
        continue
```

说明：

target_bits，用来控制挖矿难度（比特币期望出块的平均时间是10分钟，每14天整个比特币系统会根据之前出块速度调整全网难度来靠近 10 分钟。）

```
目标值 = 最大目标值 / 难度值
最大目标值=0x00000000FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF

新难度值 = 旧难度值 * ( 过去2016个区块花费时长 / 20160 分钟 )
```

实际挖矿的基本步骤：

1. 生成Coinbase交易，并与其他所有准备打包进区块的交易组成交易列表，并生成默克尔哈希;
2. 把默克尔哈希及其他相关字段组装成区块头，将区块头(Block Header)作为工作量证明的输入，区块头中包含了前一区块的哈希，区块头一共80字节数据;
3. 不停地变更区块头中的随机数即nonce的数值，也就是暴力搜索，将结果值与当前网络的目标值做对比，如果小于目标值，则解题成功，工作量证明完成。

在指定时间内，给定一个难度，找到答案的概率唯一地由所有参与者能够迭代哈希的速度决定。掌握51%的算力对系统进行攻击所付出的代价远远大于作为一个系统的维护者和诚实参与者所得到的。

### 矿机
PoW挖矿算法大致分为两个大类，第一类叫计算困难，第二类内存困难。计算机的组成分为计算单元和存储单元，一个计算机的瓶颈往往是IO，如果要制造大量的IO操作，可以通过写程序撑大内存，制造大量的数据处理过程，使工作量证明从计算单元转变为 存储单元。

- 计算困难型：Scrypt、X11、SHA-3
- 内存困难型：ETHASH（Dagger-Hashimoto的修改版本,以太坊的PoW挖矿算法）

矿机演进：CPU-GPU-FPGA-ASIC专业芯片(高性能计算)+矿池（分布式计算）

### 权益证明(POS)
PoS全称是Proof of Stake，⽬的就是为了解决使⽤PoW挖矿出现⼤量资源浪费的问题。

在PoS系统中，这个公式变更为： Hash(block_header) < target_bits * CoinAge。

说明：
- CoinAge(币龄)，字⾯意思就是币数量乘以持有天数。如果你的币龄越⼤，也就意味着你的获得答案越容易。

<img src="bc13.png" width="600" />

问题
1. 冷启动问题。
2. 囤积问题，这会造成币流通的不充分（通货紧缩）。
3. Nothing at Stake，翻译过来叫做⽆成本利益问题。⼤体的意思在PoS系统中做任何事⼏乎没有成本，⽐如在PoS系统上挖矿⼏乎没有成本，这也就意味着分叉⾮常⽅便。（反正也没什么成本，反正分叉链和主链都可以同时挖，说不定什么时候就值钱了）

# Ethereum - 迈入区块链2.0
比特币的脚本能力非常有限，大家基本就只用来转账，以太坊创始人Vitalik Buterin（中文用户常称之V神，1994出生）认为可以在区块链上构建出一种叫`智能合约（smart contract）`的东西，提供广泛的可编程能力来满足现实世界里各种各样的需求而不仅仅是转账。

以太坊相对于比特币，做了几个关键的改进：

1. 引入账户系统和`世界状态`。以太坊也有自己的原生代币 ETH，每个地址持有的 ETH 余额直接明确记录下来，转账就是分别增减两个账户的余额。
2. 引入`智能合约`。智能合约在被创建时携带一份代码，以太坊的节点客户端提供了一个叫 `EVM（Ethereum Virtual Machine）`的虚拟机环境，能够执行这个代码，智能合约的代码创建后不可修改(`code is law`)，在链上也有一个公开地址。
3. 账户发起的一笔交易可以是普通转账，也可以是调用某个智能合约的某个函数，这样就有动作可以驱动智能合约代码的执行和状态变更。

以太坊通过引入图灵完备的智能合约编程+智能合约可以持久化自己内部的状态（可以理解为它的全局变量）来向上提供丰富的可操纵性，号称世界计算机。账户的基本情况、智能合约的持久化数据共同组成整条以太坊链的世界状态，用户通过发起交易推动世界状态一次次改变。

<img src="bc06.png" width="600" />

说明：
1. 还有其他的一些知名公链比如 Solana, Avalanche, BSC, NEAR。
2. 智能合约不能获取到链外的数据，比如我们写一个合约来赌球，球赛的结果以太坊肯定是不知道的。把这种链外的数据写到链内的工具叫`预言机（oracle）`。预言机是区块链与现实世界进行数据交互的接口，已经有去中心化运行的预言机产品，保障上链信息的可信度。

## Gas费用
在技术上Gas的价格表示为 wei，是 ETH 最小的增量单位。

* 1 wei 等于0.000000000000000001 ETH（$10^{-18}$)
* 1 gwei 等于1,000,000,000 wei

用户已经习惯了以 gwei 为单位来表示 gas 价格，可以使用[Gas.Watch][3]留意实时的gas价格。Gas 会随着打包进区块链的交易需求上下波动。 

在London Upgrade之前（August 2021）

    Total Fee = Gas units (limit) * Gas price per unit

假设 Alice 需要付给 Bob 1 ETH。在这次交易中：
- gas limit = 21,000 units
- gas price = 200 gwei 
- 全部费用 = 21,000 * 200 = 4,200,000 gwei = 0.0042 ETH

账户变化：
- Alice账户：-1.0042 ETH
- Bob账户：+1.0000 ETH
- Miner账户：+0.0042 ETH

在London Upgrade之后

    Total Fee = Gas units(limit) * (Base fee + Tip)

其中：
- `Base fee`:every block has a base fee, the minimum price per unit of gas for inclusion in this block, calculated by the network based on demand for block space.
- `Tip` (priority fee):compensates miners for executing and propagating user transactions in blocks
- `max fee`(maxFeePerGas):refund = max fee - (Base fee + Tip)，refund>=0的前提下才能执行。

假设 Alice 需要付给 Bob 1 ETH。在这次交易中：
- gas limit = 21,000 units
- Base fee = 100 gwei
- Tip = 10 gwei
- 全部费用 = 21,000 * (100+10) = 2,310,000 gwei = 0.00231 ETH

账户变化：
- Alice账户：-1.00231 ETH
- Bob账户：+1.0000 ETH
- Miner账户：+0.00021 ETH
- Base fee of 0.0021 ETH is burned

说明：
- Block size
  - Before the London Upgrade, Ethereum had fixed-sized blocks. In times of high network demand, these blocks operated at total capacity. As a result, users often had to wait for high demand to reduce to get included in a block, which led to a poor user experience.
  - The London Upgrade introduced variable-size blocks to Ethereum. Each block has a target size of 15 million gas, but the size of blocks will increase or decrease in accordance with network demand, up until the block limit of 30 million gas (2x the target block size).
- Base fee
  - The base fee is calculated independently of the current block and is instead determined by the blocks before it - making transaction fees more predictable for users. When the block is mined this base fee is "burned", removing it from circulation.
  - The base fee is calculated by a formula that compares the size of the previous block (the amount of gas used for all the transactions) with the target size. The base fee will increase by a maximum of 12.5% per block if the target block size is exceeded. 

## ERC标准
[Ethereum Improvement Proposals][4] (`EIPs`) describe standards for the Ethereum platform.EIPs are separated into a number of types, and each has its own list of EIPs.

- `Standard Track (497)`:Describes any change that affects most or all Ethereum implementations, such as a change to the network protocol, a change in block or transaction validity rules, proposed application standards/conventions, or any change or addition that affects the interoperability of applications using Ethereum. 
- `Core (189)`:Improvements requiring a consensus fork
- `Interface (42)`:Includes improvements around client API/RPC specifications and standards, and also certain language-level standards like method names (EIP-6) and contract ABIs.
- `ERC (253)`:Application-level standards and conventions, including contract standards such as token standards (ERC-20), name registries (ERC-137), URI schemes (ERC-681), library/package formats (EIP190), and wallet formats (EIP-85).

### ERC-20 同质化代币标准
[ERC-20][5]标准接口主要是在智能合约中实现了Token(代币)的标准api，也就是通过函数来为代币提供基本的功能：例如Token转账、授权等等。

必要函数:

```java
// IERC20.sol
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.0;

interface IERC20 {

    // 返回token总供应量
    function totalSupply() external view returns (uint256);

    // 返回账户的余额
    function balanceOf(address account) external view returns (uint256);

    // 向recipient地址转移value数量的token，函数必须触发事件Transfer
    function transfer(address recipient, uint256 amount) external returns (bool);

    // 查询owner授权给spender的额度
    function allowance(address owner, address spender) external view returns (uint256);

    // 授权spender可以从我们在账户最多转移token的数量value，可以多次转移，但总量不超过value
    function approve(address spender, uint256 amount) external returns (bool);

    // 可以允许第三方代表我们转移代币。 如果 _from 账号没有授权调用帐户转移代币，则该函数需要抛出异常。
    function transferFrom(address sender, address recipient, uint256 amount) external returns (bool);

    // 当有token转移时，触发Transfer事件
    event Transfer(address indexed from, address indexed to, uint256 value);

    // approve函数成功执行时，触发Approval事件
    event Approval(address indexed owner, address indexed spender, uint256 value);
}
```

可选函数：ERC-20标准接口中有三个可选函数，主要是返回代币的基本信息，所以也可以称为元数据。

```java
// IERC20Metadata.sol
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.0;

import "./IERC20.sol";

interface IERC20Metadata is IERC20 {

    // 返回的是代币的名称，例如Binance Coin
    function name() external view returns (string);

    // 返回的是代币的代号，例如BNB
    function symbol() external view returns (string);

    // 返回的是代币的使用的小数位数，例如8（意味着将代币总量除以100000000来获取表现的形式）
    function decimals() external view returns (uint8);
}
```

### ERC-721 非同质化代币标准
非同质化（Non-Fungible Token，以下简称 NFT 或 NFTs）Token(代币)标准
* 智能合约中实现 NFT 的标准API。 标准提供了跟踪和转移NFTs的基本功能。
* NFT可以代表对数字或物理资产的所有权。非同质代表独一无二，NFT是可区分的
* ERC20的Token是可置换的，且可细分为N份（1 = 10 * 0.1）, 而ERC721的Token最小的单位为1，无法再分割。

每个符合ERC-721的合同都必须实现 ERC721 和 ERC165 接口（ERC-165只是一个标准，要求使用一种标准的方法去发布或者检测（supportsInterface）一个智能合约所实现的接口。）。

```java
interface IERC721 is IERC165 {

    // 变更NFT所有权、NFT的创建（from == 0）和销毁时（to == 0）触发。合约创建时除外
    event Transfer(address indexed from, address indexed to, uint256 indexed tokenId);

    // 在NFT的授权地址approved address变更或者重新确认时被触发
    // 发起transfer时，approved address会被重置为none
    event Approval(address indexed owner, address indexed approved, uint256 indexed tokenId);

    // operator被授权或撤权时触发。operator可以管理owner的所有NFT
    event ApprovalForAll(address indexed owner, address indexed operator, bool approved);

    // 返回由owner持有的NFTs的数量。
    function balanceOf(address owner) external view returns (uint256 balance);

    // 返回tokenId代币持有者的地址
    function ownerOf(uint256 tokenId) external view returns (address owner);

    // 将NFT的所有权从一个地址转移到另一个地址
    // 1. 调用者msg.sender应该是当前tokenId的所有者或被授权的地址
    // 2. from不是tokenId的所有者 、to是零地址、tokenId不是有效id均抛出异常。
    // 3. 当转移完成时，函数检查 to是否是合约，如果是，调用onERC721Receiver方法，并检查其返回值是否是0x150b7a02，（即bytes4(keccak256("onERC721Received(address,address,uint256,bytes)")),如果不是则抛出异常
    function safeTransferFrom(
        address from,
        address to,
        uint256 tokenId
    ) external;

    // 授予地址to具有tokenId的控制权，方法成功后需触发Approval 事件。
    function approve(address to, uint256 tokenId) external;

    // 获取单个NFT的授权地址
    function getApproved(uint256 tokenId) external view returns (address operator);

    // 启用或禁用第三方（操作员）管理msg.sender所有资产
    // 触发 ApprovalForAll 事件，合约必须允许每个所有者可以有多个操作员
    // approved True 表示授权, false 表示撤销
    function setApprovalForAll(address operator, bool _approved) external;

    // 查询一个地址是否是另一个地址的授权操作员
    function isApprovedForAll(address owner, address operator) external view returns (bool);

    // 将NFT的所有权从一个地址转移到另一个地址，功能同上，附带data参数，传递给接收者
    function safeTransferFrom(
        address from,
        address to,
        uint256 tokenId,
        bytes calldata data
    ) external;
}
```

### ERC-1155
ERC-1155 是 ERC-20 和 ERC-721 的升级规范，它允许在一个交易中发送多种不同的代币。

标准 | ERC-20 | ERC-721 | ERC-1155
---|--------|---------|---------
代币类型 | 同质化代币 | 非同质化代币 | 同质化代币、非同质化代币、介于同质化和非同质化代币之间可以互相切换的代币
特点 | 代币属性相同、可无损互换、可拆分 | 代币属性互不相同、不可互换、不可拆分 | 前两者的特点都有，且在一定程度上可以在两者中切换
生成处理 | 一次性只能生成一种 ERC-20 代币，一次性只能进行单笔单对象交易，并且交易处理需要多次批准 | 一次性只能生成一种 ERC-721 代币，一次性只能进行单笔单对象交易，并且交易处理需要多次批准 | 一次性可以生成多种 ERC-1155 代币资产类别，一次性可以进行多笔多对象交易，交易处理只需要一次批准

说明：

1. ERC1155相比于ERC721简而言之最大的区别就是它可以一个合约承载多类FT与NFT，可以将其理解为是ERC20和ERC721的融合加强版，想发行同质化和非同质化的代币1155全部搞定，而不用用多个合约承载再进行交互。
2. ERC721是一个合约承载1类NFT，1类NFT承载多个NFT，如无聊猿，它的合约有且仅能发行无聊猿这一套NFT，每个具体的NFT编号均不相同为递增，但是ERC1155一个合约可以发行多类NFT，它最常用的场景在游戏，比如一个游戏中，可能会有很多类装备如“武器”、“坐骑”、“药品”等，这些装备有的是非同质化的，比如屠龙宝刀只有1个，有的是同质化的比如药品都是一样的喝一瓶补10滴血，而传统的721只能发行一类实体，但是1155却可以发行多类。

阿迪达斯NFTopensea网址如下：https://opensea.io/collection/adidasoriginals。

<img src="bc07.png" width="600" />

<img src="bc08.png" width="200" />

阿迪达斯NFT共分为四个阶段，第1、2、3阶段都涉及到销毁兑换操作，第四个阶段会获得一个ERC721 NFT。

<img src="bc09.png" width="600" />

### ERC-3525 半同质化代币标准
SFT 适合于表达内含数量特征、有时需要进行合并或拆分操作的数字物品。典型的例子是金融票据、高级金融合约、土地，以及一切具有内在数量的标准化商品。例如，两张条件完全相同、面值各 500 元的债券，等同于一张相同条件、面值 1000 元的债券。再例如，两块虚拟土地，在一定条件下，可以合并视为一块。在实体经济中，两块同型号、有效面积各 20 平米的太阳能板，在管理核算的时候，可以视为一个 40 平米的太阳能板，两车皮的同型号煤炭，可以按吨位加合统计为同一批次。

## Etherscan
在区块链世界中, 有一个 "快递查询工具" 的应用, 它就是 Etherscan, 网址 https://etherscan.io/

中文网址是 https://etherscan.io/language.aspx

## 区块链扩容方案
### L0、L1、L2
区块链逻辑架构：

<img src="bc10.png" width="400" />

- 第 0 层又称数据传输层，主要涉及区块链和传统网络之间的结合问题。有这样几个作用：
  - 允许区块链相互交互。一个很好的例子是 [Cosmos][6]，它创建了一个可互操作的区块链生态系统，这要归功于其“ Tendermint IBC ”（区块链间通信协议）。
  - 更快、更便宜的交易。使用 IBC，PoS 共识可以实现跨多个链进行交易，导致最终确定时间几乎在瞬间发生（最终确定 = 当一个块被批准时，不能回滚，并且被认为是不可逆的）。这使得跨链交易所的交易更快、更便宜。
- 第 1 层是在自己的区块链上处理和完成交易的区块链（例如比特币和以太坊）。这是诸如共识（PoW、PoS）之类的事情以及诸如区块时间和争议解决之类的所有技术细节运作的地方。
  - Layer 1扩容方案又称链上扩容，指在区块链基层协议上实现的扩容解决方案。
- 第 2 层是与第 1 层结合使用的第三方集成，Layer2诞生的主要目的便是为了提高可扩展性和每秒交易数（系统吞吐量）。
  - Layer 2扩容方案又称链下扩容，指不改变区块链底层协议和基础规则，通过状态通道、侧链等方案提高交易处理速度。

各层著名的应用：

<img src="bc11.png" width="400" />  

### Ethereum2.0
以太坊 2.0 意在解决以太坊的共识问题和扩展性问题，基于共识问题提出了一个新概念就是`信标链（ Beacon Chain )`，而基于扩展性问题提出的是`分片链（Shard Chains)`。

2022年9月15日14:42:42。以太坊在区块高度15537393触发合并机制，并产出首个PoS区块——高度为15537394。

<img src="bc14.jpg" width="400" />

Hsiao-Wei Wang针对以太坊2.0系统的系统架构图

<img src="bc15.jpg" width="600" />

### 信标链（ Beacon Chain )
信标链是一种全新的权益证明（PoS）区块链，它有如下几个作用：

**1.管理验证者**

信标链的一个主要工作就是管理`验证者集合（validator set）`，即那些在合约中押注了32个以太坊的节点，这些节点负责运行整个以太坊2.0系统。每个验证者可以具有多种状态，但只有那些被标记为“活跃的（active）”节点才能参与到以太坊2.0协议中。

一旦成为“活跃的”验证者，被选中的验证者就可以通过向信标链（以及之后实现的分片链）提议区块，从而参与到以太坊2.0的协议中。这些验证者同样会加入到委员会（committees）中来对区块进行投票。

**2.提供随机性**

权益证明协议的一个关键要求是必须产生分布式的、可验证的、不可预测的、（适度）不偏不倚的随机性。信标链负责为系统的其他方面提供这种随机性

**3.区块提议者**

在权益证明（PoS）系统中，不存在挖矿，区块提议者是基于上文提到的协议中的随机性来进行随机选择的。

**4.委员会**

权益证明区块链的一个重要的安全保障来源于`委员会（committees，由验证者组成）`，这些委员会负责通过投票来决定哪些区块构成整条链的真实历史记录。信标链依赖于对来自其自身的委员会的投票进行统计，即所谓的`“认证（attestations）”`，以便同意并最终确定信标链的历史记录。

此外，信标链将为每个分片指定一个更小的`分委员会（sub-committee）`，负责在适当的时侯确认分片的提议者是否行为得当。

**5.奖励和处罚**

信标链的另一个管理角色是跟踪和更新验证者的抵押款（deposits）。如果验证者违反规则，那他们将受到处罚，即他们将损失自己押注的32个以太坊的一部分（被削减），并将从系统中剔除出去。验证者也会因为缺席（即不对区块进行投票）而受到轻微处罚，我们称之为`“quadratic leak（二次泄漏）”`。

**6.跨联**

`跨联（crosslinks）`将整个分片系统连接在一起，将每个分片锚定到作为脊柱的信标链上。

<img src="bc16.jpg" width="600" />  

_上图为带有8个分片链（浅绿色）和跨联（浅蓝色线条）的信标链（蓝色）的可视化图。所有链上的最终区块为黄色。时间从左往右递增。（注：中间的为信标链，围绕信标链的为分片链）_

### L1扩容方案
扩容的解决方案分为两大阵营：链下扩容与链上扩容。`链下扩容`即是现在通称的`Layer2`，包括状态通道、Plasma、Rollups等，它们不直接改动区块链本身的规则（区块大小、共识机制等），而是在其之上再架设一层来完成具体的工作；`链上扩容`需要直接修改区块链的基础规则，包括区块大小、共识机制等，这也被称为`L1扩容`。

#### Sharding1.0设计思路
Sharding 1.0希望实现`状态分片`。状态，指的是每个以太坊账户地址的余额，或者是智能合约地址的代码内容和变量数值；它可以被视为一个大账本，所有验证者都需要实时维护、不断更新它。而如果能把这个大账本分成多份（比如64份），验证者也随机分为多组，每组只负责一个账本相关交易的记账，那么速度无疑会加快很多。

具体做法：

- Sharding 1.0计划把以太坊分为64个分片链，每个链至少分配128位验证者（即总验证者数量至少为16384），采用PoS共识。为了协调不同分片链之间的通信，Sharding 1.0的设计中引入了`信标链（Becon Chain）`，可以为整个系统提供统一的时间轴，并且跨分片交易提供最终确定性。
- 每过6.4分钟（称作1个`epoch`），每个分片链所对应的验证者就会被打乱重排。这么做，是为了防止攻击者对分片链的蓄意攻击，毕竟如果1个分片链的验证者一直不变的话，想在这个分片链上作恶只需买通128个验证节点的大多数，这个成本其实并不高。

Sharding1.0问题：

- 每一个epoch过后，每个验证者就需要切换自己所负责的分片链。然而，处理新分片链，就需要拥有新分片链的状态和对应的交易数据，这需要进行一次大范围的网络节点数据同步。这是相当复杂的，实际上，很难保证验证者们能够在规定时间节点完成同步——这会带来网络延迟和糟糕的用户体验。
- 让所有验证者都同步存储所有分片的历史交易数据，又会怎样呢？随着分片后以太坊性能的大幅提升，存储数据的积累势必大大加快，这必然导致对节点的存储性能要求节节攀升，从而降低网络的去中心化程度。
- `MEV`，是`“矿工可提取价值”`的简称。它的存在，是由于验证者（矿工）能够事先看到用户的交易，从而他们可以针对性的提出自己的交易，通过控制交易排序的方式让自己获利。之前的PoW转PoS和Sharding 1.0不仅不能帮忙解决这个问题，反而可能会进一步加剧它的严重程度。这主要是因为ETH发行率降低、验证集中化程度提高等问题导致的。

#### Danksharding设计思路
Danksharding的核心思想，“中心化的出块 + 去中心化的验证 + 抗审查性”。具体而言，它主要做了下面三件事：

1. 通过数据可用性采样（DAS），大大降低了参与网络验证的成本，保持了网络验证的去中心化；
1. 通过出块者-打包者分离（PBS），原先维护以太坊所有历史状态数据的全节点，被进一步划分成两个角色：出块者（Proposer）和打包者（Builder）。这不仅为大量数据包的传输提供了可能，也解决了MEV问题。
1. 通过抗审查清单（Crlist），避免了交易被审查的可能性。

**1.数据可用性采样（Data Availability Sampling）**

`数据可用性`，指的是区块生产者（矿工/验证者）必须公布并提供他们生产区块的交易数据，以便全节点来检查他们的工作。如果区块生产者不提供这些数据的话，全节点就无法检查他们的工作，从而无法确保他们有在遵守区块链规则。

`数据可用性抽样（DAS）`，是一种减轻节点验证数据可用性的负担的方案。它的主要思想是通过一定的数学设计，让验证节点只需要检查部分数据碎片，就可以从概率上证明一个大数据块的可用性，而不需要验证节点去检查全量的数据。这样，对验证节点的性能要求就大大降低了。

(备注：Reed-Solomon（RS）编码，纠删码，一种冗余编码机制)

这个方案也提高了全节点的性能要求，加剧了全节点的中心化程度和MEV问题。

**2.出块者-打包者分离（Proposer-Builder Separation）**
回顾一下现在的以太坊中的全节点，它实际上同时承担了“打包者”和“出块者”两个角色：即，他们既要构建区块的内容，也要负责对出块进行验证+投票。

在PBS中，这两个角色是分开的：

- 原先的全节点若仍想参与区块打包，配置要求将进一步提高，转变为“打包者”；它们通过竞价的方式，来争取下一个区块的打包记账权。
- 从众多低配置要求的验证者中，轮换随机选出一个“出块者”；它根据“价高者得”，来选哪一个“打包者”获得真正的记账权，并即刻获得“打包者”的竞价作为收益。
- 打包者完成打包以后的区块，依然需要全体验证节点来进行验证，以决定其是否合法、有效；但无论验证结果怎样，“出块者”的收益都是已经实现、不需要退回的。

通过这种角色分离，就解决了当前的MEV价值分配问题：打包者依然可以通过交易排序，提取 MEV。但在一个有效市场中，区块打包者们会开始“内卷”，出价到它们能从区块中提取的全部价值（减去它们的摊销成本，如硬件开支等）。而所有的这些价值，都分配给了去中心化的验证者们、而不是那些中心化的打包者们

**3.抗审查清单（crList）**

PBS 增强了区块打包者审查交易的能力：区块打包者可以故意忽略某些合法的交易。

`抗审查清单（crList，CensorshipResistance List）`就是用来解决这个问题的，它要求出块者指定一个在存储池中看到的所有符合条件的交易列表；区块打包者在出价的时候需要证明自己看到了这个列表，打包的时候需要强制包含列表中的交易。

### L2扩容方案
`Layer 2`指基于底层区块链（注：通常也称为“`Layer 1`网络”）的链下网络、系统或技术，目的是为了扩展底层区块链网络。Layer 2网络可以提升任何底层区块链的吞吐量以及其他性能。

当前区块链有三个核心功能，即：执行交易、数据可用性以及达成共识。

* 执行交易——处理并完成交易。衡量指标是区块链每秒可以完成的计算次数（其中包含交易数量）。
* 数据可用性——网络中的节点和验证者需要储存交易、状态以及其他数据。衡量指标是标准存储单位，比如MB和GB等。
* 达成共识——节点和验证者需要针对网络状态和交易排序达成共识。衡量指标是去中心化水平和终局速度，或所有节点针对某一状态变更达成一致意见所需的时间。

Layer 2解决方案大致可以分为两个部分：一个是负责处理交易的网络；另一个是部署在底层区块链上的智能合约，负责解决任何分歧，并将Layer 2网络达成的共识传输到底层区块链进行验证。

不同Layer 2网络在底层区块链上的智能合约实现方式也有所不同，但智能合约的核心功能是一样的， 即：
* 保存并释放资金，转账至Layer 2；
* 收到Layer 2提交的证明，进行验证，解决分歧，并最终确认交易。

#### 状态通道 (State Channels) / 支付通道 (Payment Channels) 
假设Alice和Bob两个人经常需要转账：

一旦建立了支付通道，Alice和Bob都可以通过签名消息在链下进行交易，无需向底层区块链提交交易。Alice可以向Bob付款，Bob也可以向Alice付款，无需任何成本也不存在延时。在这个双向支付通道中，Alice和Bob的交易不会发送到底层区块链。只有当双方都决定关闭通道时，最终交易结果才会被发送至链上进行结算。

<img src="bc17.png" width="600" />  

缺点：用户每与一个人交易都需要建立一次链接，而且能做的操作非常有限。

紧接着就是Optimistic Rollups、Zk Rollup、Plasma和Validium / Zk porter方案。它们的大致的方法论是，把计算执行过程下放至链下，然后再用零知识证明/欺诈证明的方法，在链上证明智能合约运行的正确性。

<img src="bc18.png" width="400" />  

- `Plasma (欺诈证明+数据链下储存)`:在Plasma方案中，智能合约将在链下找一个节点运行，然后再把运行结果放回链上。平时合约的运算结果将直接被放在链上，我们暂且乐观地认为运行结果是正确的；然而，任何人都可以对某次合约的运行结果发起挑战，质疑某次合约运行的正确性，接到挑战后L1就会把合约在链上运行一遍，以检查合约运算是否真的有问题。如果挑战成功，上传错误结果的恶意节点将被罚款，而挑战者则能获得一部分罚款，紧接着整条链将被回滚以消除错误结果。如果有L2的资金想要提取回L1，那么他们需要等待7天 (争议期) ，以让挑战者们有足够的时间检查运行结果以进行挑战。
- `Optimistic Rollups (欺诈证明+数据链上储存)`: Optimistic Rollups使用和Plasma一样的欺诈证明保证合约运行的正确性，并且通过把签名聚合后的原始交易数据上传至无法篡改、无需许可的L1以解决Plasma的数据可用性问题。相应的，因为原始交易数据还是被打包上传至L1，原始交易数据的空间占用成为了性能瓶颈。
- `Validium / Zk porter (零知识证明+数据链下储存)`: 与Plasma类似，为了提升可扩展性，智能合约将在链下运行，然后再把运行结果放回链上。不同于欺诈证明，Validium / Zk porter (这种套技术没有统一名字，Validium和Zk porter是实现这套技术的两个头部产品) 使用零知识证明来证明链下合约运行的正确性。在Validium / Zk porter解决方案中，链下节点除了要运行合约外，还需要额外构建一个零知识证明，并将之上传至链上，让L1验证零知识证明的正确性，从而验证合约运行的正确性。构建一个零知识证明的计算量是较大的，但因为是链下运行，其成本相对来说较低。
- `Zk Rollup (零知识证明+数据链上储存)`:Zk Rollup使用和Validium / Zk porter一样的零知识证明来保证合约运行的正确性，并且通过把压缩后的原始交易数据上传进不可篡改、无需许可的L1以解决Validium / Zk porter的数据可用性问题。

# Web3.0应用架构
<img src="bc12.png" width="600" />  

1. Provider＆Signer
   1. Every Ethereum client (i.e. provider) implements a JSON-RPC specification. This ensures that there’s a uniform set of methods when frontend applications want to interact with the blockchain. If you need a primer on JSON-RPC, it’s a stateless, lightweight remote procedure call (RPC) protocol that defines several data structures and the rules for their processing.
   2. Once you connect to the blockchain through a provider, you can read the state stored on the blockchain. But if you want to write to the state, there’s still one more thing you need to do before you can submit the transaction to the blockchain— “sign” the transaction using your private key.
   3. Metamask is a tool that makes it easy for applications to handle key management and transaction signing. It’s pretty simple: Metamask stores a user’s private keys in the browser, and whenever the frontend needs the user to sign a transaction, it calls on Metamask.
   4. Metamask also provides a connection to the blockchain (as a “provider”) since it already has a connection to the nodes provided by Infura since it needs it to sign transactions. In this way, Metamask is both a provider and a signer. 
2. Storage
   1. Keep in mind that, with Ethereum, the user pays every time they add new data to the blockchain. That’s because adding a state to the decentralized state machine increases the costs for nodes that are maintaining that state machine.
   2. Asking users to pay extra for using your DApp every time their transaction requires adding a new state is not the best user experience. One way to combat this is to use a decentralized off-chain storage solution, like IPFS or Swarm.
3. Querying
   1. You can use the Web3.js library to query and listen for smart contract events. You can listen to specific events and specify a callback every time the event is fired.
   2. The Graph is an off-chain indexing solution that makes it easier to query data on the Ethereum blockchain. The Graph allows you to define which smart contracts to index, which events and function calls to listen to, and how to transform incoming events into entities that your frontend logic (or whatever is using the API) can consume. It uses GraphQL as a query language, which many frontend engineers love because of how expressive it is compared to traditional REST APIs.
4. Scaling
   1. One popular scaling solution is Polygon, an L2 scaling solution. Instead of executing transactions on the main blockchain, Polygon has “sidechains” that process and execute transactions. A sidechain is a secondary blockchain that interfaces with the main chain.
   2. Other examples of L2 solutions are Optimistic Rollups and zkRollups. The idea here is similar: We batch transactions off-chain using a “rollup” smart contract and then periodically commit these transactions to the main chain.

[1]:https://bitcoin.org/bitcoin.pdf "《Bitcoin: A Peer-to-Peer Electronic Cash System》"
[2]:http://lamport.azurewebsites.net/pubs/byz.pdf "The Byzantine Generals Problem"
[3]:https://www.useweb3.xyz/gas?source=ethgas.watch&referrer=ethgas.watch
[4]:https://eips.ethereum.org/
[5]:https://eips.ethereum.org/EIPS/eip-20
[6]:https://cosmos.network/