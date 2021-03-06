---
title: 以太坊POS-Casper技术研究报告
comments: false
date: 2018-03-21 10:20:37
categories: 矿池
tags:
- lucas556
- 以太坊
- POS
- Casper
img:
---
## 定义
### PoW：Proof of Work，工作量证明
依赖机器进行数学运算来获取记账权,资源消耗相比其他共识机制高、可监管性弱,同时每次达成共识需要全网共同参与运算,性能效率比较低,容错性方面允许全网50%节点出错。

### PoS：Proof of Stake，权益(股权)证明
主要思想是节点记账权的获得难度与节点持有的权益成反比,相对于PoW,一定程度减少了数学运算带来的资源消耗,性能也得到了相应的提升,但依然是基于哈希运算竞争获取记账权的方式,可监管性弱.该共识机制容错性和PoW相同.它是Pow的一种升级共识机制,根据每个节点所占代币的比例和时间,等比例的降低挖矿难度,从而加快找随机数的速度。

## 以太坊Casper
### 优势
- 不需要消耗大量的资源就可以保证区块链的安全性.参与 PoS 的代币数量相当于 PoW
算力，达到了同样的出块效果却不用消耗真实的电力,根据比特币能源消耗指数,当前全球用于比特币挖矿所产生的年用电量预计为30.14太瓦时(TWh),比159个国家年均用电量还高。
- 因为不再需要大量资源，就不需要最大化的发行新币来激励参与者,从而降低网络的压力.理论上甚至有可能出现负净发行,其中一部分交易费用被消耗掉，因此供应随着时间的推移而下降。
- 权益证明为更广泛的使用博弈论机制设计的技术敞开了大门,以便更好地阻止中心化垄断的形成,以及(如果确实形成了中心化垄断)伤害网络的行为。
- 降低中心化风险之后，经济规模的扩大便不是问题.在工作量证明机制中，当资本多到一定程度，便可以投入更大规模的生产设备，拉大和其他人的差距（投入一千万的成本所获得的收益不只是投入一百万的十倍）.而在权益证明机制中，一千万的投资只能保证获取一百万投资的十倍的收益。
- 相对于工作量证明，能够使用经济处罚来大大提高各种形式的 51％ 攻击的成本.正如 Vlad Zamfir 所说的，“只要参与了 51％ 攻击，ASIC 矿场就会被烧毁”。

### Casper与Pos的区别
Casper实施了一个进程，使得它可以惩罚所有的恶意因素.这就是权益证明在Casper下是如何工作的：
- 验证者押下一定比例的他们拥有的以太币作为保证金
- 验证者开始验证区块.也就是说，当验证者发现一个区块并认为这个区块可以被加到链上的时候，验证者将以通过押下赌注来验证它
- 如果该区块被加到链上，然后验证者们将得到一个跟他们的赌注成比例的奖励
- 但是，如果一个验证者采用一种恶意的方式行动、试图做“无利害关系”的事，他们将立即遭到惩罚，他们所有的权益都会被砍掉

正如以上，Casper被设计成可以在一个无需信任的系统上工作，并且是更加拜占庭容错的。
任何人，如果以一种恶意的，或者说有拜占庭叛徒式的方式行动，就会立即受到惩罚、失去他们的保证金.这就是Casper不同于其他权益证明协议的地方.恶意元素会失去一些东西，所以，“无利害关系”是不可能的。
这不是唯一一个Casper可以惩罚验证者的地方，正如Hudson James以及Joris Bontje在“StackExchange”的回答中标注的那样，Casper设计了苛刻的激励来保证网络的安全，包括惩罚离线的矿工，不管它是有意还是无意的。这意味着验证者将不得不变得对他们的节点正常运行时间小心翼翼.粗心或者懒惰都将导致他们失去自己保证金.这一属性减少了对交易和整体利用率的审查.围绕着所有这些，这种“惩罚”属性同样给予了Casper相对标准工作量证明协议的明显优势。
![](/images/pos_createblock.png)
如上图所示,在一个工作量证明协议中，一个矿工会在蓝色链上还是红色链上挖矿并不重要.因为诚实的和恶意的矿工都将花费同等数量的资源。然而，在Casper中，如果一个诚实的验证者在蓝色链上挖矿，他们将得到与他们的赌注成比例的奖励，但一个恶意的矿工将因为下注在红色链条上而失去他们的保证金。

### Casper版本
Casper不是一个具体的项目.它是两个研究项目的融合，这两个在最近一直由以太坊开发者团队承担：
- Casper the Friendly Finality Gadget（FFG）
- Casper the Friendly GHOST: Correct-by-Construction（CBC）

