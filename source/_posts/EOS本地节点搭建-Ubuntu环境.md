---
title: EOS本地节点搭建-Ubuntu环境
comments: false
date: 2018-03-25 23:12:19
categories: 区块链
tags:
- LeonBCK
- EOS
img:
---
本文编译环境为Ubuntu 16.04 LTS（腾讯云）。
## 安装local testnet
安装的过程主要参照[官方文档](https://github.com/EOSIO/eos#autoubuntulocal)
```
git clone https://github.com/eosio/eos --recursive
cd eos
./eosio_build.sh
```
这个过程持续大约十分钟，当屏幕提示"EOSIO has been successfully built."时，安装完成。
![image](/images/eos_build.png)

## 单节点testnet配置、运行
按照文档进行[单节点测试运行](https://github.com/EOSIO/eos#creating-and-launching-a-single-node-testnet)配置。
在进入build/programs/nodeos目录下，运行nodeos,运行结果如下：
![image](/images/nodeos_try1.png)
并没有出现文档中所说的报错退出，等待一分钟后，强制退出(Ctrl + C)。查看nodeos目录，并没有创建data-dir，也没有创建配置文件config.ini。
![image](/images/nodeos_ls1.png)
查看Issues，发现有人遇到了一样的问题，上面有人给出的解决方案是使用config-dir命令自己创建，使用如下命令:
```
./nodeos --config-dir data-dir/
```
查看nodeos目录，发现多出了data-dir文件夹，进入data-dir，可以看到配置文件config.ini。按照文档提示，对配置文件进行修改：
```
# Load the testnet genesis state, which creates some initial block producers with the default key
genesis-json = "genesis.json"
# Enable production on a stale chain, since a single-node test chain is pretty much always stale
enable-stale-production = true
# Enable block production with the testnet producers
producer-name = inita
producer-name = initb
producer-name = initc
producer-name = initd
producer-name = inite
producer-name = initf
producer-name = initg
producer-name = inith
producer-name = initi
producer-name = initj
producer-name = initk
producer-name = initl
producer-name = initm
producer-name = initn
producer-name = inito
producer-name = initp
producer-name = initq
producer-name = initr
producer-name = inits
producer-name = initt
producer-name = initu
# Load the block producer plugin, so you can produce blocks
plugin = eosio::producer_plugin
# Wallet plugin
plugin = eosio::wallet_api_plugin
# As well as API and HTTP plugins
plugin = eosio::chain_api_plugin
plugin = eosio::http_plugin
```
在配置genesis-json的时候，我们发现在eos目录下存在一个genesis.json文件，将之拷贝到data-dir目录下，原有配置不用修改。
再次运行nodeos，发现还是没有办法正常出块，而文档中也没有给出相应的说明。没办法，只能回过头来再次检查之前的配置，而最终解决这个问题也存在一定的偶然性，当我新开一个窗口，同时运行nodeos的时候，给出了如下提示：
![image](/images/eos_node00.png)
我们发现nodeos读取的是eos/build/etc/eosio/node_00目录下的genesis.json，而当我们进入到node_00目录下的时候，发现该目录下也生成了一个config.ini文件，其生成时间与执行nodeos -- config-dir的时间相同，这说明nodeos默认读取该目录下的配置文件，而我们刚才的配置无效，于是将data-dir目录下的config.ini覆盖掉node_00目录下的config.ini。
再次启动nodeos，发现正常启动。
![image](/images/nodeos_not_my_turn.png)
等待一定时间后，我们发现依然没有正常的出块，同时给出提示信息"Not producing block because it isn't my turn, its eosio"
我们知道eos使用的是DPOS共识模式，而在config.ini中我们配置了21个producer,那么是否是因为eosio是eos在local testnet下的默认节点，而配置文件中没有对应的producer导致的呢？ 我在producer-name = initu 下增加了 producer-name = eosio 再次运行nodeos，发现nodeos运行正常，成功出块，只是块中没有交易。
![image](/images/nodeos_produce_blocks.png)
![image](/images/cleos_get_info.png)
至此，我便完成了eos本地节点的搭建，由于eos代码更新较快，官方文档没有同步更新，踩了许多了坑，特此记录。
