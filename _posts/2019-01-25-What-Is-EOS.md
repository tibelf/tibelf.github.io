# What is EOS
最初 EOS 是基于 ETH 的 ERC-20 协议发现的代币，在 2018 年的 8 月份左右，EOS 主网才正式上线。

EOS 的每个块的出块速度可以实现平均在 500 ms。

其实我们可以发现 EOS 的很多实现和设计逻辑都是参考了 Bitshare 和 Steemit，毕竟都是同一个人开发实现的。

## 共识算法
通过白皮书可以看到，宣称是采用 ***DPOS + PBFT*** 共识算法。但是目前 EOS 实际采用的共识算法是 ***DPOS***。

为什么这么说呢，因为 `bft_irreversible_blocknum` 唯一修改的地方是在 `fork_database` 类的私有方法 `set_bft_irreversible`，调用这个私有方法的只有公有方法 `add`，目前从代码中未找到其入口，之前是在 `controller` 类的 `push_confirmation` 方法中有调用，但是好像自从有人提了一个 [issue](https://github.com/EOSIO/eos/issues/4965)，通过观察代码，发现在 [Remove push_confirmation. Add pre-key of transaction keys](https://github.com/EOSIO/eos/commit/16fdb4a15404273c16ce859b6adef21a44be6b9a#diff-147a7174fb705ef227e4e28b336acd4f) 提交中被移除了。

所以我比较担心是不是他们已经放弃实现 ***DPOS + PBFT*** 共识算法了？

## 燃料
EOS 区别于 BTC，ETH 的一个比较大的特性就是所有在 EOS 网络上触发的交易，交易手续费实际是 0。

可能有人会问，那不是很危险，我频繁进行交易，不就把这个网络搞挂了吗？注意，上面我讲的实际手续费为 0，EOS 其实提出来燃料概念，交易还是需要燃料的，只是这个燃料是可以通过抵押 EOS 代币来获取，并且这个抵押的代币最终是可以全盘回收的，所以我称之为 ***实际手续费为 0 = 免费***。

EOS 提出的燃料包括 CPU，Memory，RAM 三个新概念

* ***CPU***：执行交易的时候需要耗费，以毫秒为单位；
* ***Memory***：进行交易的内容传输时候需要耗费，以字节为单位；
* ***RAM***：存储账号的相关信息需要消耗，以字节为单位；

具体的介绍我会在之后的文章中提到，这边就先讲到这里。

## 每一个 PB 节点一轮可以挖多少个块

通过代码看到是 12 块，并非是白皮书里面写的 6 块；

此参数配置在 `config.hpp` 文件中由 `producer_repetitions` 定义。


## 单词缩写解释
* LIB：last irreversible block 
* BP：Block Producer

# 摘要
[EOS white paper 2.0](https://github.com/EOSIO/Documentation/blob/master/TechnicalWhitePaper.md)

[Introduction to EOS](https://blockchainhub.net/blog/blog/introduction-to-eos-blockchain/)

[EOS-Analysis-and-Valuation](https://multicoin.capital/wp-content/uploads/2018/04/EOS-Analysis-and-Valuation.pdf)

[issue 1144](https://github.com/bitshares/bitshares-core/issues/1144)

[issue 2191](https://github.com/EOSIO/eos/issues/2192)

[issue 6154](https://github.com/EOSIO/eos/issues/6154)

[issue 6281](https://github.com/EOSIO/eos/issues/6281)

[issue 4965](https://github.com/EOSIO/eos/issues/4965)

[issue 5946](https://github.com/EOSIO/eos/issues/5946)

[block producer reward](https://eosio.stackexchange.com/questions/2773/how-are-eos-block-producers-rewarded)
