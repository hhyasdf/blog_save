---
title: Bitcoin and Cryptocurrency Technologies - Lecture 2
date: 2016-12-03
categories: Bitcoin and Cryptocurrency Technologies
tag: Bitcoin, Cryptocurrency Technologies
---

### How Bitcoin Achieves Decentralization###

***************************************************

#### Segment 2.1 Centralization vs. Decentralization####

&nbsp;&nbsp;&nbsp;&nbsp;这部分我们主要讨论的是，我们怎样才能获得一个去中心化的（decentralized）货币系统呢？

&nbsp;&nbsp;&nbsp;&nbsp;让我觉得有趣的是，Bitcoin实现去中心化的方法不是纯技术，而是技术和clever incentive engineering的结合，在这个lecture之后你会对它是怎样实现的有一个清楚的认识，并且你会清楚Bitcoin的工作、运转以及安全原理。

&nbsp;&nbsp;&nbsp;&nbsp;事实证明，在数字技术中，中心化与去中心化概念竞争占有很重要的角色。为了理解Bitcoin的去中心化，我们将从一个观点开始——decentralization almost always is not all or nothing。几乎没有一个系统是完全去中心化或中心化的，一个很好的例子是Email。我会说Email从根本上是一个去中心化的系统。它基于一个标准分散协议——SMTP，但实际上受控于集权的网络服务提供商。这也许可以帮助我们理解Bitcoin中的工作原理。

&nbsp;&nbsp;&nbsp;&nbsp;让我们深入了解一些Bitcoin的去中心化的技术方面。我会将它分为至少5个不同的问题：

* Who maintains this ledger of transactions?
* Who has authority over which transactions are valid?
* Who creates new bitcoins?
* Who determines how the rules of the system change?
* How do bitcoins acquire exchange value?

&nbsp;&nbsp;&nbsp;&nbsp;这些都是Bitcoin protocol关于去中心化方面的成分，前三个问题是我们在这个lecture里将会思考的。接下来说到的比特币去中心化的内容都会围绕这三个问题。还需要强调的是在协议之外还有一些去中心化的方面，包括像Bitcoin exchanges（将Bitcoin转换成其他货币）、wallet software和其他一些服务提供商。因此，尽管底层的协议是去中心化的，在其上的服务也可能会是中心化的。

&nbsp;&nbsp;&nbsp;&nbsp;为了深入理解这一点，让我举出Bitcoin的基于去中心化的三个不同的方面：

Peer-to-peer network (may be the closest thing to purely decentralized):
&nbsp;&nbsp;&nbsp;&nbsp;open to anyone, low barrier to entry.

Mining:
&nbsp;&nbsp;&nbsp;&nbsp;open to anyone, but inevitable concentration of power often seen as undesirable

Updates to software:
&nbsp;&nbsp;&nbsp;&nbsp;core developers trusted by community, have great power.

&nbsp;&nbsp;&nbsp;&nbsp;其中Peer-to-peer network无疑是最接近纯去中心化的方面，虽然它也被网络状况所影响；对于开采bitcoin方面则受到了电脑资源的限制，而对于比特币软件则完全由其开发者所控制。

***********************************************

#### Segment 2.2 Distributed Consensus####

&nbsp;&nbsp;&nbsp;&nbsp;建立一个电子现金系统时，在技术层面，需要解决的主要问题：distributed consensus（分布式共识/一致性）。consensus protocols（共识协议）发展的传统方向是分布式系统中的可靠性问题。

&nbsp;&nbsp;&nbsp;&nbsp;类比ScroogeCoin中解决的double-spending问题，distributed consensus问题意味着当分布式系统中产生一个新的动作时，要么系统中所有的节点都记录一样的结果，要么都不记录，因为系统中某些节点可能会出现错误，当出现错误时，如果错误的节点不保存该动作而其他节点保存，将会出现错误。我们也可以想到，如果我们实现了一个distributed consensus protocol，我们可以用它来建立一个复杂的全局的分布式键-值存储（可以将任何键或名称映射到任意值），然后我们就可以实现很多的应用。像分布式DNS等。最酷的是BitCoin已经解决了这个问题。

