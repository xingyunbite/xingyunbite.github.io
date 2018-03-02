---
title: zcash矿池之Z-NOMP(Node Open Mining Portal)
comments: false
date: 2018-03-02 13:00:28
categories: 矿池
tags: 矿池 zcash
img:
---

**Zcash作为采用零知识证明（zk-SNARKS）实现完全隐私保护的加密货币,在技术地位上成为竞争币一哥指日可待.**

z-classic在github发布的矿池Z-NOMP，是基于Node Open Mining Portal的Equihash矿池，可以部署的币种有zen，zcl和zec等使用Equihash算法的矿池。

**警告:请勿直接用于生产环境使用该软件,任何内容的修改都会对矿池造成损失.**

首先，部署环境，我们使用的是ubuntu server 16.04 x64的系统。
  
1.安装z-nomp依赖：
  
``` shell
    sudo apt-get install update
    sudo apt-get install build-essential libsodium-dev npm git
    sudo npm install n -g
    sudo n 8.9.4
```

2.下载和安装z-nomp：

``` shell
	git clone https://github.com/joshuayabut/node-open-mining-portal.git /pool
	cd /pool
	npm config set registry https://registry.npm.taobao.org  #使用淘宝npm镜像
	npm update
	sudo npm install   #就算root账户也一定要加sudo
```

3.矿池配置文件：
修改示例文件config_example.json。

``` shell
	mv config_example.json config.json
	vim config.json
```

下面提供一些主要配置说明：
``` json
	"redis": {
		"host": "127.0.0.1",   #redis地址
		"port": 6379,          #redis端口
		"password": ""      #redis密码
	}
    },
	    
	"website": {
		"enabled": true,
		"host": "0.0.0.0",                 #website地址
		"port": 8080,                      #website端口
		"stratumHost": "cryppit.com",      #挖矿域名    
		"stats": {
			"updateInterval": 30,
			"historicalRetention": 14400,
			"hashrateWindow": 300
		},
							           
		"tlsOptions" : {
			"enabled": false,          #ssl挖矿
			"cert": "",                #ssl公钥地址
			"key": ""                  #ssl私钥地址
		}
	},
```

4.数字货币配置：

进入pool_config，重新命名zcash_example.json为zcash.json并修改示例内容.

``` shell
	cd pool_config
	mv zcash_example.json zcash.json
	vim zcash.json
```

主要配置文件说明：

``` json
	"enabled": true,         #设置打开coin
	"coin": "zcash.json",    #coin配置文件
	"address": "",           #coinbase地址，即挖到块后保存的地址，且该地址一定要在节点中 
	"zAddress": "",          #Z地址
	"tAddress": "",          #T地址

	"rewardRecipients": {
		"": 0.2,             #手续费和接收手续费地址
	},

	"paymentProcessing": {
		"minConf": 10,
		"enabled": false,             #支付进程
		"paymentMode": "prop",        #挖矿模式
		"_comment_paymentMode":"prop, pplnt",
		"paymentInterval": 20, 
		"minimumPayment": 0.1,         #最低支付额
		"maxBlocksPerPayment": 1,     
		"daemon": {                    #节点信息 这里为支付节点
			"host": "127.0.0.1",       
			"port": 19332,
			"user": "testuser",
			"password": "testpass"
		}
	},
	
	"ports": {                 #支付端口和难度																													         
		"3032": {
		"diff": 0.05,
		"tls": false,
		"varDiff": {
			"minDiff": 0.04,
			"maxDiff": 16,
			"targetTime": 15,
			"retargetTime": 60,
			"variancePercent": 30
		}
	}
	},
	
	"daemons": [                #节点  这里为挖矿节点 可与支付节点一致
		{
			"host": "127.0.0.1",
			"port": 18232,
			"user": "rpcuser",
			"password": "rpcpassword"
		}
    ],
```

5.coin配置：

进入coins目录可以看到开发者提供了zclassic和zcash等coin的配置文件，同时zcash_testnet.json为zcash的testnet方便测试，使用testnet进行测试时请保证节点一定在testnet模式。如果需要加增加coin请参考开发者提供的其他coin信息进行修改，同时在pool_config里增加池信息，以后我们在进行陆续说明。

ok，现在已经配置完成，我们返回/pool目录 ，使用npm start 来启动z-nomp 矿池，就可以把矿机接入来进行挖矿了。
