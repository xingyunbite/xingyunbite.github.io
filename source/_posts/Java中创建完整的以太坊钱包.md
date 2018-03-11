---
title: Java中创建完整的以太坊钱包
comments: false
date: 2018-03-11 09:11:02
categories: 钱包
tags: 
- ciscolxh 
- 钱包
img:
---

### Java中创建完整的以太坊钱包

以太坊中生成秘钥分为三步
1. 生成EC私钥
2. 通过私钥推出公钥
3. 通过公钥算出密码

> ###### 以太坊的私钥是一个64位的16进制数。以太坊和比特币一样，都是使用的相同的椭圆曲线算法生成

* 我们用OpenSSL来生成一个钥私，并且显示他的私有和公告部分

```
openssl ecparam -name secp256k1 -genkey -noout | openssl ec -text -noout 

read EC key
Private-Key: (256 bit)
priv:
    00:bc:70:90:97:28:61:e0:92:72:74:c0:0f:b0:1d:
    c6:4c:e0:bc:a9:32:a7:b1:d2:9e:a5:1e:cd:2c:21:
    c2:e3:1d
pub: 
    04:0f:ff:81:4f:59:ac:83:93:2f:5c:6c:74:ea:69:
    d4:75:74:9a:46:9a:f1:6e:eb:aa:2f:2a:59:d2:36:
    5f:33:1f:7d:35:05:f5:57:15:11:3d:91:47:89:2d:
    57:91:75:cc:cd:6e:61:e8:4c:b1:aa:cf:1d:fd:d7:
    86:35:c4:55:c5
ASN1 OID: secp256k1
```
> ###### openssl会在私钥部分加一个0x00的头部，在公钥部分加一个0x04的头部，我们去掉头部把公钥、私钥部分转为我们想要的十六进制格式

* 私钥部分

```
cat Key | grep priv -A 3 | tail -n +2 |  tr -d '\n[:space:]:' | sed 's/^00//'

884f0da48660a3c22257fc36be4210ec6f975924ea399e6478c7596b852f25c4
```

* 公钥部分

```
 cat Key | grep pub -A 5 | tail -n +2 |tr -d '\n[:space:]:' |sed 's/^04//'
 
f35862b021183deb90cb66978fc6f7c4bb5ae3e10d486a8109458e0f1ebbdb4602ed1f864bc91cada3eef79b3fb020313850b27c958cdd7567e5c7b9d7f66780
```
> ###### 地址是取公钥后40个字符，通过keccak-256算法生成

* 地址

```
 cat pub | keccak-256sum -x -l | tr -d ' -' | tail -c 41
 
 82a772a5c62b05cd51683872180a5dffef3a0c3e
```

接下来我们去[myetherwallet](https://www.myetherwallet.com/)验证一下这个钱包地址是否正确

* 导入私钥

![image](/images/address20180311.png)

通过比对发现私钥地址一致。我们在Java中再进行生成对比一下。

首先我们要导入[ethereum](https://github.com/ethereum/ethereumj)

Java中钱包生成公私钥地址的方法都在org.ethereum.crypto.ECKey包中的ECKey类中

通过源码我发现好像并没有提供直接通过私钥生成地址的方法,但是提供了一个可以通过私钥生成ECKey的构造方法，以及通过ECKey获取公钥，私钥，地址的方法

```
 public static ECKey fromPrivate(byte[] privKeyBytes) {
        return fromPrivate(new BigInteger(1, privKeyBytes));
    }
```


```
  @Nullable
    public byte[] getPrivKeyBytes() {
        return this.privKey == null?null:(this.privKey instanceof BCECPrivateKey?ByteUtil.bigIntegerToBytes(((BCECPrivateKey)this.privKey).getD(), 32):null);
    }
```

```
 public ECPoint getPubKeyPoint() {
        return this.pub;
    }
```


```
 public byte[] getAddress() {
        if(this.pubKeyHash == null) {
            this.pubKeyHash = computeAddress(this.pub);
        }

        return this.pubKeyHash;
    }
```

接下来根据这几个方法写一个测试类进行测试校验

![image](/images/test20180311.png)

发现私钥和地址是一样的公钥多了一个04的头部，去掉头部也是完全一样。

那我们怎么生成一个钱包？再看源码还有一个空参构造方法可以构造一个ECKey

```
 public ECKey() {
        this(secureRandom);
    }
```

我们生成一个钱包并在[myetherwallet](https://www.myetherwallet.com/)校验

![image](/images/test_wallet20180311.png
)

![image](/images/wallet20180311.png
)

通过校验发现我们生成的钱包与mywallet中一致。



参考：
1. [Generating a usable Ethereum wallet and its corresponding keys](https://kobl.one/blog/create-full-ethereum-keypair-and-address/)
2. [ethereumj](https://github.com/ethereum/ethereumj)