&nbsp;&nbsp;&nbsp;&nbsp;distributed consensus的技术目的很简单，假设有常数n个节点或者进程，每个节点都有一些输入值。这时候distributed consensus协议就起作用了。协议需要满足两点。首先，协议是会终止（不会无限循环）的，并且所有正确的节点都要能够根据输入给出一致的值（因为有些节点的判断可能会是错误的甚至是恶意的）。而后，这个一致的值不能是任意的，而应该是至少一个正确节点的输出值。

![Peer-to-peer-network](/images/Peer-to-peer-network.jpg)

&nbsp;&nbsp;&nbsp;&nbsp;让我们在Bitcoin环境下看看它意味着什么。为了理解distributed consensus在Bitcoin中是怎样工作的，首先提醒一下Bitcoin是一个peer-to-peer（对等网络）系统。这就意味着当Alice想要付钱给Bob，她需要将这个交易广播给所有对等网络中的节点。你可以从图中看到这个交易的结构，它和我们在lecture 1中的GoofyCoin中看到的是相似的。一个交易需要有Alice的签名来证明交易的正确性，并且需要有Bob的公钥来表示这是支付给Bob的，结构中也含有一个哈希指针指向该比特币之前的交易信息。她将会将这个数据结构广播给所有的对等节点。但是，此时Bob的电脑并没有在网络中，如果Bob想要知道这个交易发生了（有人向他支付了钱），他也许需要在该对等网络中运行一个节点来监听交易信息。事实上，如果只是要接收这笔钱的话，是不需要监听的。这些钱无论他是否在该网络中运行了一个节点都会是他的。

&nbsp;&nbsp;&nbsp;&nbsp;所以，给定对等系统，节点可能想要达成的共识是什么？因为一个系统中可能会有大量的用户在同一时间广播他们的交易，我们需要达成共识的是，已经发生的交易和交易的顺序。它具体意味这什么呢？在Bitcoin中为了达成共识，在任何给定的时间里，所有的对等网络中的节点都保存了一个已经达成共识了的交易数据块序列，类似于ScroogeCoin中的transaction block chain。（我们可以一个一个交易地达成共识，但那很没有效率，所以我们以区块位单位进行）并且每个节点会有一组它所监听到的未完成交易的集合，对于这些交易尚未产生共识。根据定义，每个节点可能有一组略有不同的版本的未完成交易的集合。因为对等网络不是完美的，可能一些节点听说了一个交易，而另外的节点没有。

![consensus-work](/images/consensus-work.jpg)

&nbsp;&nbsp;&nbsp;&nbsp;根据这个设定，我们有了一串每个人都承认的块序列，每个区块都是一系列交易（灰色部分）。如图，现在我们假设系统中有三个节点。每个节点都提议，每个节点都有一个输入也有它所听到的未完成交易集合，它们一起执行consensus protocols，为了使共识协议得以成功，我们可以选择任易有效的区块加入block chain，哪怕它只是由一个节点提出的，（对于一个正确的交易块，其中的每个交易必须是合法地（有正确的数字签名等等））共识协议仍不会有问题。如果某些交易没有成功进入为一致性协议选出结果的特定区块，它可以等到下一个区块再加入。

&nbsp;&nbsp;&nbsp;&nbsp;所以可能绿色的区块被选择了，现在将它加入已共识的区块链，然后协议继续并路由。所以如果你有一个分布式共识的传统理论，并将其应用于bitcoin，这将某种程度是你可能最后得到的系统。现在这和比特币工作有一些相似，但不是比特币真正的工作方式。原因很简单，这么做因为各种原因是有技术上的硬伤的，很明显的原因是：节点可能会崩溃，一些节点可能就是恶意的，但也有网络不是很完美的原因（它是一个点对点系统，不是所有节点都彼此相连、存在网络错误等等导致的大量延迟，大量延迟的主要原因也在于没有一个全局时间的概念，节点不能简单地通过时间戳达成共识）。

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