### Casper FFG
Casper FFG也就是Vitalik版Casper，是一个混合PoW/PoS共识机制,它是正准备进行初步应用的版本，也是被精心设计好来缓冲权益证明的转变过程的.设计的方式是，一个权益证明协议被叠加在正常的以太坊版工作量证明协议上.虽然区块仍将通过工作量证明来挖出，每50个区块就将有一个权益证明检查点，也就是网络中验证者评估确定性（Finality）的地方。

### 确定性（Finality）
确定性（Finality），从一个非常宽松的意义上来说，意味着一旦一个特定的操作完成，它将永远被蚀刻在历史上，没有任何东西可以逆转这个操作.在处理金融事务的领域，这是非常重要的.例如,A在一个公司里拥有特定数目的一种资产.就算公司的某些进程中出了一点小故障，A也不应该需要恢复对该资产的所有权.
有人说，工作量证明是唯一一种在区块链上实现确定性的方式.但是，这并不是必然正确的.真相远比这个要复杂很多。
正如Vitalic Buterin提出的:世界上没有一个系统可以提供100%的确定性.黑进一个系统，或者物理上破解一份注册表并篡改数字以改变一个人的资产负债表，都是有可能的.这也是中心化机构的一个大问题.但是，分布式的系统也会面临同样的问题。
实际上，比特币是工作量证明机制的典范，至少三次曾经面临确定性问题.在一个例子中，链必须分叉，因为一个Bug存在于软件的一个版本中但在其它版本中并不存在.这在社区中导致了分裂，一部分人拒接接受被另一部分人所接受的链.这次分裂在6个小时中被解决。
所以，问题在于，Casper FFG如何能够提供确定性？根据Vitalik的说法 ，因为下面三个理由，Casper保证可以提供比工作量证明更强的确定性:
- 完全经济确定性.三分之二的验证者会下最大几率的赌注使区块达到最终一致.因此，对他们来说，串谋以及攻击网络的激励是非常小的，因为，如果他们这样做的话，他们将危及自己的保证金.Vlad Zamfir更好地解释了这一点，他说：“设想一种版本的工作量证明，如果你参与一场51%供给的话，你的矿机会烧毁.”
- 假设整个网络由三个人组成：Alice，Bob，和Charlie.假设Alice和Bob将他们的保证金放在一种结论上，同时，Bob和Charlie把他们的保证金放在一个与之对立的结论上。不管发生什么事，Alice或者Charlie其中一人肯定会损失一大笔钱.所以，正如你可以看到的，验证者没有动机去串谋或用恶意的方式行动，因为他们总会失去一大笔钱
- 然而，如果双重确定性（Double Finality）发生的话，还有一种意外事故处理方案.如果双重确定性发生的话，用户可以选择他们想到哪条链上去。不管哪条链，得到多数票的就成为主链.基本上，在Casper上，双重确认会导致硬分叉而不是回滚。

### Casper CBC
Casper CBC也就是Vlad版Casper使用建构修正（correct-by-construction，CBC）协议。
下面是一个普通的协议设计：
- 正式指定协议
- 定义该协议必须满足的属性
- 证明该协议可以满足给定的属性

而CBC协议的设计是：
- 正式地但只是部分地指定协议
- 定义该协议必须指定的属性
- 从满足所有它被规定去指明的属性中推导出该协议
- 提出一个合理估计的错误的例外情况
- 列出所有在未来可能发生的错误

所以，Casper CBC要做的事情，就是不断进行微调、让这个只是部分建构好的协议更加完美，直到它变成完全版。目前来说，以太坊开发者团队一直在努力地开发这两个Casper项目.很明显，这不会是最终版本，但不管这最终版本是什么，它肯定会受到Vlad的和Vitalik的Casper的深刻影响.正如刚刚提到过的，Vitalik的Casper将被初步运行以缓冲从PoW到PoS的转变.而Vlad的Casper，通过使用一个“理想对手”推导出一个安全性论证

