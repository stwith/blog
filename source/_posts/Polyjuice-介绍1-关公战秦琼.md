---
title: 'Polyjuice 介绍1: 关公战秦琼'
date: 2019-09-16 16:19:23
tags:
- Bitcoin
- Nervos

category:
- Nervos
---

![](https://i.loli.net/2019/09/16/b4ajfvXK5xSnu92.jpg)

### Polyjuice 是什么？ 

按照Github里的描述:
> Polyjuice is a layer 2 solution that provides a Web3 compatible interface on top of Nervos CKB. The design goal here is 95%+ compatible with existing Ethereum solution, so that your Solidity smart contracts and tools for Ethereum could work directly on top of Nervos CKB.

简单来说，Polyjuice 就是一整套实验性质的基于CKB的Layer2框架，近乎完整地兼容了 Ethereum 生态，你可以在上面直接部署 Solidity 合约，甚至可以使用 Web3JS.

当然，与原生的Ethereum 还是有区别的:

1. 在Polyjuice上，每个人都需要为自己的状态存储付费；
2. 可以体验到支持热升级而无需硬分叉的EVM；
3. 不必忍受 Ethereum 状态爆炸或是交易拥堵的困扰；

### 关公战秦琼

是的，刚开始尝试理解 Ployjuice 的时候我也是一样，对这两条链是如何结合到一起的也是一脸懵逼。ETH 怎么就能成为CKB的Layer2了？ 

根据以前的理解，CKB和ETH两条链从代币模型到合约完全不相同，CKB是UTXO模型并且每个用户都需要为自己的存储付费；Ethereum是Account模型，用户的存储不需要付费。怎么想两条链都不可能兼容到一起好吧？脑海里满是**关公战秦琼**的画面。 

![](https://i.loli.net/2019/09/16/PhIQbCKsYWt9rEm.jpg)


### Polyjuice的配方

通读了一遍代码之后，我开始理解了。接下来，将尝试用我自己的理解给你讲讲 Polyjuice 到底是如何实现的。能力不高水平有限，如有错误欢迎砍脸....

*那么，问题来了，关公战秦琼一共分几步？*

#### 第一步：打电话叫他俩过来

在运行Polyjuice网络之前，需要部署几个[Scirpt](https://github.com/nervosnetwork/polyjuice/tree/master/c)。其中包含了Ethereum 的私钥解锁代码，sha3算法，以及合约的解锁代码。他们统统按照CKB的方式，写成了`Scripts`形式。

以私钥解锁Script为例，参数就变成了ETH的公钥，而不是CKB的公钥，你需要用ETH私钥来解锁啦，你的CKB私钥没有权限解锁哦。

Polyjuice 通过`get_live_cell`来获取CKB全网哪些Cells 是使用特定的hash的Lock Script, 就知道全网有多少个 Cells 是以Ethereum的形式存在的。


你可以这么理解，当一个 Cell 的 `Lock Script` 改写成 我们指定的`Lock Script`, Polyjuice
就认为这个Cell 是 Ethereum 的块，只有用ETH私钥才能做解锁操作。这个Cells里面的CKB就通过**预先设定好的比例**，就可以认为是一定数量的ETH.

Polyjuice 获取到全网的`Etheruem Cells`之后，就可以按照 EThereum 的方式存储到本地。这样，所有节点都可以共享一个ETH全网状态。

Polyjuice 通过本地的服务，对外提供Web3的接口，完美的兼容了Web3。

#### 第二步：打一架打一架

原生ETH的转账之后是Account模型的数据加减，但是对Polyjuice来说，逻辑是这样的：

一笔交易（A->B):
 对使用者来说，是A的以太坊地址发送到B的以太坊地址；
 对Polyjuice来说， 是能用A的以太坊私钥解锁的Cells 转变成了 能用B的以太坊私钥解锁的Cells.

【此处应该有图】

这样，就可以完美实现基于CKB的Ethereum转账，对Web3用户来说，使用体验和原生几乎毫无差别。

【此处应该吹一波CKB的可插拔密码学原语.(找Ryan领钱，每贴5毛)】

#### 第三步：各回各家

想一下: CKB的原生Cells只需要更换Scripts就能变成ETH，完全无痛。那么反过来，ETH可以转成CKB吗？ 

答案是当然可以。你只需要用你的以太坊私钥解锁这个定制Cells, 然后替换回默认的Scripts,你就可以换回你的CKB。


### 小结