&nbsp;&nbsp;&nbsp;&nbsp;因此，比特币有哪些事情是不同的呢？首先，它引入了激励的概念（在货币系统环境下适用，但不适合普遍的分布式系统）。其他的不同之处是它确实拥抱了随机的概念。意味着，它从一个有特殊概念的起点开始，最终却达成了一致性，而不是在一个很长的时间段保持一致性，例如现实系统中的一小时。但是，即使在最终，你也不能百分之百保证你感兴趣的交易或区块已经进入了一直的区块链。随着时间发展，你的一致性的可能性越来越高，这种交易出错的假设的可能性成指数下降。这就是比特币能给的固定概率保证，也是它能绕开传统分布式系统不可解问题的原因。

*********************************************************

#### Segment 2.3 Consensus without Identity: the Block Chain####

&nbsp;&nbsp;&nbsp;&nbsp;所以，让我们深入了解比特币共识算法的一些技术细节，同时当我们看到这些，我们需要记住比特币所有的节点是在节点没有任何可持久化的长期身份标识情况下完成所有运作的（值得再次提醒的是，比特币和传统分布式算法的应用环境不同）。如果节点拥有长久的身份标识，事情会简单很多，我们可以通过对身份ID的一些要求简化操作，在安全方面也有更多的可行性。

&nbsp;&nbsp;&nbsp;&nbsp;那么为什么比特币没有身份标识呢？一个原因是如果你在一个点对点系统的去中心化模型中，在其中没有中心化的权威组织可以给予节点身份标识并验证他们不能任意创建新的节点，就容易出现女巫攻击（Sybil attack），女巫仅仅是恶意的对手任意制造的拷贝节点以使其看起来有很多的参与者，实际上，这些伪参与者都是被同一个对手所控制的。另外的原因是，使用假名是比特币的一个固有目标，即使能简单地给所有节点或者参与者创建身份标识，我们也没有必要这么做。所以，比特币没有给你很强的匿名保证，你的不同交易可能会被联系在一起，但是同时，没有人强迫你出示你真实生活的身份标识（像名字、IP之类的）才能参与点对点系统，在区块链中，这是一个重要属性。

&nbsp;&nbsp;&nbsp;&nbsp;所以，在这里我们可以取而代之做一个假设，并且我们需要对这个假设的可行性很有信心。之后我们会知道这个假设实际上是怎样完成的。这个很弱的假设是，我们将假设有这样的能力，无论何种方式，在系统中挑选一个随机节点。我们可以给他们令牌或者票据或者其他这类东西，然后允许我们晚一些挑选一个随机的令牌ID并访问这个人，进一步假设，当前，令牌的生成和分布式算法已经足够聪明，如果对手试图创建很多合规的节点，所有这些节点都仅会得到一个令牌并且对手无法复制这个能力。

&nbsp;&nbsp;&nbsp;&nbsp;现在让我们看看在随机选取节点和某种被称为隐式一致性的假设下，什么会成为可能。那么什么是隐式一致性呢？在每一轮中，都会有随机的节点提出要达成共识的区块，每一轮都对应区块链的不同的区块。现在某一轮中一个节点被神奇地选中了，这个节点将可以提议区块链中的下一个区块，这里没有一致性算法也没有投票，这个节点只是简单地单方面提议了区块链中的下个区块是什么，但是如果那个区块是恶意的呢？有一个这样的过程，其他节点将会接受或拒绝那个区块，如果它们接受了这个区块，它们将从这个区块开始扩展区块链，如果它们拒绝了这个区块，它们将会从上一个区块开始扩展区块链（由区块链的特殊性我们可以得知是从哪开始扩展的）。这就是比特币中整体共识算法看起来是如何工作的，但现在讨论的有些简化，简化的愿意是我们假定了一个可以说是魔术般的随机节点选取过程，但是除了这个简化，已经非常接近比特币的实际工作方式了。