## Casper知识
### Casper:去中心化
![](/images/pos_decentralization.png)
正如在上图看到的，工作量证明协议不再是真正对去中心化友好的了。
大部分哈希算力集中在几个特定的矿池，而这意味着无论发生什么事，他们总会比其他人有更大的机会挖到区块、获得奖励。
因为他们可以获得更多钱，他们可以买得起更好更快的ASIC设备.这基本上意味着，无论发生什么，大矿池将总是比个人和小矿池拥有优势.换句话来说，富有的将变得更富有。
权益证明通过让挖矿完全虚拟化使得这一切都无关紧要.然而，这不是权益证明减缓中心化效果的唯一途径.为了理解这个，首先需要弄懂“经济规模”意味着什么。
跟大企业很像，大矿池可以通过下列途径减少他们投入资源的成本：
- 在大规模运营中分摊固定成本
- 作为一个更大的经营主体，拥有议价能力

这意味着，一个大而有影响力的矿池可以一美元又一美元地比其他矿池生产更多哈希算力，即便他们投入了等量的资金。
这一问题在权益证明中完全被消解了，因为一个简单的原因.在权益证明中你投资一份保证金.你不能轻易地结成矿池然后让你的保证金一美元又一美元地升值.在一天结束的时候，一美元还是一美元.规模经济在这里不起作用。

### Casper:能源效率
上面讲过,当前全球用于比特币挖矿所产生的年用电量预计为30.14太瓦时(TWh),比159个国家年均用电量还高.最糟糕的部分是，能源浪费是为了能源浪费.尤其是比特币，其对能源的欲望是非常贪婪的。

所以，很明显，比特币使用太多电力了，太多钱被花在资源上了.但是，外部成本是大量的电力消耗对环境产生的影响必定是巨大的。

虽然毫无疑问地比特币和工作量证明已经造成了大量积极的社会影响，我们至少应该看看权益证明系统可以做到怎样的规模，以及，它是否可以在不消耗那么多电力的情况下工作。

### Casper:经济安全
Casper的最大优势是它的经济安全性.假设你是一个验证者，并且你将你自己的钱作为保证金存入网络.以最大化网络利益的方式行事也是你自己的利益.知道了如果有恶意的行为你的保证金中的一大部分将被罚没，你为什么还会那样做呢？

当你有很多钱锁定在里面的时候，你为什么还要攻击网络、损害币值呢？同样地，“惩罚效应”消除了Vitalik定义的“驻守重生点攻击（spawn camping attack）”的可能性。

在权益证明中，重生点攻击可以被防止，因为一个简单的事实：一次攻击将导致惩罚，没收已投入的保证金.如果你没有投入任何保证金，就不能成为Casper验证人。

### 以太坊Casper的未来
Vitalik
Buterin说不仅Casper准备好接受测试了，它还可以在客户端更新代码时提供一个安全提升.虽然看起来并不像它已经准备好用来广泛普及了，但第一个Casper测试网发布的日子看起来是越来越近了。

权益证明是否将要实施并不是一个问题，问题是什么时候能够实施.以太坊“宁静”（Serenity）应该是一个权益证明网络.此前我们并不是从未见过PoS的实行，Peercoin非常成功地运用了它.但是，我们从还从来没有看过在这个层面采用该协议。也许，如果Casper被成功运行的话，另一个加密货币会跟风并且做出转换.不管事情会变成什么样，Casper带来了非常多迷人的可能性。

## Casper技术
### 投注共识
Casper是一种以太坊下一代的共识机制,属于PoS Casper的共识是按块达成的而不是像PoS那样按链达成的。

Casper向公开经济学共识这个领域引入了一个根本上全新的理念作为自己的基础：投注共识.投注共识的核心思想很简单：为验证人(validator)提供与协议对赌哪个块会被最终确定的机会.在这里对某个区块X的投注就是一笔交易，在所有区块X被处理了的世界中都会带给验证人Y个币的奖励（奖励是凭空“印”出来的，因而是“与协议”对赌），而在所有区块X没有被处理的世界中会对验证人收取Z个币的罚款（罚金被销毁）。

为了防止验证人在不同的世界中提供不同的投注,还有一个简单严格的条款:如果有两次投注序号一样,或者说提交了一个无法让Casper合约处理的投注,会失去所有保证金..从这一点可以看出,Casper与传统的PoS不同的是Casper有惩罚机制,这样非法节点通过恶意攻击网络不仅得不到交易费,而且还面临着保证金被没收的风险。

只有在相信区块X在人们关心的那个世界中被处理的可能性足够大时，这才是值得去做的交易，验证人才会愿意投注.然而，接下来就是经济上递归的有趣部分：人们关心的那个世界，也就是用户的客户端在用户想要知道他们的账户余额或是合约的状态时所展现的那个状态，本身就是根据人们对哪个区块投注最多推导出来的.因此,每一个验证人都具有根据他们所预期的其他人的投注情况进行投注的动机，驱使这个过程走向收敛。

