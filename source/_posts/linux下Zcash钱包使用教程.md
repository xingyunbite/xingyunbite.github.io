---
title: linux下Zcash钱包使用教程
comments: false
date: 2018-02-08 19:04:15
categories: 矿池
tags: 
- lucas556 
- zcash 
- 矿池
img:
---

>Zcash (ZEC)是首个使用零知识证明机制的区块链系统，它可提供完全的支付保密性，同时仍能够使用公有区块链来维护一个去中心化网络。与比特币相同的是，Zcash代币（ZEC）的总量也是2100万，不同之处在于，Zcash交易自动隐藏区块链上所有交易的发送者、接受者及数额。只用那些拥有查看秘钥的人才能看到交易的内容。用户拥有完全的控制权，他们可自行选择向其他人提供查看秘钥.  ZCash 是 bitcoin 的分支，保留了 bitcoin 原有的模式，基于比特币 0.11.2 版代码修改的。 ZCash 钱包资金分 2 种：透明资金、私有资金，透明资金类似比特币资金；私有资金加强了 隐私性，涉及到私有资金的交易是保密不可查的，透明资金与透明资金的交易是公开可查的.

>目前Zcash流通市值在12亿美元左右，每日交易额也高达1亿美元，其使用zero-knowledge proof（零知识证明）使得数字化货币更加具有安全性；隐私性而备受市场关注。

现在zcash官方提供的钱包是linux版本的，而原因是zcash团队没人熟悉gui，不过zcash官方提供了编译后的可执行程序，这里也不用我们来进行编译了。

下载地址 ： https://z.cash/downloads/zcash-1.0.14-linux64.tar.gz
hash     :  352ea2a67ae3484046a6bd43af9a5ce125e2d103a6a32ac71805658918f7076a

下载后请务必进行哈希值验证，以保证文件的真实和准确性。