Consensus algorithm（simplified）

1. New transactions are broadcast to all nodes
2. Each node collects new transactions into a block
3. In each round a random node gets to broadcast its block(提出下一个共识区块的建议)
4. Other nodes accept the block only if all transactions in it are valid (unspent, valid signatures)
5. Nodes express their acceptance of the block by including its hash in the next block they create

&nbsp;&nbsp;&nbsp;&nbsp;所以，无论何时当爱丽丝向鲍勃付款时，她将创建一个交易，她将向所有节点广播交易，任何一直监听网络的节点都收集着一个还未被加入到区块链的未完成交易列表。在某一时刻，这些节点中一个被随机地选中而可以提议下一个区块，它将所有监听到的未完成交易聚拢，并提议下一个区块，现在被选中的节点可能是诚实的，但它也可能是一个恶意节点，或者一个错误的节点提交了包含了非法交易的区块，非法区块是那些没有正确数字签名 或者有已经被消费的交易，换句话说，试图重复消费。所以如果这发生了，另外一些节点会表明它们对于该区块的接受或者拒绝，通过在下一个区块中包含该区块的哈希值表示接受该区块，或者忽略该区块，而包含该区块前一个它们认为合法的区块的哈希值。

![double-spending-success](/images/double-spending-success.png)

&nbsp;&nbsp;&nbsp;&nbsp;假设爱丽丝是一个恶意节点，由于底层密码是坚固的，爱丽丝不可能简单地偷走其他节点的货币。简单地拒绝为鲍勃服务也没有任何意义（鲍勃只需要等待其交易进入其他节点提出的区块就好），所以我们面临的问题主要是double-spending攻击。假设爱丽丝要向鲍勃购买一款软件。现在爱丽丝合法地支付给了鲍勃货币，鲍勃没有再进行确认，将软件给了爱丽丝，并假设有诚实的节点提出了包含这个交易的区块并加入了区块链（绿色）。但此时，让我们们假设下一个随机节点是爱丽丝控制的恶意节点，爱丽丝将忽略整个合法区块，并提议一个区块，将给鲍勃的货币重复消费给自己控制的节点（红色）。那么我们怎样才能知道这个double-spending攻击成功了还是失败了呢？这取决于最终是绿色还是红色的区块进入了长期性一致链，这是由**诚实节点总是扩展最终最长的合法分支**这个事实决定的。但是，此时正常与重复消费的区块在技术上来说都是合法的，如果之后重复消费的区块先被扩展而成为较长分支，那么它将成为最后的长期性一致链的可能性是很大的，如此的话double-spending攻击就成功了，而合理支付的区块成了孤儿区块。

![double-spending-fail](/images/double-spending-fail.png)

&nbsp;&nbsp;&nbsp;&nbsp;幸好，我们可以换个思路，当鲍勃接受爱丽丝的货币时没有鲁莽地马上认定交易成功，而是继续监听，等待合法分支上更多的区块被确认，这被称为零确认交易。当然此时也有可能发生重复交易攻击，如果重复交易成功了，鲍勃只要意识到合理交易的区块变成了孤儿区块就可以放弃交易从而保护自己。另一种情况，如果重复支付发生了，鲍勃关心的区块被证明被下一个产生的区块所扩展，因为诚实节点总是扩展它们看到的最长的合法的分支，获得越多的确认就有越高的可能性交易最终会进入长期共识链，如果鲍勃能等待更多的确认（在比特币生态系统中最普遍的是等待6个确认），那么重复支付的区块进入长期一致链的可能性就会指数下降，从而保证交易的合法性。大约6个确认之后，你几乎已经不可能弄错了。

***************************************************

#### Segment 2.4 Incentives and Proof of Work####