一个有帮助的类比是工作量证明共识 - 它本身看起来非常独特，实际上可以成为投注共识的一个特别子模型.理由如下：当你基于一个块挖矿时，你是在花费每秒E的电力成本换取每秒p的出块概率，并且在所有包含你的出块的分叉中获得R个币，在其它分叉中分文不得。
![](/images/pos_createblock2.png)
因此，每一秒钟，在你挖矿的链上你可以获得p * (R-E) 的期望收益，在其它链上遭受E的损失；因此你的挖矿选择可以理解为下注赌你所在的链有 E:p * (R-E) 的相对概率(odds)胜出。
比如，假设p等于百万分之一，R是25个币约等于10000美元，而E是0.007美元，则在胜出链上每秒钟的期望收益是0.000001 * 10000 - 0.007 = 0.003，在失败链上的损失是0.007的电力成本，因此这是在赌自己挖矿的链有7:3的相对概率胜出.注意工作量证明满足上面所说的经济上递归的要求：用户的客户端通过处理拥有最大工作量的那条链来计算其账户余额。

投注共识可以看作是包含了以特定方式看待的工作量证明的一个框架，也适合为其他多种类型的共识协议提供能促进收敛的经济博弈.例如传统的拜占庭容错共识协议中，通常在对某个结果进行最后"commit"之前还有"pre-votes"和"pre-commits"的概念；在投注共识的模型下，我们可以把每一阶段都变成投注，这样后面阶段的参与者就有更大把握相信前面阶段的参与者“真的是这个意思”。

投注共识还可以用于激励链外人类共识(out-of-band human consensus)的正确行为，为了克服类似51%攻击的极端情况有需要的话.假设有人购买了一条PoS链上超过一半的币，并进行攻击，那么社区只需要协商出一个让客户端忽略攻击者分叉的补丁，就能自动让攻击者失去所有的币.一个极有野心的目标是让在线节点可以自动的产生这种分叉决定,如果能成功实现，传统容错研究中的一个被低估但却重要的结论,在强同步假设下，即使几乎所有节点都在尝试攻击系统，剩下的节点依然可以达成共识,也可以被纳入投注共识的框架中。

在投注共识的情境中，不同的共识协议只在一件事情上有区别：谁，可以以什么赔率，投多少注？工作量证明只提供了一种赌局：投注胜出链有E:p * R-E的相对概率包含你自己出的块.在广义的投注共识中，依据一种被称为评分规则(scoring rule)的机制我们本质上可以提供无限多种赌局：在1:1上压极小的一注，在1.000001:1上也压极小注，在1.000002:1上也压极小注，如此继续。

![](/images/pos_bet.png)
参与者依然可以选择在每一个概率等级上的这些极小边际投注的确切大小，但大体上这个技术让我们能打探出验证人认为某个块会被确认的相当精确的概率读数.如果验证人认为一个块有90%的概率会被确认，那么他们就会接受所有相对概率低于9:1的赌局，拒绝相对概率高于9:1的赌局，而协议就能基于这一点准确得出这个块会被确认的概率是90%的“看法”.事实上，显示原理(revelationprinciple)告诉我们可以要求验证人直接给出他们对某个块被确认概率的“看法”的签名消息，让协议代表验证人计算投注。

![](/pos_voting.png)
从上面的微积分图中，可以得到在每一个概率等级上计算总奖励和总惩罚的简单函数，计算结果在数学上等价于根据验证人信心在所有概率等级上形成的投注的无限集合的总和.一个简单的例子是s(p)= p(1-p)和f(p) = (p/(1-p))^2/2，这里s计算如果你投注的事件发生能获得的奖励，f 计算如果没有发生你受到的惩罚。
投注共识的广义形式有一个重要优点.在工作量证明中，给定区块背后的“经济权重”仅仅随着时间线性增加：如果一个块有6个确认，那么要撤销它只需要花费矿工大约6倍于出块奖励（在均衡态下）的成本，而如果一个块有600个确认，那么撤销它的成本就是出块奖励的600倍.在广义的投注共识中，验证人在一个块上投入的经济权重可以指数级增加：如果其他大多数验证人愿意以10:1下注，你可能会想冒险以20:1下注；而一旦几乎所有人都增加到20:1，你可能会跳到40:1或者更高.因此，一个块很可能在几分钟之内，取决于验证人有多少勇气（以及协议提供的激励大小），就达到一种“准最终确定”的状态，这种状态下验证人的所有保证金都成为了支持这个块的投注。
