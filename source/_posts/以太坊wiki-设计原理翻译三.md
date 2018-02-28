---
title: 以太坊wiki-设计原理翻译三
comments: false
date: 2018-02-24 10:39:17
categories: 区块链
tags: 区块链 以太坊
img:
---
[以太坊wiki-设计原理翻译二](https://xingyunbite.github.io/2018/02/09/%E4%BB%A5%E5%A4%AA%E5%9D%8Awiki-%E8%AE%BE%E8%AE%A1%E5%8E%9F%E7%90%86%E7%BF%BB%E8%AF%91%E4%BA%8C/)

## 树的使用方式

警告：本节内容假设你已经了解了布隆过滤器是如何工作的。详情可见:[http://en.wikipedia.org/wiki/Bloom_filter](http://en.wikipedia.org/wiki/Bloom_filter)

以太坊区块链中的每个区块都包含了 3 个指针指向 3 个 树：状态树（the state trie），代表连接区块之后的全部状态，交易树，代表区块中索引关联的所有交易（即 `key 0`:第一笔要执行的交易，`key 1`:第二笔要执行的交易，等等），以及收据树，代表了符合每笔交易的收据（receipts）。对应于一笔交易的收据是一个经过 `RLP` 编码的数据结构：

	[ medstate, gas_used, logbloom, logs ]

对应的：
* `medstate` 是处理交易之后的状态树的根。
* `gas_used` 是处理该笔交易之后消耗的燃料数量。
* `logs` 是交易执行过程中由 `LOG0 ... LOG4` 操作码生成的 `[address, [topic1, topic2...], data]`形式的项目列表（包括主调用和次调用）。`address` 是产生日志的合约的地址，`topics` 最多为 4 个 32 字节值，`data` 是任意大小的字节数组。
* `logbloom` 是交易中所有的日志的 `topics` 和地址组成的布隆过滤器。
	区块头中也有一个布隆（bloom），它是区块中所有交易的布隆的“或”结果。这个结构的目的是使以太坊的轻客户端尽可能的友好。更多以太坊轻客户端和它们的使用实例，见:[light client page (principles section)](https://github.com/ethereum/wiki/wiki/Light-client-protocol#principles)。

## 叔块激励

“贪婪的最重观察子树（Greedy Heaviest Observed Subtree）”（GHOST）协议是 Yonatan Sompolinsky 和 Aviv Zohar 于 2013 年 12 月[首次](http://eprint.iacr.org/2013/881.pdf)推出的一项创新，也是第一次有人认真尝试解决“阻塞越来越快”的问题。`GHOST` 背后的动机是，目前区块链通过最快确认次数来确认区块的方案，由于过高的陈旧率而降低了安全性 - 因为区块在网络中传播需要一定的时间，如果矿工 A 挖出一个块，然后在矿工 A 的块传播到 B 之前矿工 B 恰巧挖出另一个块 ，矿工 B 的块将会浪费掉（“陈旧”），并且对于网络安全没有任何帮助。此外，还有一个集中化问题：如果矿工 A 是一个拥有3 0％ 算力的矿池，而 B 拥有 10％ 的算力，那么 A 将有 70％ 的时间产生陈旧块的风险（因为另外 30％ 的时间 A 产生了最后一个块，因此将立即获取挖掘数据），而 B 将有 90％ 的时间产生陈旧块的风险。因此，如果区块间隔足够短而导致陈旧率较高，那么凭借其算力，明显地，A 将更高效。将这两种效应结合起来，快速生成区块的区块链很可能导致一个具有足够算力的矿池对挖矿产生垄断。

正如 Sompolinsky 和 Zohar 所描述的那样，`GHOST` 通过在计算哪个链是“最长”时包含陈旧块来解决网络安全损失的第一个问题; 也就是说，在计算最长链的工作量证明中，除了该区块的直系祖先区块，还包含了该区块之前被废弃的叔块。

为了解决可能的中心化问题，我们采用了一个不同的策略：我们对叔块进行奖励：一个叔块会获得它的基本奖励的 7/8(87.5%)，而该叔块的下一个区块会获得基本奖励的 1/32(3.125%)。当然交易费不会作为奖励。

在以太坊中，一个陈旧的块只能被当做一个叔块，然后从它的兄弟区块往下最多被直系区块包含 7 代，除此之外叔块和其他任何区块都没有关系。这么做的原因如下：首先，不受限制的 `GHOST` 会在计算哪个叔块对于当前的区块是合法的时候出现大量的困难。其次，在以太坊中使用无限制的叔块激励，会导致矿工对于是否在主链挖矿不再关心，而这可能导致主链被公共攻击。最后，计算表明，限制到七个级别时利大于弊。

* 这里有一个可用的衡量集中化风险的仿真器:[https://github.com/ethereum/economic-modeling/blob/master/ghost.py](https://github.com/ethereum/economic-modeling/blob/master/ghost.py)
* 一个更高级的讨论:[https://blog.ethereum.org/2014/07/11/toward-a-12-second-block-time/](https://blog.ethereum.org/2014/07/11/toward-a-12-second-block-time/)

我们的区块时间算法中的设计决定包含：

* **12 秒出块时间**：选择 12 秒作为尽可能快的时间，但同时比网络等待时间长得多。2013年，Decker和Wattenhofer在苏黎世的[一篇文章](http://www.tik.ee.ethz.ch/file/49318d3f56c1d525aabf7fda78b23fc0/P2P2013_041.pdf)测量了比特币的网络延迟，并确定了 12.6 秒是一个新块传播到95％的节点所花费的时间。然而，论文还指出，大部分的传播时间与块的大小成正比，因此越快的货币传播时间越少（in a faster currency we can expect the propagation time to be drastically reduced）。传播间隔的恒定部分约为 2 秒，但是，为了安全起见，我们假设块在我们的分析中需要 12 秒才能传播。
* **7 个区块祖先限制**：这是一个设计目标的一部分，希望在块数很少的情况下很快就可以使块的历史被“遗忘”，并且已经证明 7 块已经提供了大部分预期的效果。
* **一个区块后代限制**（如，`c(c(p(p(p(head)))))`，其中 c = child，p = parent，是不合法的）：这是简洁的设计目标的一部分，上面的模拟器显示它不会带来很大的集中化风险。
* **叔块验证要求**：叔块必须包含一个有效的头部，而不是区块。这是为了简单起见，并将区块链模型保持为线性数据结构（而不是 Sompolinsky 和 Zohar 的新模型中的块 `DAG`）。 要求叔块的块合法也是一个有效的方法。

## 难度调整算法

以太坊目前的难度调整规则如下：

```
diff(genesis) = 2^32

diff(block) = diff.block.parent + floor(diff.block.parent / 1024) *
	1 if block.timestamp - block.parent.timestamp < 9 else
	-1 if block.timestamp - block.parent.timestamp >= 9
```

难度调整规则的设计目标是：

* **快速更新**：区块之间的时间的调整要快。

* **稳定**：如果算力不变，难度不应该过大。

* **简单**：算法实现要简单。

* **占用内存低**：算法不应依赖太多历史区块，并尽可能少地使用内存。假设最后十个块，加上最后十个块的块头中的所有内存变量，这就足够算法运行了。

* **非开发性**：该算法不应该过度鼓励矿工修改时间戳，或者矿池频繁地增加和移除算力，以试图最大化他们的收入。

我们已经确定，我们目前的算法在稳定性和不可利用性方面非常不理想，然后我们打算至少将时间戳相对于父母和祖父母进行切换，这样当矿工挖矿时，他们只会修改时间戳。另一个更强大的仿真器公式位于[`https://github.com/ethereum/economic-modeling/blob/master/diffadjust/blkdiff.p`](https://github.com/ethereum/economic-modeling/blob/master/diffadjust/blkdiff.py)（仿真器使用比特币的挖矿能力，但只使用当天的均值;它在一天的时间内，在某个点模拟 95% 的崩溃）。