&nbsp;&nbsp;&nbsp;&nbsp;本节主要讲的是比特币中的工程激励。首先我们仍然需要假设我们能随机选取一个节点，并且有50%的概率选到一个诚实的节点。当然诚实的假设是很有问题的。我们的问题是，我们能给节点的诚实行为以激励吗？让我们看看我们之前谈过的共识问题。

![Incentives-for-honest-behave](/images/Incentives-for-honest-behave.png)

&nbsp;&nbsp;&nbsp;&nbsp;上方是一个长期共识链，下方的区块包含一个重复消费尝试。有人可能会问，我们可以用某种方式惩罚创造这个（重复消费）区块的节点吗？但由于一些原因这很成问题，包括节点没有身份标识，所以没有方法找到它们去惩罚它们。所以替代的，我们反过来问，我们可以奖励那些创造了所有这些最终进入长期共识链的区块的节点吗？然而，同样的，我们没有节点的身份标识，所以无法给向他们的家庭住址寄去现金或邮件。如果我们有某种可以用来激励他们的去中心化的电子货币，换句话说，我们要用比特币来激励创造了这些区块的节点。所以我们该怎么做呢？我们将要跳出分布式一致的模型，因为我们通过分布式共识完成的实际上是一种货币，我们将向它们支付一定单位的这种货币以激励这些节点。

&nbsp;&nbsp;&nbsp;&nbsp;在比特币中实际上有两种独立的激励机制。第一种我们称之为区块奖励。根据比特币的规则，节点每创建一个区块，都会将一个特别的交易包含进那个区块，这个特别的交易是一个创币交易，这个节点可以选择这个交易的接收地址，所以当然，那个节点一般都会选择属于它自己的地址。你可以把它看成一笔为将创建的区块放入共识链中的服务而支付的费用。实际上，这个创币交易有个有趣的属性，现在它被固定为25个比特币，但实际上它每四年会减半，我们现在在第二个时期。在比特币诞生之后4年中，是50个比特币，它将会持续减半。

&nbsp;&nbsp;&nbsp;&nbsp;基于我们已经说过的，节点可以得到区块奖励，无论其提议的区块是否只包含合法交易或有恶意行为，所以我们如何将作为激励的区块奖励真正地发放给诚实节点呢？让我们考虑另一个问题，节点如何很好地收集到它的奖励呢？只有在这个区块最终进入长期共识链这才会发生。因为那是唯一的创币交易会被视为有效的情况。这就是这里的激励机制。是一个非常巧妙的花招。它会激励节点以它们认为其他节点会认同的方式创建区块链中的下一个区块。

&nbsp;&nbsp;&nbsp;&nbsp;让我们回到诡异的减半现象。比特币将区块奖励作为唯一的创币方式，通过降低创建的比特币的数量从而可以降低比特币增长的速率，限制比特币的总供应量。这个新区块的创建奖励实际上是会结束的。所以，听起来十分怪异，因为这意味着系统将不会再激励诚实行为而变得不安全吗？答案并不是这样。因为这只是两个激励机制中的一个。还有另一种激励机制，被称为交易费。任意交易（而不是区块）的创建者可以选择使输出的币值小于输入值，根据比特币的规则，所有节点都将这种方法产生的差值理解为交易费。无论谁创建的区块首先将那个交易放入了区块链，都可以收获这笔交易费。所以，如果你是一个节点，创建了一个包含了假设200个交易的区块，这200笔交易的交易费都归你了。当然，这个交易纯粹是自愿的，就像小费。但是我们希望，基于我们对于系统的理解，因为区块奖励开始减少，交易费会变得越来越重要。以致将交易费放入他们的交易几乎成为节点提供合理服务的强制手段。某种程度上，这已经开始发生了。但这整个系统如何演化还没有被充分解决，成为了比特币中一个有趣的开放研究领域。

&nbsp;&nbsp;&nbsp;&nbsp;所以现在我们已经理解了节点创建区块是如何被激励表现诚实或者遵守协议。所以如果我们解决了剩下的一些问题，我们将对比特币如何成功地去中心化有一个真正的好的理解。

