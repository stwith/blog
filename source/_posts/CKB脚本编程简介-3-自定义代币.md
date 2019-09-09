---
title: 'CKB脚本编程简介[3]: 自定义代币'
date: 2019-09-09 12:26:56
tags:
- Bitcoin
- Nervos

category:
- Nervos
---

CKB的 Cell 模型和VM支持许多新的用例。然而，这并不意味着我们需要抛弃现有的。如今区块链中的一个常见用途是令牌发行者发布具有特殊目的/意义的新令牌。在以太坊中，我们称之为ERC20令牌，让我们看看我们如何在CKB中构建类似的概念。为了与ERC20区分，在CKB中的令牌我们称之为 `user defined token` , 简称UDT。

本文使用CKB v0.20.0版本来演示，具体来说，我在每个项目中使用以下提交的版本:

* [ckb](https://github.com/nervosnetwork/ckb): 472252ac5333b2b19ea3ec50d54e68b627bf6ac5
* [ckb-duktape](https://github.com/nervosnetwork/ckb-duktape): 55849c20b43a212120e0df7ad5d64b2c70ea51ac
* [ckb-sdk-ruby](https://github.com/nervosnetwork/ckb-sdk-ruby): 1c2a3c3f925e47e421f9e3c07164ececf3b6b9f6

# 数据模型

与以太坊为每个合约账户提供了独特的存储空间不同，CKB是在多个 Cell 之间传递数据。Cell 的 Lock Sript 和 Type Sript 会告知 Cell 属于哪个帐户，以及如何与 Cell 进行交互。其结果是，与ERC20将所有令牌用户的余额存储在ERC20合约的存储空间中不同，在CKB中，我们需要一种新的设计来存储UDT用户的余额。

当然，我们可以构造一个特殊的 Cell 来保存所有UDT用户的余额。这个解决方案看起来很像以太坊的ERC20设计。但是出现了几个问题：

* 令牌发行者必须提供存储空间以保存所有用户的余额。随着用户数量的增长，存储空间也将增长，这在CKB的经济模型中，不是一个高效的设计。
* 考虑到更新CKB中的 Cell 实际上是在销毁旧 Cell 并重新生成新 Cell ，因此保存所有余额的单个 Cell 会产生瓶颈：需要更新UDT余额的每个操作都必须更新唯一的 Cell, 使用过程中将会产生冲突。

虽然有一些解决方案可以缓解甚至解决上述问题，但我们开始质疑这里的基本设计：将所有UDT保存在一个地方真的有意义吗？一旦转账，UDT应该真的属于接受者，为什么余额仍然保持在中心化的地方？

这引出了我们提出的设计：

1. 一个特殊的 Type Script 表示此 Cell 存储UDT。
2. Cell 数据的前4个字节包含当前 Cell 中的UDT数量。

这种设计有几个含义：

* UDT Cell 的存储成本始终是恒定的，与存储在 Cell 中的UDT数量无关。
* 用户可以将 Cell 中的全部或部分UDT转账给其他人。
* 实际上，可能有许多 Cell 包含相同数量的UDT。
* 用于保护UDT的 Lock Script 与UDT本身分离。
