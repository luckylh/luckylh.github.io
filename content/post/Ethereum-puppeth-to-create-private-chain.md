---
title: "Ethereum - 使用Puppeth创建以太坊私有链"
date: 2017-11-02T21:06:31+08:00
description: "Puppeth是以太坊go客户端工具中提供的一个实用工具. 通过puppeth可以在自有环境中快速搭建起以太坊私链."
tags: [ "Ethereum", "blockchain" ]
categories:
  - "Blockchain"
draft: false
---

Puppeth是以太坊go客户端工具中提供的一个实用工具. 通过puppeth可以在自有环境中快速搭建起以太坊私链.

## 安装Ethereum Go 客户端(geth)

下载并安装Ethereum的Go客户端, 目前最新稳定版为1.7.2

https://ethereum.github.io/go-ethereum/downloads/

安装完成go-ethereum后, go客户端中就带有puppeth

## 搭建以太坊私有链

本文档搭建3个Ethereum节点, 分别为: node1, node2, node3.

这3个节点可以分别安装在不同电脑上, 也可以安装在同一台电脑上.

以下配置说明都是基于在同一台电脑上, 电脑环境:

- OS: macOS High Sierra 10.13
- go: go1.9.1 darwin/amd64
- geth: 1.7.2

#### 环境准备

分别创建3个节点目录, 每个目录下各自保存各节点的账号, 区块链数据
```shell
mkdir ethereum
cd ethereum
mkdir node1
mkdir node2
mkdir node3
```

#### 建立账号

为每个节点各自建立一个账号. 运行命令后会提示输入密码用于保护你的以太坊账号私钥. 这里直接回车跳过密码设置. 生成账号后, 命令返回账号对应的地址.
```shell
geth --datadir ./node1/data account new
Your new account is locked with a password. Please give a password. Do not forget this password.
Passphrase:
Repeat passphrase:
Address: {117b88219433dcbe8d24e961283bf57a800b0e91}
```
使用相同命令给node2, node3各自创建新账号
```shell
geth --datadir ./node2/data account new
geth --datadir ./node3/data account new
```
记录下新生成的账号, 后续测试私链可以用到. (万一没有记录,也没有关系, 后续可以通过geth console查询)

- account1 address(node1):117b88219433dcbe8d24e961283bf57a800b0e91
- account2 address(node2):2ceb27a7d194e32dbe30f13dfb33b558c758e044
- account3 address(node3):7fa7dff92c1a0a3384975f7c5dd8272ec5349712

#### 建立创世块(Genesis Block)

创世块是区块链中第一个区块(唯一无需引用前个区块的区块).
通过运行puppeth, 来生成创世区块. 运行puppeth后首先提示输入私链名称.
```shell
puppeth
+-----------------------------------------------------------+
| Welcome to puppeth, your Ethereum private network manager |
|                                                           |
| This tool lets you create a new Ethereum network down to  |
| the genesis block, bootnodes, miners and ethstats servers |
| without the hassle that it would normally entail.         |
|                                                           |
| Puppeth uses SSH to dial in to remote servers, and builds |
| its network components out of Docker containers using the |
| docker-compose toolset.                                   |
+-----------------------------------------------------------+

Please specify a network name to administer (no spaces, please)
>frank-private-chain
Sweet, you can set this via --network=frank-private-chain next time!
```
输入完私链名称后, 出现选择菜单. 选择2: 配置新创世块
```shell
What would you like to do? (default = stats)
 1. Show network stats
 2. Configure new genesis
 3. Track new remote server
 4. Deploy network components
```
出现子菜单, 选择私链的共识机制(Consensus Protocols).

- proof-of-work(PoW): 工作量证明, 通过算力证明来达成共识
- proof-of-authority(PoA): 权威证明, 通过预先设定的权威节点来负责达成共识