Remaining problems

1. How to pick a random node?
2. How to avoid a free-for-all due to rewards?
3. How to prevent Sybil attacks?

&nbsp;&nbsp;&nbsp;&nbsp;结果是这些问题是相关的，它们都有相同的解决方法。这个解决方法被称为**工作量证明（Proof of work）**。

Proof of work

&nbsp;&nbsp;&nbsp;&nbsp;To approximate selecting a random node:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;selects nodes in proportion to a resourse that no one can monopollize (we hope)

* In proportion to computing power: proof-of-work
* In proportion to ownership: proof-of-stake

&nbsp;&nbsp;&nbsp;&nbsp;工作量证明的关键想法是，我们不是随机地选择一个节点，而是做一些不同的事情，那就是通过以某种无人可独占的资源为依据，近似选取一个随机节点。如果那个我们谈到的资源是计算能力，那么这就是工作量证明，我们依据它们的计算能力选择节点。我们也可以依据所拥有的货币量，它被称为权益证明。我们将在后面看到它。让我们试图给出关于工作量证明的更好的定义：根据它们的计算能力选择节点，另外一种理解方法是，我们允许节点和其它节点通过它们的计算能力竞争，这将导致节点们被依据计算能力比例自动选出。

![hash-puzzles](/images/hash-puzzles.png)

&nbsp;&nbsp;&nbsp;&nbsp;让我们看看什么是真正在比特币中使用工作量证明系统。要提到的是Hash puzzles的问题，为了创建区块，提议区块的节点被要求寻找一个随机数 nonce ，这样，当你将随机数 nonce、前一个区块的哈希值 prev_hash 和组成该区块的交易列表放在一起进行哈希，得到一个长字符串。然后这个输出的哈希值需要是一个很小的数字，以能够装入一个很小的目标空间。目标空间很小是相对于哈希函数的输出空间非常巨大而言的。这个想法实际上是我们要通过哈希包括随机数的整个区块生成一个特定的输出来让寻找一个满足条件的随机数 nonce 的过程适度困难一些。如果哈希函数是安全的，唯一能成功解决Hash puzzles的方法就只能是不断地尝试足够多的随机数 nonce 直到找到一个满足条件的随机数。如果目标空间是所有输出空间的1%，你将需要尝试100个随机数，因为如果这个哈希函数确实表现得随机，100个随机数中将有一个会满足条件。事实上目标空间远远小于所有输出空间的1%。本质上这是一个计算问题，节点被要求解决这个问题以制造区块。

&nbsp;&nbsp;&nbsp;&nbsp;Hash puzzles的概念和工作量证明取代了要求某人以某种方式选取一个随机节点的方法，所有节点同时独立地解决Hash puzzles，它们中会有不断地出现一个个找到满足条件的随机数 nonce 的幸运节点，这个节点就可以提议下一个区块。这就是它如何完全地去中心化。让我们再多看一些细节，Hash puzzles工作量证明有三个基本特征。

1. 首先，它需要相当难以计算出来，大约每个区块你需要计算10的20次方个哈希，因为这，只有一些会专注于区块创建过程的节点，被称为比特币矿工。
2. 其次，我们要这个计算成本能参数化，它不是一个所有时间都固定的成本，完成这个的方法是，在比特币点对点网络中的所有节点都将自动重算目标值（目标值是目标空间占输出空间的比值大小），每两周重算一次，并且它们将会保证任何两个节点成功的时间间隔的平均值为不变量，大概为十分钟。这意味着，如果你是一个矿工，你在比特币挖矿中投资了一个固定数量的硬件，同时整个挖矿生态在增长，更多的矿工在进入，或者他们在部署越来越快的硬件，区块的平均创建速度增快，此时节点们将会自动地调节目标值，以增加节点创建区块所需的工作总量。所以如果你投入的硬件投资是固定数量的，你创建区块的速率实际上取决于其他矿工在干什么（你的计算能力与全网的计算能力的比值）。维持创建频率为十分钟的理由很简单，如果区块们产生的时间间隔过小，就会降低效率，我们将会失去能将很多交易放入区块的优化收益。（这个规则下会自动保证50%的概率是诚实节点）
3. 最后，工作量证明的第三个重要特性，就是验证的简单性。另一个节点证明工作量证明的正确性应该是很简单的。这意味着当一个节点尝试了10的20次方次之后找到了一个随机数 nonce 使其满足Hash puzzles之后，必须将 nonce 作为区块的一部分公开，这样，对于任何其他看到区块内容的节点，都可以很简单地将它们放在一起哈希，验证其输出结果小于目标值。这让我摆脱中心化的权威验证，任何其他的矿工或节点都可以验证区块的正确性，以保证每个矿工都投入了很多计算能力来创建区块。

