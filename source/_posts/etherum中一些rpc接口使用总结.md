---
title: etherum中一些rpc接口使用总结
comments: false
date: 2018-04-08 19:53:00
categories: 区块链
tags:
- ironman
- 区块链
img:
---
都是基于eth的rpc接口
通过geth attach到ipc上就可以使用js实现的web3接口了

```
geth attach ipc:geth.ipc
```

会有这几个模块可用

```
modules: admin:1.0 clique:1.0 debug:1.0 eth:1.0 miner:1.0 net:1.0 personal:1.0 rpc:1.0 shh:1.0 txpool:1.0 web3:1.0
```

## 获取区块链上的交易信息

### 获取当前区块高度

```
> eth.blockNumber
139552
```

当对区块链上的信息一无所知，也没有要查的区块，交易等等这个方法是一个好的开始

### 通过区块高度获取区块数据：

```
> eth.getBlock(139915)
{
  difficulty: 2,
  extraData: "0x78796274000000000000000000000000000000000000000000000000000000008d8646efe4d2759af7c43aa3d434546ef15f9806e9cbab5b114c899828c6a72448553a7aa715f446325c893677fb8924332e12f8920793f51bba8126ce72416101",
  gasLimit: 4712388,
  gasUsed: 21000,
  hash: "0x37b41662f06762fb1c550e8fa6e77acba1e04310222b57a0f6b1617672a66ce2",
  logsBloom: "0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
  miner: "0x0000000000000000000000000000000000000000",
  mixHash: "0x0000000000000000000000000000000000000000000000000000000000000000",
  nonce: "0x0000000000000000",
  number: 139915,
  parentHash: "0xf58618117b59dca82c55db1aec1d1faaf07b31e8c16c99bdd7b46d0a51829cfb",
  receiptsRoot: "0x056b23fbba480696b65fe5a59b8f2148a1299103c4f57df839233af2cf4ca2d2",
  sha3Uncles: "0x1dcc4de8dec75d7aab85b567b6ccd41ad312451b948a7413f0a142fd40d49347",
  size: 725,
  stateRoot: "0xf2cb3e3a4ce4cf9b5e11750952ef0c697bddb80b5f1a8b77fc70905a34df212c",
  timestamp: 1522581583,
  totalDifficulty: 279831,
  transactions: ["0xd608e02de2314c8b574e60be4d6726a156c5d8f8b88c897fc03a5eb3d82ccec4"],
  transactionsRoot: "0x8c9e01d2afe57484bfceef90103674dfdbc56adad74a1b179c5eafe35797ce93",
  uncles: []
}
```

这里有几个变量可能用的比较多

* number：区块高度，可以通过区块高度获取到区块信息，区块高度并不能唯一确定一个区块，当出现分叉的时候有可能通过同一个区块高度获取到不同的区块数据。不过大多数情况下通过区块高度获取到的区块信息都不会变
* timestamp：区块被打包的时间，当需要把这个里打包的相关交易入库的时候，这个时间就是这写交易被打包的时间。
* transactions：当前区块打包的交易的hash列表，是一个数组，可以通过遍历这个数组取出这个区块中被打包的交易

### 通过交易hash获取交易的具体内容

```
> eth.getTransaction("0xd608e02de2314c8b574e60be4d6726a156c5d8f8b88c897fc03a5eb3d82ccec4")
{
  blockHash: "0x37b41662f06762fb1c550e8fa6e77acba1e04310222b57a0f6b1617672a66ce2",
  blockNumber: 139915,
  from: "0x5ca923f9561198d64a328d41ae0ded2f15b3e388",
  gas: 90000,
  gasPrice: 10000000000,
  hash: "0xd608e02de2314c8b574e60be4d6726a156c5d8f8b88c897fc03a5eb3d82ccec4",
  input: "0x",
  nonce: 50,
  r: "0xc19aff35724f52659afd73ec1066a196fce999917be7289d7f778ce4b45b0ee0",
  s: "0x72170437e0f7fa3e440b42585871fface521e1f8a457cc5cf4275eb4cf7ddb20",
  to: "0x77446f036e90b866566770a7e742e72df2f934bd",
  transactionIndex: 0,
  v: "0xa95",
  value: 5000000000000000000
}
```

返回的数据就包含了一笔交易的全部内容

* from：从哪个地址转出
* to：数据接收方。当时一笔eth的交易的时候to的值就是用户地址，当是一笔token交易的时候to的值是token地址而不是用户地址
* value：转账金额，现在看到的是5乘以10的18次方，虚拟币里用精度来标示小数的概念，eth的精度是18。5个eth转换成最小单位就是5乘以10的18次方。
* blockHash：被打包的区块的hash，可以通过这个hash获取到整个区块的数据。
* blockNumber：区块高度，也可以获取到区块信息，当然也可以通过当前整个区块链的区块高度减去这比交易的区块高度得到这比交易的确认数。
* hash：交易的hash，可以通过这个值获取到这笔交易的完整信息。
* gas：这笔交易消耗的gas
* gasPrice：这笔交易的gasPrice，gas乘以gasPrice就是这笔交易的手续费。
* input：跟随着比交易附带的数据，当是一笔token交易的时候，用户地址和value以及调用的token方法都会在input里。

### 发送一笔eth的交易：

从coinbase地址给账户2地址转5个eth

```
eth.sendTransaction({from:eth.coinbase, to:eth.accounts[1], value: web3.toWei(5, "ether")})
0xd608e02de2314c8b574e60be4d6726a156c5d8f8b88c897fc03a5eb3d82ccec4
//转账到d1ade25ccd3d550a7eb532ac759cac7be09c2719这个地址
eth.sendTransaction({from:eth.coinbase, to:"d1ade25ccd3d550a7eb532ac759cac7be09c2719", value: web3.toWei(5, "ether")})
```

真实转账的时候可能需要输入密码，具体的可以参考：https://github.com/ethereum/go-ethereum/wiki/Sending-ether

如何构造一笔token（智能合约）交易

* to：的值不再是转账的账户地址而是token地址
* input：合约方法名的hash前几个字符，合约参数比如”0xa9059cbb000000000000000000000000aefd52d5a5fd165c4f73b413261b8e20cada84c0000000000000000000000000000000000000000000000000000000000000000a”就构造了一个transfer（address， amount）转账到address地址的一个input参数。

具体的参数如何构造可以看这篇文章：



一般用浏览器工具来发送token交易比较简单：
http://remix.ethereum.org/#optimize=false&version=soljson-v0.4.8+commit.60cc1668.js
在右边的runtab下可以选择environment，即链接自己的测试链
![](/images/browser_debug_token_20180408.png)




可以创建一个token
比如选择SGCC，填入参数："2100000000","token name”, "8”,"token_symbol"
或者添加已经有的token

这里还可以修改智能合约的代码，创建自己的智能合约，非常方便


