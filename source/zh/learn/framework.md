---
title: 离子网络（Plasma）框架
lead: 框架总览。Get a high-level overview of the tools provided by the plasma framework.
date: 2018-08-21 16:26:02
categories:
tags:
links:
  before:
    引子: /zh/learn
  after:
    最简可用离子网络（Plasma MVP）: /zh/learn/mvp.html
    Plasma Cash: /zh/learn/cash.html
    Plasma Debit: /zh/learn/debit.html
    对比: /zh/learn/compare.html
---

## 离子网络（Plasma）框架
首先要记住，离子网络（Plasma）是一个可以遵循的，用来搭建可扩容的区块链应用的*框架*或者*范式*。
所以并没有某一个具体的项目叫做离子网络（Plasma）。而是有很多不同的使用离子网络（Plasma）中提到的工具的项目。
这个一开始可能很让人费解，需要一些耐心来理解：）

离子网络（Plasma）作为框架的本质，让它不太容易解释。
离子网络（Plasma）原著[论文](http://plasma.io/plasma.pdf) 中也提到了这个问题。
如果不举具体的例子的话，很难描述离子网络（Plasma）到底*是啥*。
但是，同时也要注意，这些具体的例子只是离子网络（Plasma）潜力的冰山一角。

因此，在介绍具体的例子之前，得先把离子网络（Plasma）作为一种框架来解释。
如果你习惯把离子网络（Plasma）作为一个具体的项目（代码）来看，那么这个章节可能会感觉非常的模糊，不知所云，但别担心，我们会在之后的讲具体应用部分加入细节。

---

### 我们为什么需要离子网络（Plasma）?

在我们尝试解释什么*是*离子网络（Plasma）之前，我们应该先了解*为什么*离子网络（Plasma）存在。
这将是一个相对概括的概述，并将尽量避免详细的介绍，但它应该为其余的部分奠定基础。
基本上，它归结为当前区块链系统太*慢*了。
想满足处理任何现实容量的应用场景，比如简单的支付，区块链大约需要处理每秒几千笔交易的订单。
根据网络条件，以太坊目前每秒最多只能支持20-30次交易。

这就是区块链应用目前面对的最大难题之一，扩容性问题 **blockchain scalability** problem。
我们希望扩容区块链系统，但我们也希望确保以保持安全性和去中心化。毕竟，这是区块链的核心卖点。

项目正在以许多不同的方式解决可扩展性问题。
其中一些项目试图通过升级区块链本身来使区块链更具可扩展性。我们经常称之为“第1层”扩容，因为我们正在修改基础层。
其他项目正试图在现有系统的上层扩容。我们通常将此称为“第2层”扩容，因为我们正在添加新图层而不更改底层系统。

离子网络（Plasma）属于第二类“第2层”扩容项目。
了解第2层项目如何工作原理重要。许多第2层设计来自这个核心思想：即并不是每个人都需要了解网络上发生的每一笔交易。
重要的事重复几遍：并不是每个人都需要了解网络上发生的每一笔交易。
重要的事重复几遍：并不是每个人都需要了解网络上发生的每一笔交易。
（这个思想和闪电网络的出发点是一样的）

让我们思考一下。
假如你很喜欢和咖啡，每天早上都会从当地的咖啡店买一杯咖啡。如果你使用咖啡店的提前充值的手机钱包，支付会更方便。
你只要充值一次，就不用每次都掏出钱包。
对于商家来说，它也更快，更便宜，因为他们不必每次都通过第三方支付。
双赢。

如果我们细想这一点，就会意识到，这基本上将钱投入到您在咖啡店维护的本地“记账本”中，和提前充值的手机钱包一回事。
世界上的其他人没有必要知道你和咖啡点之间的交易
只要你和咖啡店同意对你当前的充值余额达成共识就好。

这其实就是大多数“第2层”项目的核心。
例子很简单 - 它只涉及两个参与者（你和咖啡店）。
下面看我们是否可以将这个想法扩展到区块链。
是否有可能暂时将资金从主区块链转移到更小（同属更便宜，更快）的区块链（侧链）。然后在第二条链上转移资金，交易，并最终结算，转回资金？

在这里，我们开始看到“主链”到“侧链”的演变。
基本思想是，我们可以从一个链中获取资产并通过将资产锁定在主链（或“根链”）上，同时在侧链上再次“创建”它们。这等同于将它们从主链转移到了“侧链”。
当您想要返回时，您只需要“销毁”侧链上的资产并在主链上解锁它们。

这听起来很简单，但有一个很大的问题 - 有人必须同意在侧链上“创造”这些资产。
那么谁可以“创造”资产？答案是：共识机制。
具体说，如果侧链共识机制运作正常，那么您的资产是安全的。不幸的是，侧链通常远远不如主链安全。
如果有人能凭空从侧链“造币”，而这些资金与锁定在根链上的资产不对应，那么他们就可以“销毁”这些凭空创造的资产，并从主链窃取大量真实的资金。

显然这并不理想。
只有侧链是安全的，你的资产才是安全的。
但如果侧链发生问题，那么你的资产可能会被盗！这是我们需要离子网络（Plasma）的地方！
离子网络（Plasma）是为了发挥侧链的优势（更快，更便宜），同时确保存储在侧链上的资产始终是安全的（只要主链是安全的）。
当然我们没有得到*所有*侧链的好处，但我们保留了一些最重要的东西（比如能够便宜地进行交易），同时还保持安全性。
离子网络（Plasma）的基本保障是，如果侧链发生安全性问题，所有用户资产都可以“撤回”到根链。

---

### 离子网络（Plasma）核心概念和组件
区块链又慢又贵。那我们希望让区块链变得更快更便宜，同时不牺牲安全性。
这就是离子网络（Plasma）存在的意义。

所以离子网络（Plasma）到底是啥？
离子网络（Plasma）是一种搭建可扩容去中心话应用，同时不为速度牺牲安全性的*方法*。
那下面让我们了解一下是这称为可能的离子网络核心组件。

##### 链下执行
离子网络（Plasma）应用的大部分运算都是在主链（或根链）之外的（比如以太坊）。
主链通常都比较慢，因为需要保证安全性。所以，如果可能的话，应用应该尽量在主链外执行运算。（你买咖啡，不需要让世界都知道）


比如说在[Plasma MVP](/zh/learn/mvp.html)中, 几乎所有的交易时间（tx）都是在以太坊之外进行的。
只有充值（deposit）和提现（withdrawals），这样资产进入和退出的关键点是在智能合约上执行的。
这个流程是离子网络（Plasma）的标准。所有不需要进出只能合约的数据和交易，八成都可以在链下执行。

##### 状态承诺
当我们在链下执行大量交易的时候，我们需要一种方法来确保所有的改动都是最终的（不可继续篡改）。
为此，我们引入了“状态承诺”的概念（state commitment）。
一个[state commitment](https://en.wikipedia.org/wiki/Commitment_scheme) 是一种可以存储一个压缩的应用状态的加密学方法。

但是，如果把和应用相关的所有状态都存下来的话就违反了离子网络（Plasma）的本意。
因此我们通常会用 [Merkle trees](/zh/learn/mvp.html#merkle-trees).
这些“承诺”就变成了应用的存档点。（打过游戏对吧，打boss前要存档哈）

##### 退出
离子网络（Plasma）应用会通过“状态承诺”来完成用户从侧链的退出。我们通常称之为“退出应用”（exiting application）。

让我们来用一个支付网络的应用来说明这个应用场景。
这个支付网络的“状态承诺”（state commitments)八成会包含每个用户的余额信息。
如果一个用户想要他们的资产退出（exit）这个应用，他需要想主链的智能合约证明，他在侧链的确有相应的资产来提现。
为完成这个证明，我们通常会使用[Merkle proof](https://blog.ethereum.org/2015/11/15/merkling-in-ethereum/).

---

这些就是基本的离子网络（Plasma）的基本组件。
结下来我们讲介绍Plasma MVP,第一个我们会详细介绍的离子网络（Plasma）的应用.
在那个例子里，我们会探索如何用这些基本组件来搭建一个真实的基于离子网络（Plasma）的支付网络（payment network）。