*************************************************************

#### Segment 2.5 Putting It All Together####

&nbsp;&nbsp;&nbsp;&nbsp;现在让我们看看 mining economics。我们已经说了挖矿的成本已经相当昂贵，因为创建一个区块需要10的20方次哈希，但同时，我们也需要看到区块奖励是25个比特币，这是很大一笔钱，因此，对于一个挖矿的矿工，确实要绞尽脑汁思考一个问题，挖矿是否赚钱。为了回答这个问题，我们来简单考虑一下挖矿成本和收入。矿工挖矿的成本主要在于硬件投入和电力成本（实际上，根据其消耗的电量，比特币挖矿是很昂贵的，这部分是成本中极为显著的一部分，因此不能只考虑硬件投入），主要收入在于区块奖励和交易费用。如果收入高于成本则矿工获利，如果低于，则矿工亏损。但其中也有复杂的部分，首先电力成本是随时间变化的，并且矿工创建区块的速率取决于其硬件计算能力与网络整体的计算能力的比值，这使得估计变得更加复杂。并且，因为比特币也是一种货币，那么该估计也要考虑比特币的汇率。所以，估计矿工不同挖矿策略的的获利情况是很复杂的问题。

&nbsp;&nbsp;&nbsp;&nbsp;既然我们对比特币的去中心化策略有了不错的了解，让我们把它们都放在一起并做一个小小的回顾，并进行更高层次的思考以得到更好的理解。我们将用针对性的方法了解比特币中所渗透的分布式共识的概念。在传统货币中，共识只在有限的范围内起作用，那就是在交换货币的相关方之间有共识存在。在比特币中的共识比在传统货币中更加深入。在比特币中，共识是由区块链维护的，它是一个有效交易，或者已完成交易的记录。所以即使你拥有多少比特币是一个客观的共识，当我说我拥有一定数量的比特币时，我的意思是比特币点对点网络区块链中的记录认为我的所有地址上拥有特定数目的比特币。这可以说是比特币意义上的最终的事实描述。最终我们需要系统规则的一致性，因为系统规则有时必须发生变化，会发生叫做软分叉和硬分叉的事情。

&nbsp;&nbsp;&nbsp;&nbsp;奇妙的是，我们所讨论的比特币的三个主要方面是相互依赖关联的：

![bitcoin-bootstrapped](/images/bitcoin-bootstrapped.png)

&nbsp;&nbsp;&nbsp;&nbsp;比特币系统需要的所有这些特征以相互依存的方式发展。

&nbsp;&nbsp;&nbsp;&nbsp;让我们考虑一下如果有攻击者控制了51%的节点即控制了共识过程，他能做到什么？显然，由于底层密码的存在他要想偷取货币只能创建非法的区块和交易，当其他诚实节点看到这个非法分支（即便是最长链）也会忽略该最长链，因为它是非法的，所以攻击者无法从已存在的地址偷取比特币。再想到阻止某节点的交易，显然攻击者可以做到从区块链的角度阻止包含交易的区块进入区块链，但却无法阻止交易沿着点对点网络广播到所有节点。总而言之，比特币目前面临的主要威胁是人们对其失去信心。