---
title: Nervos Script 初体验
date: 2019-07-09 20:37:05
tags:
- Bitcoin
- Nervos

category:
- Nervos
---

# 【首杀达成】Nervos Script 初体验

## 起因

一直很好奇 Nervos 的 Script 到底是怎么运行的，也想了解CKB的 Risc-V 的神奇之处，想自己亲手体验下，但是目前还没找到这方面的文档，我们就打算把这实践当做一个CKB的高难度的副本，来做一次开荒抢首杀的体验～

## 合约编写

理论上，CKB底层是Risc-V，任何可以编译的语言都可以使用，这里我们使用了C来写一个简单的Demo：

```C
int main(int argc, char* argv[])
{
  if (argc == 3) {
    return -2;
  } else if (argc == 5 ) {
   return -3;
  } else {
   return 0;
  }
}
```

简单描述下，合约接受参数，如果参数数量 = 3 ，就返回-2，如果参数数量等于5，就返回-3，其他任意参数都可以返回0.

合约使用了`riscv-tools`编译，编译之后为6.1KB，已知1CKB = 1Bytes...所以我们大概需要6100CKB 来存储这个合约.... (貌似有点大....)

*本来挖到了2个块一共3800CKB..然后发现钱不够...也再挖不出了...就默默寻求场外帮助了...丹妞赞助了1100WCKB...(如果是主网Token多好...)*

## 合约部署

*有钱了做事情就有底气了...(我要打10个！)*

接下来就需要把编译好的合约部署到网络里。首先读取这个二进制文件，转成16进制，然后构造一笔交易，塞到Output 的 Data里面去，效果如下：
![](https://i.loli.net/2019/07/09/5d240eb09d60741721.jpg)

[交易TX](https://explorer.nervos.org/transaction/0x2928ab517396220e3c8336588bb70a445e4fc271975a3a3cf59309253e05de78)

*这笔交易我们构造了一个1100W的Cell空间，有钱就是NB！*

## 合约调用

接下来我们构造了一笔交易，将这个 Cell 空间 作为Input，分成了5个Output，也就是创建了5个新的Cell，这5个Cell的 `LockScript` 的`Codehash`都引用了上面部署的Data的`Hash`, 调用了上面Data的代码;`args`分别给了0-4个参数，来校验能不能正确解锁。

[交易TX](https://explorer.nervos.org/transaction/0x395fc85dc84950ee394447418405e33413e3c0700e12d6e67c98452e797b0a3f)

> 知识点：VM在尝试解锁的时候，会默认帮我带一个参数进去，所以这5个Cell的参数个数分别是：1-5，所以我应该是可以解锁1，2，4个Cell的...另外....3个Cell 任何人都能解锁...他不会验证私钥的...

## 合约验证

现在我们尝试解锁第1个Cell，把他们作为Input!

当当当～成功了！入链了！

[交易TX](https://explorer.nervos.org/transaction/0x6a39d410a1eb50a5e6fcf2c3722c2c652df6271860545bc1b9ecc95cae0e4c76)

接着我们尝试解锁第3个Cell～

这时候报错了：

```
Error: {"code":-3,"message":"InvalidTx(ScriptFailure(ValidationFailure(-3)))"}
```

说明解锁失败～我们的代码生效了！

到此，合约调用和验证的完整流程结束撒花～～

## 最后

最后附上吐槽和总结...

这次的首杀过程中，Nervos的VM的灵活性带给我们无限的可能，我们可以更加灵活的创造，不受VM的限制。

但是...这种灵活性会给安全性带来一系列的挑战，所以开发者需要更谨慎的开发，避免出现风险。

## 彩蛋环节

我们留了2个 Cell 未解锁，如果你能成功解锁并留言TX的话，我个人赠送1个木猿～一共2枚～

一起来玩吧～
