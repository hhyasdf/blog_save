---
title: Bitcoin and Cryptocurrency Technologies - Lecture 2
date: 2016-12-03
categories: Bitcoin and Cryptocurrency Technologies
tag: Bitcoin, Cryptocurrency Technologies
---

### How Bitcoin Achieves Decentralization###

***************************************************

#### Segment 2.1 Centralization vs. Decentralization####

&nbsp;&nbsp;&nbsp;&nbsp;这部分我们主要讨论的是，我们怎样才能获得一个权力分散的（decentralized）货币系统呢？

&nbsp;&nbsp;&nbsp;&nbsp;让我觉得有趣的是，Bitcoin实现分散的方法不是纯技术，而是技术和clever incentive engineering的结合，在这个lecture之后你会对它是怎样实现的有一个清楚的认识，并且你会清楚Bitcoin的工作、运转以及安全原理。

&nbsp;&nbsp;&nbsp;&nbsp;事实证明，在数字技术中，集权与权力下放概念竞争占有很重要的角色。为了理解Bitcoin的权利分散，我们将从一个观点开始——decentralization almost always is not all or nothing。几乎没有一个系统是完全集权或分权的，一个很好的例子是Email。我会说Email从根本上是一个分权的系统。它基于一个标准间隔协议——SMTP，但实际上受控于集权的网络服务提供商。这也许可以帮助我们理解Bitcoin中的工作原理。

&nbsp;&nbsp;&nbsp;&nbsp;让我们深入了解一些Bitcoin的分权的技术方面。我会将它分为至少5个不同的问题：

* Who maintains this ledger of transactions?
* Who has authority over which transactions are valid?
* Who creates new bitcoins?
* Who determines how the rules of the system change?
* How do bitcoins acquire exchange value?

&nbsp;&nbsp;&nbsp;&nbsp;这些都是Bitcoin protocol关于分权方面的成分，前三个问题是我们在这个lecture里将会思考的。接下来说到的比特币分权的内容都会围绕这三个问题。还需要强调的是在协议之外还有一些分权的方面，包括像Bitcoin exchanges（将Bitcoin转换成其他货币）、wallet software和其他一些服务提供商。因此，尽管底层的协议是分权的，在其上的服务也可能会是集权的。

&nbsp;&nbsp;&nbsp;&nbsp;为了深入理解这一点，让我举出Bitcoin的基于分权的三个不同的方面：

Peer-to-peer network (may be the closest thing to purely decentralized):
&nbsp;&nbsp;&nbsp;&nbsp;open to anyone, low barrier to entry.

Mining:
&nbsp;&nbsp;&nbsp;&nbsp;open to anyone, but inevitable concentration of power often seen as undesirable

Updates to software:
&nbsp;&nbsp;&nbsp;&nbsp;core developers trusted by community, have great power.

&nbsp;&nbsp;&nbsp;&nbsp;其中Peer-to-peer network无疑是最接近纯分权的方面，虽然它也被网络状况所影响；对于开采bitcoin方面则受到了电脑资源的限制，而对于比特币软件则完全由其开发者所控制。

***********************************************

#### Segment 2.2 Distributed Consensus####

&nbsp;&nbsp;&nbsp;&nbsp;建立一个电子现金系统时，在技术层面，需要解决的主要问题：distributed consensus（分布式共识）。consensus protocols（共识协议）发展的传统方向是分布式系统中的可靠性问题。

&nbsp;&nbsp;&nbsp;&nbsp;类比ScroogeCoin中解决的double-spending问题，distributed consensus问题意味着当分布式系统中产生一个新的动作时，要么系统中所有的节点都记录一样的结果，要么都不记录，因为系统中某些节点可能会出现错误，当出现错误时，如果错误的节点不保存该动作而其他节点保存，将会出现错误。我们也可以想到，如果我们实现了一个distributed consensus protocol，我们可以用它来建立一个复杂的全球的分布式键-值存储（可以将任何键或名称映射到任意值），然后我们就可以实现很多的应用。像DNS等。最酷的是BitCoin已经解决了这个问题。

