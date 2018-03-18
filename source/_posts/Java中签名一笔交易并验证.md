---
title: Java中签名一笔交易并验证
comments: false
date: 2018-03-18 11:29:49
categories: 钱包
tags:
- ciscolxh
- 钱包
img:
---
### Java中签名一笔交易并验证
>上一节讲了Java中创建钱包，这一节我们讲如何在Java中签名一笔交易

准备工作就不做详细介绍了
* 集成Java的环境查看[这里](https://github.com/ethereum/ethereumj)
* 搭建测试的私链查看[这里](http://c60block.com/2018/03/16/%E9%83%A8%E7%BD%B2%E4%BB%A5%E5%A4%AA%E5%9D%8A%E7%A7%81%E6%9C%89%E9%93%BE%E5%B9%B6%E6%8C%96%E7%9F%BF/)

直接切入重点。包里给我们提供了这样的一个方法，来构造一笔交易。

```
 public Transaction(byte[] nonce, byte[] gasPrice, byte[] gasLimit, byte[] receiveAddress, byte[] value, byte[] data, Integer chainId) {
        this.chainId = null;
        this.parsed = false;
        this.nonce = nonce;
        this.gasPrice = gasPrice;
        this.gasLimit = gasLimit;
        this.receiveAddress = receiveAddress;
        if(ByteUtil.isSingleZero(value)) {
            this.value = ByteUtil.EMPTY_BYTE_ARRAY;
        } else {
            this.value = value;
        }

        this.data = data;
        this.chainId = chainId;
        if(receiveAddress == null) {
            this.receiveAddress = ByteUtil.EMPTY_BYTE_ARRAY;
        }

        this.parsed = true;
    }
```

#### 以上参数是什么意思呢？
* nonce 交易序列号，交易序列号默认是0，每增加一笔交易nonce+1
* gasgasLimit 通常我们说的gas
* gasPrice 单位gas需要的价格
* receiverAddress 接收者的地址
* value 要转的代币
* data eth交易可以为空字符串。
* chainId 网络id，主网络是1  我这里测试网络配置了999，根据自身私链设置。

需要注意的是：这里的交易单位都是wei，单位转换参考下表。

 Unit|Wei Value|Wei
--- |--- | ---
wei                 | 1        | 1
Kwei (babbage)      | 1e3 wei  | 1,000
Mwei (lovelace)     | 1e6 wei  | 1000,000
Gwei (shannon)      | 1e9 wei  | 1,000,000,000
microether (szabo)  | 1e12 wei | 1,000,000,000,000
milliether (finney) | 1e15 wei | 1,000,000,000,000,000
ether               | 1e18 wei | 1,000,000,000,000,000,000

知道这些概念后我们就可以签名一笔交易。

现在我们模拟从accounts[0]地址给accounts[1]地址发送一笔交易，数额为1eth。并且携带了交易信息。

1. 首先获取nonce值

```
> eth.getTransactionCount(eth.accounts[0])
0
> 
```
> 得到nonce为0

2. 获取gas

```
> eth.estimateGas({from:eth.accounts[0],to:eth.accounts[1],data:"0x0000000000000000000000001b2fdc6e4453fcc9f0e115c8bc8320b1e084af5f000000000000000000000000000000000000000000000000000001d1a94a2000",value:5000000000000000000})
22856
```
> 得到gas为22856

3. 获取推荐的gasPrice

```
> eth.gasPrice
18000000000
```
4. 获取接收者的地址

```
> eth.accounts[0]
"0xcd2a3d9f938e13cd947ec05abc7fe734df8dd826"
> eth.accounts[1]
"0x94e94dcab84ad8805c2ebb843c1cf06393cca620"
> 
```
> 当前我在私有链上导入两个地址，然后用"0xcd2a3d9f938e13cd947ec05abc7fe734df8dd826"这个地址挖矿，挖到币发送给"0x94e94dcab84ad8805c2ebb843c1cf06393cca620"
所以获取到的接收地址为：=="0x94e94dcab84ad8805c2ebb843c1cf06393cca620"==

5. data在以太坊交易中可以为“”，这里是做测试我随便填了一组字符串。
6. value为1eth，上面说了这里交易设置的值为wei，所以这里是1000000000000000000
7. chainId为私有链配置的网络id我这里为999

还有在签名的时候需要用到私钥。我这里测试账号的私钥为"c85ef7d79691fe79573b1a7064c19c1a9819ebdbd1faaab1a8ec92344438aaf4"

接下来我们对需要发起的这笔交易进行签名。


```
String s ="0000000000000000000000001b2fdc6e4453fcc9f0e115c8bc8320b1e084af5f000000000000000000000000000000000000000000000000000001d1a94a2000";
        BigInteger bigInteger =  new BigInteger("94e94dcab84ad8805c2ebb843c1cf06393cca620",16);
        //nonce
        byte[] nonce = ByteUtil.intToBytesNoLeadZeroes(0);
        //
        byte[] gasPrice = ByteUtil.bigIntegerToBytes(new BigInteger("18000000000"));
        //
        byte[] gas = ByteUtil.longToBytesNoLeadZeroes(22856);
        //
        byte[] address = ByteUtil.bigIntegerToBytes(bigInteger);
        //交易金额
        byte[] value = ByteUtil.bigIntegerToBytes(new BigInteger("1000000000000000000"));
        //字符串转成数组上传
        byte[] data = ByteUtil.hexStringToBytes(s);
        Transaction tx = new Transaction(
                nonce,
                //
                gasPrice,
                //gas限度
                gas,
                //接收者地址
                address,
                //发送的金额
                value,
                //携带消息
                data,
                //发送到主链当中
                999
        );

        //开始签名
        tx.sign(ECKey.fromPrivate(new BigInteger("c85ef7d79691fe79573b1a7064c19c1a9819ebdbd1faaab1a8ec92344438aaf4",16)));
        byte[] rawTx = tx.getEncoded();
        //打印出签名后的值
        System.out.println("0x"+ Hex.toHexString(rawTx));
```


```
得到签名后的数据为：
0xf8af80850430e234008259489494e94dcab84ad8805c2ebb843c1cf06393cca620880de0b6b3a7640000b8400000000000000000000000001b2fdc6e4453fcc9f0e115c8bc8320b1e084af5f000000000000000000000000000000000000000000000000000001d1a94a20008207f2a066674d401df4f4150f0650a54e3932308cbe5f121bf5b42fbfc7b0fb8d7ca0d7a04d575c545ad23fb91f71091af2c7807a88eff3de2b2604742e1188ce2c0bc216
```

然后我们在这私有链上广播出去这笔交易看是否签名正确。


```
> eth.sendRawTransaction("0xf8af80850430e234008259489494e94dcab84ad8805c2ebb843c1cf06393cca620880de0b6b3a7640000b8400000000000000000000000001b2fdc6e4453fcc9f0e115c8bc8320b1e084af5f000000000000000000000000000000000000000000000000000001d1a94a20008207f2a066674d401df4f4150f0650a54e3932308cbe5f121bf5b42fbfc7b0fb8d7ca0d7a04d575c545ad23fb91f71091af2c7807a88eff3de2b2604742e1188ce2c0bc216")
INFO [03-17|08:13:31]Submitted transaction                    fullhash=0x256b52bb9995e95c51065cef83e8e7170dd934cb77e174a8999f0c3d674c78e7 recipient=0x94E94dcab84Ad8805c2EbB843C1cf06393cca620
"0x256b52bb9995e95c51065cef83e8e7170dd934cb77e174a8999f0c3d674c78e7"
```
我们查看accounts[1]地址现在是否收到了1eth
```
> eth.getBalance(eth.accounts[1])
1000000000000000000
> web3.fromWei(eth.getBalance(eth.accounts[1]),"ether")
1
```
查看交易结果

```
> eth.getTransaction("0x256b52bb9995e95c51065cef83e8e7170dd934cb77e174a8999f0c3d674c78e7")
{
  blockHash: "0x53bb7e8ef200dfa7dd2e72cba30bc9bcfb9e2de7bb49da1c35f49dbd18dd7c14",
  blockNumber: 4,
  from: "0xcd2a3d9f938e13cd947ec05abc7fe734df8dd826",
  gas: 22856,
  gasPrice: 18000000000,
  hash: "0x256b52bb9995e95c51065cef83e8e7170dd934cb77e174a8999f0c3d674c78e7",
  input: "0x0000000000000000000000001b2fdc6e4453fcc9f0e115c8bc8320b1e084af5f000000000000000000000000000000000000000000000000000001d1a94a2000",
  nonce: 0,
  r: "0x66674d401df4f4150f0650a54e3932308cbe5f121bf5b42fbfc7b0fb8d7ca0d7",
  s: "0x4d575c545ad23fb91f71091af2c7807a88eff3de2b2604742e1188ce2c0bc216",
  to: "0x94e94dcab84ad8805c2ebb843c1cf06393cca620",
  transactionIndex: 0,
  v: "0x7f2",
  value: 1000000000000000000
}
```

由此结果可以验证我们签名的交易正确。













 
