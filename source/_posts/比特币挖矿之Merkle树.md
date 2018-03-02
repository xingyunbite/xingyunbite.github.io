---
title: 比特币挖矿之Merkle树
comments: false
date: 2018-03-02 16:08:16
categories: 矿池 
tags: 比特币 矿池
img:
---

## Merkle树概念
Merkle树，通常也被称为Hash Tree，顾名思义，就是存储hash值的一棵树。Merkle树的叶子是数据块的Hash值，非叶子节点是其对应子节点串联字符串的Hash值。
在最底层，和哈希列表一样，我们把数据分成小的数据块，有相应的哈希与之对应。但是往上走，并不是直接去运算根哈希，而是把相邻的两个哈希合成一个字符串，然后运算这个字符串的哈希。如果最底层的哈希总数是单数，那么便将最后一个叶子节点复制一份，以构成偶数个叶子节点，这种偶数个叶子节点的树也被称为平衡树。

## Merkle树特点
1、Merkle树是一种树，大多数是二叉树，也可以是多叉树，当然也都具有树结构的所有特点；
2、Merkle树的叶子节点的值是数据集合的单元数据或者单元数据的HASH。
3、非叶子节点的值是根据它下面所有的叶子节点值，按照Hash算法计算得到的。

## 比特币与Merkle树
根据之前文章对比特币区块头的分析，可以知道比特币区块头中存在一个hashMerkleRoot字段，这儿的hashMerkleRoot便是区块中所有交易构建的Merkle树的树根value，比特币使用Merkle树来归纳一个区块中的所有交易，当某个节点试图修改某个区块中的交易时，交易的Hash值改变，这种改变层层传递，会导致MerkleRoot值改变，进而导致区块头的Hash值改变，这将导致软分叉，要想使这种改变被全网接受，只能通过大算力，使包含该区块的链成为最长链，这种修改的成本无疑是巨大的。由于Merkle树结构的特殊性，通过Merkle树，轻量级节点SPV可以很容易验证交易的存在，下面进行具体介绍。
![](/images/miner.png)
SPV节点不保存所有交易，也不会下载整个区块，仅保存区块头，当需要验证交易时，假设要验证区块图中交易tx3,SPV节点会通过向相邻节点索要Merkle树分支（H4->H12->H5656->root）来确认交易的存在性和正确性。得到了分支数据，SPV节点可以很容易计算出H34->H1234->root,通过这种计算得到的root与本地存储的区块头中的hashMerkleRoot字段比较，可以很容易得到验证结果。

比特币[最初版本](https://github.com/trottier/original-bitcoin/blob/92ee8d9a994391d148733da77e2bbc2f4acc43cd/src/main.h#L868)构建Merkle树的实现代码如下：
```
uint256 BuildMerkleTree() const
{
    vMerkleTree.clear(); // 清空Merkle树原有内容 vMerkleTree是一个vector<uint256> 结构
    foreach(const CTransaction& tx, vtx) // 遍历所有交易 vtx是一个vector<CTransaction> 结构 
		vMerkleTree.push_back(tx.GetHash());  // 将交易Hash放入Merkle树，作为叶子节点 
	int j = 0;
    // 构建Merkle树的非叶节点
    for (int nSize = vtx.size(); nSize > 1; nSize = (nSize + 1) / 2)
     {
		  for (int i = 0; i < nSize; i += 2)
		  {
			  int i2 = min(i+1, nSize-1);
              vMerkleTree.push_back(Hash(BEGIN(vMerkleTree[j+i]),  END(vMerkleTree[j+i]),
			  BEGIN(vMerkleTree[j+i2]), END(vMerkleTree[j+i2])));
		  }
		  j += nSize;
	 }
	// 返回vMerkleTree的最后一个元素，即为MerkleRoot
	return (vMerkleTree.empty() ? 0 : vMerkleTree.back());
}
```
获取[Merkle树分支](https://github.com/trottier/original-bitcoin/blob/92ee8d9a994391d148733da77e2bbc2f4acc43cd/src/main.h#L887)的代码如下：
```
vector<uint256> GetMerkleBranch(int nIndex) const
{
	    // 判断MerkleTree是否为空，若为空，则重新构建
	if (vMerkleTree.empty())
	  BuildMerkleTree();
	// vMerkleBranch用来保存分支数据
	vector<uint256> vMerkleBranch;
	int j = 0;
	// 填充分支数据
	for (int nSize = vtx.size(); nSize > 1; nSize = (nSize + 1) / 2)
	{
		int i = min(nIndex^1, nSize-1);
		vMerkleBranch.push_back(vMerkleTree[j+i]);
		nIndex >>= 1;
		j += nSize;
	}
	return vMerkleBranch;
}
```