&nbsp;&nbsp;&nbsp;&nbsp;distributed consensus的技术目的很简单，假设有常数n个节点或者进程，每个节点都有一些输入值。这时候distributed consensus协议就起作用了。协议需要满足两点。首先，协议是会终止（不会无线循环）的，并且所有正确的节点都要能够根据输入给出一致的值（因为有些节点的判断可能会是错误的甚至是恶意的）。而后，这个一致的值不能是任意的，而应该是与输入对应的期望的（正确的）值。

![Peer-to-peer-network](/images/Peer-to-peer-network.jpg)

&nbsp;&nbsp;&nbsp;&nbsp;让我们在Bitcoin环境下看看它意味着什么。为了理解distributed consensus在Bitcoin中是怎样工作的，首先提醒一下Bitcoin是一个peer-to-peer（对等网络）系统。这就意味着当Alice想要付钱给Bob，她需要将这个交易广播给所有对等网络中的节点。你可以从图中看到这个交易的结构，它和我们在lecture 1中的GoofyCoin中看到的是相似的。一个交易需要有Alice的签名来证明交易的正确性，并且需要有Bob的公钥来表示这是支付给Bob的，结构中也含有一个哈希指针指向该比特币之前的交易信息。她将会将这个数据结构广播给所有的对等节点。但是，此时Bob的电脑并没有在网络中，如果Bob想要知道这个交易发生了（有人向他支付了钱），他也许需要在该对等网络中运行一个节点来监听交易信息。事实上，如果只是要接收这笔钱的话，是不需要监听的。这些钱无论他是否在该网络中运行了一个节点都会是他的。

&nbsp;&nbsp;&nbsp;&nbsp;所以，给定对等系统，节点可能想要达成的共识是什么？因为一个系统中可能会有大量的用户在同一时间广播他们的交易，我们需要达成共识的是，已经发生的交易和交易的顺序。它具体意味这什么呢？在Bitcoin中为了达成共识，在任何给定的时间里，所有的对等网络中的节点都保存了一个已经达成共识了的交易数据块序列，类似于ScroogeCoin中的transaction block chain。并且每个节点会有一组它所知道的未完成交易，对于这些交易尚未产生共识。根据定义，每个节点可能有一组略有不同的版本的它听说的未完成的事务。因为对等网络不是完美的，可能一些节点听说了一个交易，而另外的节点没有听过。

![consensus-work](/images/consensus-work.jpg)

&nbsp;&nbsp;&nbsp;&nbsp;根据这个设定，我们有了一串每个人都承认的块序列，每个块都是一系列交易（灰色部分）。如图，现在系统中有三个节点。每个人都有提出和输入的它所听到的一系列未完成交易，三个节点一起执行consensus protocols，你可以选择任易的正确的交易块加入block chain，哪怕它只是由一个节点提出的。对于一个正确的交易块，其中的每个交易必须是合法地（有正确的数字签名等等）。如果一个交易没有加入该block，它可以等到下一个block再加入。

&nbsp;&nbsp;&nbsp;&nbsp;如果你将传统的distributive consensus理论应用于比特币，那么最终就会得到上面的系统。这跟Bitcoin的工作原理有一些共同点，但不是准确的Bitcoin的工作方式，因为由于各种原因（如下），这种方式的技术难度很大。

* Nodes might crash
* Nodes might outright be malicious（节点是完全恶意的）
* Network is imperfect:
&nbsp;&nbsp;&nbsp;&nbsp;Not all pairs of nodes connected
&nbsp;&nbsp;&nbsp;&nbsp;Faults in network
&nbsp;&nbsp;&nbsp;&nbsp;Latency: one particular consequense is that there is no notion of global time. It means that not all nodes can agree to a common odering of events simply based on cbserving timestamps.

Some well-known protocols:

* Pxos:
&nbsp;&nbsp;&nbsp;&nbsp;Never produces inconsistent result, but can(rarely) get stuck.

*********************************************************

#### Segment 2.3 Consensus without Identity: the Block Chain####