tar -xvf zcash-1.0.14-linux64.tar.gz       //解压缩
mv -t /usr/local/bin/ zcash-1.0.14/bin/*   //移动可执行文件

										   现在我们已经安装了zcash，运行下面的命令下载 key ，用于创建和验证参数。

										   zcash-fetch-params

										   这里由于国内网络的原因，可能会失败，请多试几次。

										   验证通过后，我们需要对客户端进行配置。

										   mkdir ~/.zcash //创建zcash目录

										   vim zcash.conf 

										   以上按照个人配置的不同，设置不同的配置文件，下面提供一个配置供参考。

										   //这是测试链接的配置
										   addnode=testnet.z.cash    //节点
										   rpcuser=test              //rpc用户名
										   rpcpassword=test          //rpc密码
										   gen=0                     //屏蔽cpu挖矿
										   testnet=1                 //开启测试链
										   rpcallowip=100.100.60.10 //允许访问的IP
										   rpcport=8333             //rpc端口号
										   equihashsolver=tromp     //指定算法，非挖矿钱包可以删除


										   好了，保存后输入zcashd就可以启动zcash客户端了，也可以使用zcashd -daemon在后台运行zcash客户端,在shell输入zcash-cli getinfo命令就可以看到网络和块信息了。

										   下面提供一些zcash节点常用命令：

										   zcash-cli getinfo                           //显示节点信息
										   zcashd -daemon                              //后台启动zcash守护
										   zcash-cli getnetworkhashps                  //获取全网算力
										   zcash-cli z_getnewaddress                    //生成一个Z-addr
										   zcash-cli getnewaddress                      //生成一个t-addr                   
										   zcas-cli getblockhash                       //区块高度
										   zcash-cli getaddressesbyaccount ""         //显示所有t-addr钱包
										   zcash-cli z_listaddresses                   //显示所有Z-addr钱包
										   zcash-cli z_getbalance ""                   //z钱包余额

										   如果有其他的需要，可以使用zcash-cli help来查看zcash的全部命令。


										   附1：

										   zcash钱包配置：

										   //这里是zcash钱包主链配置

										   rpcuser=rpc用户名
										   rpcpassword=rpc密码
										   rpcport=rpc端口
										   rpcallowip=允许链接rpc ip地址
										   server=1        //打开服务
										   daemon=1        //后台运行守护
										   mainnet=1       //主链
										   addnode=mainnet.z.cash //主链节点

										   附2：

										   //zcash命令

										   == Blockchain ==
										   getbestblockhash
										   getblock "hash|height" ( verbose )
										   getblockchaininfo
										   getblockcount
										   getblockhash index
										   getblockheader "hash" ( verbose )
										   getchaintips
getdifficulty
getmempoolinfo
getrawmempool ( verbose )
		gettxout "txid" n ( includemempool )
		gettxoutproof ["txid",...] ( blockhash )
		gettxoutsetinfo
		verifychain ( checklevel numblocks )
		verifytxoutproof "proof"

		== Control ==
		getinfo
		help ( "command" )
		stop

		== Disclosure ==
		z_getpaymentdisclosure "txid" "js_index" "output_index" ("message") 
		z_validatepaymentdisclosure "paymentdisclosure"

		== Generating ==
		generate numblocks
		getgenerate
		setgenerate generate ( genproclimit )

		== Mining ==
		getblocksubsidy height
		getblocktemplate ( "jsonrequestobject" )
		getlocalsolps
		getmininginfo
		getnetworkhashps ( blocks height )
		getnetworksolps ( blocks height )
		prioritisetransaction <txid> <priority delta> <fee delta>
		submitblock "hexdata" ( "jsonparametersobject" )

		== Network ==
		addnode "node" "add|remove|onetry"
		clearbanned
		disconnectnode "node" 
		getaddednodeinfo dns ( "node" )
		getconnectioncount
		getnettotals
		getnetworkinfo
		getpeerinfo
		listbanned
		ping
		setban "ip(/netmask)" "add|remove" (bantime) (absolute)

		== Rawtransactions ==
		createrawtransaction [{"txid":"id","vout":n},...] {"address":amount,...}
		decoderawtransaction "hexstring"
		decodescript "hex"
		fundrawtransaction "hexstring"
		getrawtransaction "txid" ( verbose )
		sendrawtransaction "hexstring" ( allowhighfees )
		signrawtransaction "hexstring" ( [{"txid":"id","vout":n,"scriptPubKey":"hex","redeemScript":"hex"},...] ["privatekey1",...] sighashtype )

		== Util ==
		createmultisig nrequired ["key",...]
		estimatefee nblocks
		estimatepriority nblocks
		validateaddress "zcashaddress"
		verifymessage "zcashaddress" "signature" "message"
		z_validateaddress "zaddr"

		== Wallet ==
		addmultisigaddress nrequired ["key",...] ( "account" )
		backupwallet "destination"
		dumpprivkey "zcashaddress"
		dumpwallet "filename"
		encryptwallet "passphrase"
		getaccount "zcashaddress"
		getaccountaddress "account"
		getaddressesbyaccount "account"
		getbalance ( "account" minconf includeWatchonly )
		getnewaddress ( "account" )
		getrawchangeaddress
		getreceivedbyaccount "account" ( minconf )
		getreceivedbyaddress "zcashaddress" ( minconf )
		gettransaction "txid" ( includeWatchonly )
		getunconfirmedbalance
		getwalletinfo
		importaddress "address" ( "label" rescan )
		importprivkey "zcashprivkey" ( "label" rescan )
		importwallet "filename"
		keypoolrefill ( newsize )
		listaccounts ( minconf includeWatchonly)
		listaddressgroupings
		listlockunspent
		listreceivedbyaccount ( minconf includeempty includeWatchonly)
		listreceivedbyaddress ( minconf includeempty includeWatchonly)
		listsinceblock ( "blockhash" target-confirmations includeWatchonly)
		listtransactions ( "account" count from includeWatchonly)
		listunspent ( minconf maxconf  ["address",...] )
		lockunspent unlock [{"txid":"txid","vout":n},...]
		move "fromaccount" "toaccount" amount ( minconf "comment" )
		sendfrom "fromaccount" "tozcashaddress" amount ( minconf "comment" "comment-to" )
		sendmany "fromaccount" {"address":amount,...} ( minconf "comment" ["address",...] )
		sendtoaddress "zcashaddress" amount ( "comment" "comment-to" subtractfeefromamount )
		setaccount "zcashaddress" "account"
		settxfee amount
		signmessage "zcashaddress" "message"
		z_exportkey "zaddr"
		z_exportviewingkey "zaddr"
		z_exportwallet "filename"
		z_getbalance "address" ( minconf )
		z_getnewaddress
		z_getoperationresult (["operationid", ... ]) 
		z_getoperationstatus (["operationid", ... ]) 
		z_gettotalbalance ( minconf includeWatchonly )
		z_importkey "zkey" ( rescan startHeight )
		z_importviewingkey "vkey" ( rescan startHeight )
		z_importwallet "filename"
		z_listaddresses ( includeWatchonly )
		z_listoperationids
		z_listreceivedbyaddress "address" ( minconf )
		z_sendmany "fromaddress" [{"address":... ,"amount":...},...] ( minconf ) ( fee )
		z_shieldcoinbase "fromaddress" "tozaddress" ( fee ) ( limit )
		zcbenchmark benchmarktype samplecount
		zcrawjoinsplit rawtx inputs outputs vpub_old vpub_new
		zcrawkeygen
		zcrawreceive zcsecretkey encryptednote
		zcsamplejoinsplit

