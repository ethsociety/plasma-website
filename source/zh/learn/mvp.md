---
title: 最简可用离子网络（Plasma MVP）
lead: 最简可用离子网络。Learn more about Minimal Viable Plasma by exploring it in depth.
date: 2018-08-21 16:26:02
categories:
tags:
links:
  before:
    引子: /zh/learn
    离子网络（Plasma）框架: /zh/learn/framework.html
  after:
    Plasma Cash: /zh/learn/cash.html
    Plasma Debit: /zh/learn/debit.html
    对比: /zh/learn/compare.html
---

## 最简可用离子网络（Plasma MVP）
[Plasma MVP](https://ethresear.ch/t/minimal-viable-plasma/426) 是一个基于 [UTXO](https://www.investopedia.com/terms/u/utxo.asp)的离子网络链（plasma chain）的极简设计。
最简可用离子网络（Plasma MVP）设计支持高吞吐量的的支付，但是除此外，并不支持相对复杂的构建，比如脚本和智能合约。
那在这个页面，我们会介绍最简可用离子网络（Plasma MVP）背后的设计过程。
你会了解最简可用离子网络（Plasma MVP）到底是怎样工作的，还有它的设计考量。
我们还提供了详细的 [设计文档](/zh/resources#plasma-mvp-specification)，这样你可以实现自己的最简可用离子网络（Plasma MVP）！

---

### 背景
区块链目前非常的慢（比特币和以太坊等主流的链）。其他号称高速的链大多也只是在测试网中达到过每秒几千笔的交易速度。
而即使想满足最简单的支付应用场景，区块链也必须要提速。
如果我们在以太坊上处理交易的话，每秒仅仅能处理10-25笔交易。
而现实生活的交易的话，每秒至少要能处理几千笔。
更别提以太坊的交易费在拥堵的时候非常高，因为交易的数额和交易的优先级无关，所以即使很小数额的交易，链对安全的保证也是没有调整的。这显然很没必要。

#### 链下扩容
我们很久前就知道，有一种相对简单的扩容去区块链的方法，那就是 [造](https://gendal.me/2014/10/26/a-simple-explanation-of-bitcoin-sidechains/) [新](https://bitcoinmagazine.com/guides/what-altcoin/) [链](https://github.com/ethereum/wiki/wiki/Sharding-FAQs).
如果我们增加链的数量，那么，所有的链的处理吞吐量总和也会增加。
但是，每次性能出问题就加新链并不是好办法。
增加新链只会减少整个区块链生态的安全性（有限的总验证算力需要分给更多的链），并且带来糟糕的用户体验。

[侧链](https://blockstream.com/sidechains.pdf)的引入，提供了一种新的选项，这样我们可以创造新的资产锚定于主链的”侧“链。
和其他的区块链系统一样，侧链需要自己的共识机制来确定区块的次序。
如果共识机制崩溃，或者被攻击，用户的资产则有被盗的风险。而这就是我们需要通过离子网络（Plasma）来解决的问题。

---

### 共识
区块链通常需要共识机制。
离子网络（Plasma）链是特别的区块链，这在于即使在离子网络（Plasma）共识机制崩溃的时候，用户的资产也可以有保证。
因此，最简单的离子链依赖于一个叫做“运营员”（**operator**）的组件。
一个“运营员”（**operator**）是离子链的实际操控者，并且负责创建所有区块。
如果你想更多了解区块链的专有名词，我们可以说最简可用离子网络（Plasma MVP）是依赖于“权威证明”的（**Proof-of-Authority**）。

这可能感觉多少有些奇怪，我们通常讲去中心化，在设计区块链协议种，不去信任第三方么？ 
但是离子网络（Plasma）的特性可以保证用户的资产，即使在“运营员”想要做恶的时候，也是可以保证的。
这个核心特性，使得通过最简可用离子网络（Plasma MVP）实现私有链，同时保证用户始终完全掌控资产，成为可能。
接下来，我们会介绍具体是如何保证用户对资产的控制的。

### 充值
用户开始使用离子链，要先在一个以太坊的智能合约中**充值**。
最基本的最简可用离子网络（Plasma MVP）只允许用户充值ETH，但是设计很容易通过扩展来支持ERC20代币。
当用户充值的时候，一个离子链上的区块会专门为这一笔单独的交易产生。
这笔交易会充值的用户创建一个新的和用户充值价值相等的交易余额（output）。
在用户充值之后，用户就可以开始在离子链上进行交易了！

### 交易
用户可以在离子链上用过使用已经拥有的余额（output）进行交易，并产生新的余额。
在实际操作中，这意味着用户需要对一笔交易进行签名，然后把交易发给“运营员”

接下来，离子链“运营员”会把收到的很多交易按照次序打包到一个区块里面。
当“运营员”收到足够多的交易的时候，它就会把这个区块的commitment发到以太坊。
为了解释这个commitment具体的原理，我们要先了解一下Merkle trees。

#### Merkle Trees
Merkle trees在区块链领域中是极为重要的数据结构(广义讲在计算机科学中也是如此)。
简单讲，Merkletrees可以让我们能够在一组数据中写入一些新的数据，在不需要暴露具体数据的情况下，允许用户证明，自己新的数据的确已经被包含在这一组数据中了。

比如，假设我有10个数字，那我可以写入一个这10个数字的commitment，而之后我可以证明，这组数字中包含某个特定的数字。
这样的写入，数据量很小，并且大小固定的，所以发布到以太坊很便宜。

发散这个想法，我们可以用类似的方法写入一组（多笔）新的交易。然后，我们之后可以证明，特定的交易已经被包含。
而这完全就是“运营员”的实际作用！每个新的区块都会包含一组交易，而之后会被转化成一个Merkle tree。
这个Merkle Tree 的根节点就是同每个离子链区块一同发布到以太坊的写入操作（**commitment**）。

### 提现
用户必须能够从离子链对他们的资产进行提现（我们有时候也称之为从链上”退出“）
当用户需要提现的时候，他们会向以太坊提出”退出“的trannsaction。

#### 开始一次退出
因为最简可用离子网络（Plasma MVP）中的资产是由累计资产余额（UTXOs）来表示的，每一次推出都必须指向一个具体的余额。
同时，我们也想确保只有余额的拥有者才能提现。
因此，想要提出一次提现，用户需要在推出的同时提交一个**Merle Proof**证明。
智能合约会验证这个证明，并由此确保产生用户余额的的交易已经被包含在某一个区块中。合约然后会验证余额确实属于退出的发起者。

#### 挑战一个退出
但是，如果这就是所有退出的条件，那用户则可以用已经花掉的余额来提现！我们需要确认提现中所引用的余额确实是未花的，因此，我们引入了“挑战期”（**challenge period**）这一概念。简单的说，挑战期就是一段人们可以对退出提出挑战的时间（通过提供实际的未花交易余额）。
其他用户可以用一个有退出者签名的相应的交易来证明一个余额已经被划掉了。

#### 退出优先级
我们刚刚介绍过的退出协议让人们可以从离子链对他们的资产进行提现。
但是不幸的是，离子链的运营员还是可以做恶的。比如要是运营员进行双花，我们并不能做什么来阻止。运营员甚至可以用一笔不合法的交易余额来提现。

这个问题怎么解决呢？ 我们想要保证有合法余额的用户可以在任何虚假交易发生之前得到他们的资产。
自然的，我们只需要多一些法则来保证用户资产的安全。第一条法则就是，我们规定未花余额要根据他们被加入到离子链的次序，有一个“退出优先级”。具体的优先级是根据未花余额字链中的具体位置决定的。优先级首先有块的次序决定，然后是在单个块中的交易次序，最后是交易中的余额指数。
这样，每一笔未花余额都会有一个固定的位置次序。需要注意的是“老”的交易会优先于“新”的。这意味者如果有一笔不正确交易被加入到了区块中，那么，所有在错误交易之前的交易，都会比错误交易先处理。这样，问题就已经解决了一半！

#### 签名确认
但是如果一笔交易在一笔坏交易**之后**被写入区块怎么办呢？这是完全有可能的，比如用户操作了一笔交易，然后交易被发送到了运营员，但是运营员在写入正确的区块前，先写入了一笔非法的交易。
用户可以尝试退出交易，但是退出的行为可能被其他用户通过对支出的签名进行挑战。
为了处理这种情况，我们会要求交易必须完成两次签名，才会被认为合法。
每当用户发起一笔交易，他们会先对交易签名，让交易被加入到一个区块里。
然后，一旦交易被加入到区块，用户会再次进行一次签名，这被称为**确认签名**。
如果用户正确的遵循这个法则的话，那他在确认交易已经被加入区块之前，绝不会发出确认签名。

然后，我们对退出挑战增加一个要求，那就是你必须要提供确认签名。
这样以来，如果运营员在一笔非法交易之后，在区块中记录了用户的交易，那用户只需要**拒绝对交易确认签名**就好。这意味着，在一笔非法交易之后被加入到区块的交易都不会得到确认签名，也就是永远不被承认。这样以来，遵守法则的用户就可以拿回他们的资产。


#### 监控离子网络侧链
如果离子链在正常运转，那用户不需要做任何处理。但是万一发生了不可扭转的错误，那用户的钱包就会开始自动的对用户的离子链资产开始提现。这个自动提现机制可以保证用户的资产是安全的。即使在非常条件下，比如运营员做恶的情况。即使这对保证用户的资产100%的安全非常关键。这就是为什么说在系统设计的时候，就要对用户行为进行激励，让用户乐意去为别的用户运行验证离子链的程序，这与闪电网络的机制类似。

#### 更可用的离子网络
确认签名解决问题，但是会带来糟糕的用户体验。
用户需发交易要签名，确认交易上链还要再签一次。而且第二次签名也要上链（离子链），占用更多本来可以记录交易的空间！
[更可用离子网络More Viable Plasma](https://ethresear.ch/t/more-viable-plasma/2160)缩写为 
MoreVP，是最简离子网络MVP（Minimal Viable Plasma)的扩展，并且不需要确认签名（第二次签名）。
最简离子网络必须依赖确认签名，因为提现需要按照被提现的余额结果在链中的位置来处理。
简单说，更可用离子网络（MoreVP）修正了这个用户可以对资产进行提现的过程。
提现的次序变成了基于 创造被提现交易余额 **最近输入值**的位置
新的次序要求对提现挑战进行非常多的更新，来保证只有诚信的用户可以对资产自行提现。
一个[更新版本的](https://github.com/omisego/elixir-omg/blob/develop/docs/morevp.md) 更可用离子网络正在由OmiseGO开发和维护。





