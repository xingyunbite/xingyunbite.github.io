---
title: 解析钱包KeyStore文件
comments: false
date: 2018-03-25 23:58:04
categories: 钱包
tags:
- ciscolxh
- 钱包
img:
---
无论是从[MyEtherWallet](https://.com/) ，还是从市场中常见的钱包创建一个钱包，或是自己搭建一个主链，都可以导出一个keyStore 文件，
在刚接触钱包就知道keyStore是钱包加密后的表现形式。今天我们来看一看他是怎么加密钱包的。

首先看一下keyStore的样子

```
{
　　"version":3,
　　"id":"0010d1fd-fcaf-425e-8a34-990388be3ed9",
　　"address":"address",
　　"Crypto":{
　　　　"ciphertext":"73e2dd32843ab58356b93248bcde001db8d6aff091d888d1631ede1261bc3e403f",
　　　　"cipherparams":{
　　　　　　"iv":"469f8eb7b10f3ca5f50ec75a8949c0ed"
　　　　},
　　　　"cipher":"aes-128-ctr",
　　　　"kdf":"scrypt",
　　　　"kdfparams":{
　　　　　　"dklen":32,
　　　　　　"salt":"0a48fe56ed1fe152b6f85ab6ed537d819aa79782b45ab5b586f04315917501bd",
　　　　　　"n":262144,
　　　　　　"r":8,
　　　　　　"p":1
　　　　},
　　　　"mac":"3efa6c00d39b4f9719fea0486c32063711d3424465df4e7a9954c8abe5336355"
　　}
}
```
从这个文件中我们可以看出KeyStore生成的是一个JSON文件。并且一些重要的内容都保存在了"Crypto"下

再看一Crypto的参数
* 首先看到cipher这个参数为aes-128-ctr 说明是用的AES-128算法加密模式为ctr
*  cipherparams参数里面包含了一个iv，那么就知道这个是加密参数向量了。
*  ciphertext ，通过名字就能知道这是加密后的密文
* kdf （key derivatioin function）秘钥生成函数 
* kdfparams kdf的一些重要参数
* mac 用于验证密码。

我们在使用AES加密的过程中知道，AES有加密模式，向量，密码。上面kdf又是什么鬼。

在这次AES加密中没有直接使用我们输入的密码去做加密。而是用我们输入的密码使用kdf派生出来一个秘钥。然后通过这个派生秘钥去加密

最后一个参数mac是校验我们输入的密码和密文是否匹配。

#### 所以加密的完整流程是这样
1. 生成一个随机数，作为盐，对密码进行加密生成派生秘钥
2. 然后从派生秘钥中截取16位字符作为密码
3. 再生成一个16位的随机数作为向量，
4. 然后拿着生成的密码，向量，私钥进行加密得到密文
5. 最后拿着生成的密文，和派生秘钥做hase处理作为校验码

####    看一下代码实现
    
    
```
byte[] salt = generateRandomBytes(32);
        byte[] derivedKey = generateDerivedScryptKey(
                password.getBytes(Charset.forName("UTF-8")),
                salt,
                N_STANDARD,
                R,
                P_STANDARD, DKLEN
        );
        byte[] encryptKey = Arrays.copyOfRange(derivedKey, 0, 16);
        byte[] iv = generateRandomBytes(16);
        byte[] privateKeyBytes = ByteUtil.bigIntegerToBytes(new BigInteger(privateKey, 16));
        byte[] cipherText = performCipherOperation(Cipher.ENCRYPT_MODE, iv, encryptKey, privateKeyBytes);
        byte[] mac = generateMac(derivedKey, cipherText);
```


```
private static byte[] performCipherOperation(int encryptMode, byte[] iv, byte[] encryptKey, byte[] privateKeyBytes) throws Exception {
        try {
            IvParameterSpec ivParameterSpec = new IvParameterSpec(iv);
            Cipher cipher = Cipher.getInstance("AES/CTR/NoPadding");
            SecretKeySpec secretKeySpec = new SecretKeySpec(encryptKey, "AES");
            cipher.init(encryptMode, secretKeySpec, ivParameterSpec);
            return cipher.doFinal(privateKeyBytes);
        } catch (Exception e) {
            return throwException(e);
        }
    }

```


```
  private static byte[] generateMac(byte[] derivedKey, byte[] cipherText) {
        byte[] result = new byte[16 + cipherText.length];
        System.arraycopy(derivedKey, 16, result, 0, 16);
        System.arraycopy(cipherText, 0, result, 16, cipherText.length);
        return Hash.sha3(result);
    }
```

#### 首先我们分析解密需要用的得内容
* AES是对称加密，所以我们需要加密时候的加密秘钥

>>  
    我们的加密秘钥是通过我们输入的密码用fdf的算法生成的，而生成的KeyStore里保存了完整的keyStore信息。所以我们还可以通过kdf以及加密时候的参数对这个算法达到我们要的派生秘钥
    
>>    
```
WalletEntity.CryptoBean.KdfparamsBean scryptKdfParams = crypto.getKdfparams();
        int dklen = scryptKdfParams.getDklen();
        int n = scryptKdfParams.getN();
        int p = scryptKdfParams.getP();
        int r = scryptKdfParams.getR();
        byte[] salt = Numeric.hexStringToByteArray(scryptKdfParams.getSalt());
        derivedKey = generateDerivedScryptKey(
                password.getBytes(Charset.forName("UTF-8")), salt, n, r, p, dklen);
```

 * 要获得加密后的密文

    上面对KeyStore结构分析时候已经找到，他就存在ciphertext这个字段下
    
 * 加密时候用到的向量
 
     存在cipherparams对象下的iv属性中

#### 所以解密的完整流程是这样

1. 获取到密文
2. 获取到向量
3. 获取到加密时候用的私钥
4. 校验密码
5. 输出私钥

####  展示下实现代码


```
WalletEntity walletEntity = JsonUtils.deserialize(keyStore, WalletEntity.class);
        WalletEntity.CryptoBean crypto = walletEntity.getCrypto();
        byte[] mac = ByteUtil.hexStringToBytes(crypto.getMac());
        byte[] iv = ByteUtil.hexStringToBytes(crypto.getCipherparams().getIv());
        byte[] cipherText = ByteUtil.hexStringToBytes(crypto.getCiphertext());
        byte[] derivedKey;

        WalletEntity.CryptoBean.KdfparamsBean scryptKdfParams = crypto.getKdfparams();
        int dklen = scryptKdfParams.getDklen();
        int n = scryptKdfParams.getN();
        int p = scryptKdfParams.getP();
        int r = scryptKdfParams.getR();
        byte[] salt = Numeric.hexStringToByteArray(scryptKdfParams.getSalt());
        derivedKey = generateDerivedScryptKey(
                password.getBytes(Charset.forName("UTF-8")), salt, n, r, p, dklen);
        byte[] derivedMac = generateMac(derivedKey, cipherText);
        if (!Arrays.equals(derivedMac, mac)) {
            //这对密码校验 验证是否输入的密码正确
            throw new Exception("Invalid password provided");
        }
        byte[] encryptKey = Arrays.copyOfRange(derivedKey, 0, 16);
      
        byte[] privateKey = performCipherOperation(Cipher.DECRYPT_MODE, iv, encryptKey, cipherText);
        return ByteUtil.toHexString(privateKey);
``` 