这里选择PoW的认证方式, 和目前以太坊主链保持一致.(PoA配置方式: 请点击[传送门](#poa))
```shell
Which consensus engine to use? (default = clique)
 1. Ethash - proof-of-work
 2. Clique - proof-of-authority
> 1
```
选择是否需要初始化就给指定账号ether, 这里可以输入之前建立账号的地址.直接回车结束输入
```shell
Which accounts should be pre-funded? (advisable at least one)
> 0x
```
输入私链ID, 直接输入回车,已默认随机数作为私链ID
```shell
Specify your chain/network ID if you want an explicit one (default = random)
>
```
创世块中可以输入个人个性化信息, 这里可以随便输入字符.
```shell
Anything fun to embed into the genesis block? (max 32 bytes)
>This is Frank's private chain.
```
回到主菜单, 重新选择2
```shell
What would you like to do? (default = stats)
 1. Show network stats
 2. Manage existing genesis
 3. Track new remote server
 4. Deploy network components
>
```
选择导出创世块配置文件
```shell
 1. Modify existing fork rules
 2. Export genesis configuration
>2
```
在提示创世块配置文件位置时, 直接输入回车, 默认当前目录生成创世块配置文件
```shell
Which file to save the genesis into? (default = frank-private-chain.json)
>
```
创世块配置文件生成后, 会回到主菜单, 直接按Ctrl-C退出puppeth.

#### 初始化/启动私链节点

通过如下命令初始化各个节点的数据. 如下命令
```shell
geth --datadir node1/data init frank-private-chain.json
geth --datadir node2/data init frank-private-chain.json
geth --datadir node3/data init frank-private-chain.json
```
开启3个终端, 每个终端中分别启动各自的以太坊节点
```shell
geth --datadir ./node1/data --port 2001
geth --datadir ./node2/data --port 2002
geth --datadir ./node3/data --port 2003
```
参数说明:

- datadir: 自定义区块链数据存储路径
- port: 网络监听端口(默认为30303, 安装在不同电脑上, 端口可相同. 在同一台电脑上则需要设置不同端口号)

### 添加节点

私链中设置的节点, 默认是无法互相发现并连接, 需要手工添加节点.(以太坊主链则通过在代码中预设了启动节点, 可以启动发现其他节点)

登录node1, 获取节点地址
```shell
geth attach ipc:node1/data/geth.ipc

instance: Geth/v1.7.2-stable-1db4ecdc/darwin-amd64/go1.9.1
coinbase: 0x117b88219433dcbe8d24e961283bf57a800b0e91
at block: 12 (Tue, 07 Nov 2017 22:59:42 CST)
 datadir: /Users/liuhao/Downloads/ethereum/node1/data
 modules: admin:1.0 debug:1.0 eth:1.0 miner:1.0 net:1.0 personal:1.0 rpc:1.0 txpool:1.0 web3:1.0

> admin.nodeInfo.enode
"enode://8168f0ffc7b29680c3afaaf5e88c63bf5f89d147fa8a84ca11e5205fc27886bafb6bfc6a1bde78ed2f1c02fd9eea8b53120bc548276d3f7de58e3bf4a02dd643@[::]:2001"
```
在返回的节点地址中, 地址组成为"enode://<节点id>@IP地址:端口"

将IP地址部分替换成127.0.0.1(如果是安装在不同电脑上, 则替换成当前节点所在电脑的IP地址)

分别登录node2, node3, 添加链接到node1
```shell
geth attach ipc:node2/data/geth.ipc
>admin.addPeer("enode://429dd761506640fd2d581927b969ecdd6876437118a6c14bf8d99096e7d98d1cd2af2da9f93614263a773bf8ec1f6e9cbf9e75b686e113341e4ced78b0673b84@127.0.0.1:2001”)
```
再次登录node1, 查看是否节点链接正确

- node1应该链接上2个节点(node2, node3)
- node2, node3 各自链接上1个节点(node1)

```shell
geth attach ipc:node1/data/geth.ipc
>net.peerCount
2
```
也可以获得链接节点的详细信息
```shell
geth attach ipc:node1/data/geth.ipc
>admin.peers
[{
    caps: ["eth/63"],
    id: "51fa46b906bbe98a55448d6ec84f20df1ee879123d9cbe3d25ac81982db4fb14a9b42b9f332aa7bd2d1c006f64d15c3740b0e9d1e42bb3858a4fe852d815f860",
    name: "Geth/v1.7.2-stable-1db4ecdc/darwin-amd64/go1.9.1",
    network: {
      localAddress: "127.0.0.1:2001",
      remoteAddress: "127.0.0.1:57710"
    },
    protocols: {
      eth: {
        difficulty: 1048576,
        head: "0x632e6d1e792d4e92e85e717067bb6aa586832db03d79c5decb48d826ff7a715a",
        version: 63
      }
    }
}, {
    caps: ["eth/63"],
    id: "88d8df057ad1c1e863ba91a0f04fd6011bbf91ca9e806a5408a17bf653176064614e78416afbe3227b825f2ddab279e86d348bca1baaa6ae1c30f054a020fa17",
    name: "Geth/v1.7.2-stable-1db4ecdc/darwin-amd64/go1.9.1",
    network: {
      localAddress: "127.0.0.1:2001",
      remoteAddress: "127.0.0.1:57719"
    },
    protocols: {
      eth: {
        difficulty: 1048576,
        head: "0x632e6d1e792d4e92e85e717067bb6aa586832db03d79c5decb48d826ff7a715a",
        version: 63
      }
    }
}]
```

## 测试私链

#### 挖矿

使用node3节点进行挖矿, 挖矿前可以看到account3账户余额为0.

使用miner.start()开始挖矿(geth采用CPU算力挖矿). 等待10到20秒后, 可以使用miner.stop()停止挖矿. 

再次查询余额就可以看到挖矿获得的ether
```shell
geth attach ipc:node3/data/geth.ipc
>eth.getBalance(eth.accounts[0])
0
>miner.start()
null
>miner.stop()
true
>eth.getBalance(eth.accounts[0])
45000000000000000000
```

#### 转账

完成挖矿后, 可以将挖到的ether转账给其他账户. 

登录node3 console, 首先解锁account3账号, 然后将指定金额的ether转账给其他账号\(例子中为转账给account1账号, 如果没有在之前记录account1账号地址, 则可以通过"eth.accounts"获取账号地址信息\).

转账命令运行后会返回transaction id, 通过该transaction id可以查询到转账交易相关信息

```shell
geth attach ipc:node3/data/geth.ipc
>personal.unlockAccount(eth.accounts[0])
>eth.sendTransaction({from:eth.accounts[0], to:"117b88219433dcbe8d24e961283bf57a800b0e91", value:20000})
"0x9da356310da637a8b9856098d95cf08fd9feb687991abbf9bcfc4f1d7b9a08ab"
>eth.getTransaction("0x9da356310da637a8b9856098d95cf08fd9feb687991abbf9bcfc4f1d7b9a08ab")
```

**注:**

- 运行转账命令后, 还需要有节点进行挖矿, 否则该transaction不会被记录保存在私链的区块中.
- 转账后剩余余额会比减去转出ether后还要小一点, 少的差额就是transaction消耗的Gas.

## 其他说明

<span id = "poa"></span>
#### PoA说明

建立创世块时, 也可以选择采用PoA方式的共识算法(不消耗计算力, 可以用于私链测试开发)
```shell
Which consensus engine to use? (default = clique)
 1. Ethash - proof-of-work
 2. Clique - proof-of-authority
> 2
```
选择PoA后, 会提示选择出块的间隔时间. 由于是用于内部测试的私链, 可以将出块时间设置较少, 这里配置5秒.
```shell
How many seconds should blocks take? (default = 15)
> 5
```
设置那个账号作为权威来生成块, 这里输入account3账号作为权威账号
```shell
Which accounts are allowed to seal? (mandatory at least one)
> 0x7fa7dff92c1a0a3384975f7c5dd8272ec5349712
> 0x
```
之后的配置和PoW的配置相同

**挖矿注意:**

PoA共识算法下, 在挖矿之前,需要保持账号unlock.

```shell
geth attach ipc:node3/data/geth.ipc
> personal.unlockAccount(eth.accounts[0])
> miner.start()
```

#### 链接节点参数

可以将启动链接节点添加到命令行参数中, 而不需要再在geth控制台中通过admin.addPeer添加节点
```shell
geth --datadir ./node2/data --port 2002 --bootnodes enode://7b9d6c06ea731c4e3c990472fccf41e80e1f10699c526c3d02b1adf8a5bd1ca51861f10f8d55b95c666e5df607efc21063d81e9e21dbaec800865efdcbfcbe9b@127.0.0.1:2001
geth --datadir ./node3/data --port 2003 --bootnodes enode://7b9d6c06ea731c4e3c990472fccf41e80e1f10699c526c3d02b1adf8a5bd1ca51861f10f8d55b95c666e5df607efc21063d81e9e21dbaec800865efdcbfcbe9b@127.0.0.1:2001
```
