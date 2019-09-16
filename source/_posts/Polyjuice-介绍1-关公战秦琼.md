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

## Polyjuice 是什么？ 

按照Github里的描述:
> Polyjuice is a layer 2 solution that provides a Web3 compatible interface on top of Nervos CKB. The design goal here is 95%+ compatible with existing Ethereum solution, so that your Solidity smart contracts and tools for Ethereum could work directly on top of Nervos CKB.

简单来说，Polyjuice 就是一整套实验性质的基于 CKB 的 Layer2 框架，近乎完整地兼容了 Ethereum 生态，你可以在上面直接部署 Solidity 合约，甚至可以使用 Web3.js。

当然，与原生的 Ethereum 还是有区别的:

1. 在 Polyjuice 上，每个人都需要为自己的状态存储付费；
2. 可以体验到支持热升级而无需硬分叉的 EVM；
3. 不必忍受 Ethereum 状态爆炸或是交易拥堵的困扰；

## 关公战秦琼

是的，刚开始尝试理解 Ployjuice 的时候我也是一样，对这两条链是如何结合到一起的也是一脸懵逼。ETH 怎么就能成为 CKB 的 Layer2 了？ 

根据以前的理解，CKB 和 ETH 两条链从代币模型到合约完全不相同，CKB 是 UTXO 模型并且每个用户都需要为自己的存储付费；Ethereum 是 Account 模型，用户的存储不需要付费。怎么想两条链都不可能兼容到一起好吧？脑海里满是**关公战秦琼**的画面。 

![](https://i.loli.net/2019/09/16/PhIQbCKsYWt9rEm.jpg)


## Polyjuice 的配方

通读了一遍代码之后，我开始理解了。接下来，将尝试用我自己的理解给你讲讲 Polyjuice 到底是如何实现的。能力不高水平有限，如有错误欢迎砍脸....

*那么，问题来了，关公战秦琼一共分几步？*

### 第一步：打电话叫他俩过来

在运行 Polyjuice 网络之前，需要部署几个 [Scirpt](https://github.com/nervosnetwork/polyjuice/tree/master/c)（关于 Script 的更多资料可以参考 Xuejie 的 [Introduction to CKB Script Programming](https://xuejie.space/)）。其中包含了 Ethereum 的私钥解锁代码，sha3 算法，以及合约的解锁代码。他们统统按照CKB的方式，写成了`Scripts`形式。

以私钥解锁 Lock Script 为例，参数就变成了 ETH 的公钥，而不是 CKB 的公钥，你需要用 ETH 私钥来解锁啦，你的 CKB 私钥没有权限解锁哦。

Polyjuice 通过`get_live_cell`来获取 CKB 全网哪些 Cells 是使用 Ethereum 特定的 hash 的 Lock Script, 就知道全网有多少个 Cells 是以 Ethereum 的形式存在的。


你可以这么理解，当一个 Cell 的 `Lock Script` 改写成我们指定的 Ethereum 形式的 `Lock Script`, Polyjuice 就认为这个 Cell 是以 Ethereum 形式存在的 Cell，只有用 ETH 私钥才能做解锁操作。而这个 Cells 里面的 CKB 就通过**代码中预先设定好的比例**，被认为是一定数量的 ETH。

Polyjuice 获取到全网的`Etheruem Cells`之后，就可以按照 EThereum 的方式存储到本地节点。这样，所有节点都可以共享一个 ETH 全网状态。

Polyjuice 通过本地节点的服务，对外提供 Web3 的接口，完美的兼容了 Web3。

### 第二步：打一架打一架

原生 ETH 的转账之后是 Account 模型下的数据加减，但是对 Polyjuice 来说，实现逻辑是这样的：

```
一笔交易（A->B):
 对使用者来说: 是 A 的以太坊地址发送到 B 的以太坊地址；
 对 Polyjuice 来说: 是能用 A 的以太坊私钥解锁的 Cells，转变成了能用 B 的以太坊私钥解锁的 Cells.
 ```

【此处应该有图】

这样，就可以完美实现基于 CKB 的 Ethereum 转账，对 Web3 用户来说，使用体验上和原生的 Ethereum 几乎毫无差别。

*【此处应该吹一波 CKB 的可插拔密码学原语.(找 Ryan 领钱，每贴5毛)】*

### 第三步：各回各家

想一下: CKB 的原生 Cells 只需要更换 Scripts 就能变成 ETH，完全无痛。那么反过来，可以把这些 ETH 再次转回 CKB 吗？ 

答案是当然可以。你只需要用你的以太坊私钥解锁这个定制 Cells, 然后替换回默认的 Scripts,你就可以换回你的 CKB，不多不少，你的 CKB 还在这里等着你，只是你之前创造的 ETH 不再存在了。


## 小结

呐，以上是 Polyjuice 的 Ethereum 的基本逻辑。是不是感觉少了点什么？ 有点意犹未尽的样子。

别着急呐，下一篇我们会讲 Polyjuice 的精华部分: 与 EVM 结合，来实现 Ethereum On CKB 的合约模型。

敬请期待（欢迎打钱）

