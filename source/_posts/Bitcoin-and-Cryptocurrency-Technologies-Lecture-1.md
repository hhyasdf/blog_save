---
title: Bitcoin and Cryptocurrency Technologies - Lecture 1
date: 2016-11-28
categories: Bitcoin and Cryptocurrency Technologies
tag: Bitcoin, Cryptocurrency Technologies
---

### Introduction to Crypto and Cryptocurrencies###

***********************************************

#### Segment 1.1 Cryptographic Hash Functions####

Hash fuctions：

1. 一个加密哈哈希函数可以将任何字符串当作输入（包括任何长度）。
2. 产生一个固定长度的输出（这里产生的是bitcoin中用到的256位）。
3. 可被高效计算的（在有效的时间内可以计算出输出）。

Security properties：

1. collision-free（无冲突）：maybe impossible to find a collision by a regulaer people with a regular computer
2. hiding：Given H(x), it’s infeasible to find x.(x has to be chosen from a “very spread out” set).
3. puzzle-friendly：for any possible output value y that you mught want from the hash function, while k is chosen from some set that’s very spread out, then there is no way to find an x, such that the hash of k and x(H(k|x)) is equal to y.

&nbsp;&nbsp;&nbsp;&nbsp;如果我们得到了一个collision-free的哈希函数，我们可以假设当H(x)=H(y)时，x=y，反之亦然（作为**信息摘要**）.

    Given a "puzzle ID" id(from high min-entropy distrb.), and a target set Y:  
    Try to find a "solution" x such that H(id | x) belong to Y. 

Puzzle-friendly property implies that no solving strategy is much better than trying random valuas of x.

比特币用到的加密哈希函数叫做：**SHA-256**:

&nbsp;&nbsp;&nbsp;&nbsp;首先读取需要哈希的信息，将其分解成若干个512位的块。当然，一个信息不一定可以正好分成若干个512位的块，于是我们在信息后面加上一些填充物，在填充物的最后面的部分是64位的长度域，用来记录信息位数的长度。在长度域前面是一个由1（位）加若干个0（位）的填充，你可以选择0的个数来将信息填充到512位的整数倍。填充之后就可以分解为若干个块开始计算了。先从标准文档中找到一个256位的数IV，然后将IV和第一个信息块整个768位用函数C进行计算，压缩函数C将输出一个256位的结果，然后继续取下一个512位的块，再用函数C计算，keep going直到结束。可以简单地看出，如果函数c是collision-free，那么整个哈希函数也将是collision-free的。如下图：

![SHA-256](/images/Bitcoin/SHA-256.jpg)

************************************************

#### Segment 1.2 Hash Pointers and Data Structures####

hash pointer is:

* pointer to where some info is stored, and (cryptographic) hash of the info

if we have a hash pointer, we can

* ask to get the info back, and verify that it isn’t change

&nbsp;&nbsp;&nbsp;&nbsp;我们可以用hash pointer构造数据结构，比如说链表：

![hash-pointer-link-list](/images/Bitcoin/hash-pointer-link-list.jpg)

&nbsp;&nbsp;&nbsp;&nbsp;这就是我们叫做block chain（区块链）的数据结构（将普通的指针换成哈希指针，哈希指针中的哈希值是它所指向的节点（包括数据和哈希指针）的哈希值）。区块链第一种应用场景是作为防篡改日志，比如我们可以建立一个日志数据结构储存信息，我们将数据添加到它的末尾，如果后来的人将之前的日志数据篡改了，我们可以察觉到（无论其修改哪一个节点，指向该节点的哈希指针和它的哈希值都会不符合，除非将该节点之前的所有节点全部更改，包括头指针，然而这是不可能的，头指针是我们要记住的）。

&nbsp;&nbsp;&nbsp;&nbsp;另一个我们可以用哈希指针建立的有用的数据结构是二叉树：

![hash-pointer-binary-tree](/images/Bitcoin/hash-pointer-binary-tree.jpg)

&nbsp;&nbsp;&nbsp;&nbsp;这种二叉树又叫做Merkle tree（梅克尔树），它的结构特点为，我们将一些数据放在结构的底部（叶子节点），叶子节点的上层节点为一个含有两个哈希指针的节点，分别指向它的两个儿子，每一层相似地向上推进直到根节点，每一层节点都有两个哈希指针指向它的两个儿子。当然我们记住树的头指针（指向根节点的哈希指针）。如果我们从头指针开始读到表中的任意节点（数据），我们可以保证数据没有被篡改过。原理跟block chain差不多。

&nbsp;&nbsp;&nbsp;&nbsp;不像block chain，Merkle tree的另外一个好的特性是，如果有人要向我们证明一个特定数据块属于这棵梅克尔树，他们所要做的只是提供给我们如下图的数据（从上至下依次确定，最后我们可以确定数据）。

![hash-pointer-binary-tree-prove](/images/Bitcoin/hash-pointer-binary-tree-prove.jpg)

&nbsp;&nbsp;&nbsp;&nbsp;整个确认的过程需要提供log n个数据，时间复杂度为log n。因此，再含有大量数据的Merkle tree中，我们仍然可以花费较短的时间确认数据的成员关系。

&nbsp;&nbsp;&nbsp;&nbsp;因此，Merkle tree有很多优点，一个是其中有很多元素但我们只需要记住一个只有256位的根节点的哈希值。另一个是我们可以在对数的时间和空间内确定数据成员关系。并且，Merkle tree可以变为有序Merkle tree（我们用任意指定的规则顺序排序），在这个变体中，我们是可能确认数据非成员关系的，我们可以简单指出到该元素之前应该是哪个元素和之后是哪个元素，如果它们之间不存在元素，则该数据不可能在树中。

&nbsp;&nbsp;&nbsp;&nbsp;更多的，我们可以将哈希指针用在任何没有回路的、基于指针的数据结构（如果结构中有回路，那么就不能让所有的哈希值一一对应）。

***********************************************************

#### Segment 1.3 Digital Signatures####

&nbsp;&nbsp;&nbsp;&nbsp;数字签名和写在纸上的签名一样，只不过是数字的形式

What we want from signatures

* Only you can sign, but anyone can verify.
* Signature is tied to a particular document(can’t be cut-and-pasted to another doc)

&nbsp;&nbsp;&nbsp;&nbsp;首先根据特定的keysize生成一个secret signing key（私钥）和一个public verification key（公钥），然后使用私钥对一段特定数据进行“签名”生成一个数字签名。然后可以使用公钥对数据、数字签名进行确认。生成公钥、私钥和数字签名的过程可以使用随机算法（类似于不同的人应该有他们不同的签名），而确认的步骤不能，它只能是确定的。

Requirements for signatures：

* “valid signatures verify”: verify(pk, message, sign(sk,message)) == true.
* “can’t forge signatures”无法伪造签名: 一个知道公钥和可以看到他选择的消息上的签名的人，不能在其它未签名消息上产生一个可以用同样公钥确认的签名。

Pratical stuff…

* algorithms are randomized: need good source of randomness.
* limit on massege size: use Hash(message) rather than message.
* fun trick: sign a hash pointer and the signature “cover” the whole structure.

&nbsp;&nbsp;&nbsp;&nbsp;比特币用到的数字签名方案叫做ECDSA（Elliptic Curve Digital Signature Algorithm）。良好的随机性对ECDSA尤为重要。ECDSA的一个怪癖是，尽管你在生成签名时使用了你的完美私钥但key没有较好的随机性，你的私钥仍然会被泄漏。（在两次签名中使用相同的key、key可预测、甚至泄漏一些key的字节都可能会泄漏私钥http://zhiqiang.org/blog/it/das-and-ecdsa-rsa.html）

************************************************************

#### Segment 1.4 Public keys as Identities####

Useful trick: public key == an identity

&nbsp;&nbsp;&nbsp;&nbsp;if you see **sig** such that verify(pk, msg, sig)==true, think of it as
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**pk says, “[msg]”**
&nbsp;&nbsp;&nbsp;&nbsp;to “speak for” pk, you must know mathching secret key **sk**

&nbsp;&nbsp;&nbsp;&nbsp;当将一个公钥当作一个身份标识时，创建一个新的随机的公钥-私钥对，即创建了一个新的身份。公钥即是你能使用的公共的“名字”，你可以通过私钥来用这个“名字”说话。也因为只有你知道私钥，你可以控制这个身份。这给了我们关于**分散身份管理（decentralized identity management）**的灵感，我们不需要一个固定地去注册身份的地方，也不需要获得一个用户名，你也不需要通知谁你要用一个特定的名字，如果你想要一个新的身份，自己做一个就好了。任何人可以在任何时候生成任意数量的身份。这就是Bitcoin实际管理身份（identity）的方式，这个身份在Bitcoin行话中叫addresses（地址）。一个身份即是一个公钥或一个公钥的hash。

Privacy

&nbsp;&nbsp;&nbsp;&nbsp;Addresses not directly connected to real-world identity.
&nbsp;&nbsp;&nbsp;&nbsp;But observer can link together an address’s activity over time, make inferences.

************************************************************

#### Segment 1.5 A Simple Cryptocurrency(加密货币)####

&nbsp;&nbsp;&nbsp;&nbsp;在这个部分，我们会讨论并介绍一些简单的加密货币例子。

![GoofyCoin](/images/Bitcoin/GoofyCoin.jpg)

&nbsp;&nbsp;&nbsp;&nbsp;GoofyCoin大概是我们能想象的最简单的加密货币。它只有一些简单的规则。

![Goofy-make-coins](/images/Bitcoin/Goofy-make-coins.jpg)

&nbsp;&nbsp;&nbsp;&nbsp;第一条规则是Goofy可以创建新的货币（coins）。Goofy可以在任何时间创建货币（由图中类似数据结构表示），当他创建货币时，货币属于他（Goofy会用他的私钥对货币签名，任何人都可以确认）。CreateCoin是一个操作，uniqueCoinID是由Goofy产生的货币ID。

![Goofy-pass-coins](/images/Bitcoin/Goofy-pass-coins.jpg)

&nbsp;&nbsp;&nbsp;&nbsp;第二条规则是任何拥有货币的人可以将货币传递给另外一个人。现在我们取一个hash pointer指向之前Goofy产生的货币，然后我们创建一个声明（statement）：Goofy将把货币支付给Alice（某个公钥）。因为Goofy是货币的拥有者，Goofy必须在对该货币的每一项交易上签字。一旦上述事件发生之后，Alice就拥有了这个货币。Alice可以证明她拥有了这个货币，因为她可以给出这个由Goofy合法签名的指向Goofy合法拥有的货币的数据结构。所以在这个系统中，货币的正确性可以由货币自己证明。现在Alice也可以使用这个货币了。

![Alice-pass-coins](/images/Bitcoin/Alice-pass-coins.jpg)

&nbsp;&nbsp;&nbsp;&nbsp;这就是GoofyCoin的所有规则。Goofy可以通过简单地分配一个声明，他在用一个惟一的货币ID创建新的货币。然后无论谁拥有了该货币都可以通过签署一份传递的声明的方式，将它传递给另外一个人。你可以简单地沿着这条数据链通过确认所有签名的方式，来确认货币的合法性。

![Goofy-secure-problem](/images/Bitcoin/Goofy-secure-problem.jpg)

&nbsp;&nbsp;&nbsp;&nbsp;但是，GoofyCoin中存在一个严重的安全问题，我们可以从上图中看到。Alice在将货币支付给Bob之后又发表了一份她将同一个货币支付给了Chuck的声明。如果Chuck不知道左边发生的事情（Alice把货币给了Bob但没有告诉Chuck），他会认为一切都是合法的，他合法地拥有了这个货币。此时Chuck和Bob看起来都有效地拥有了这个货币。但货币是不允许这样工作的。这被叫做double-spending attack，因为Alice将同一个货币支付了两次。double-spending attack是加密货币系统需要解决的主要问题之一，GoofyCoin并没有解决这个问题，因此它是不安全的。也正因为GoofyCoin允许double-spending，它不能被称之为加密货币。

&nbsp;&nbsp;&nbsp;&nbsp;因此为了建立一套可行的加密货币，我们需要一些double-spending问题的解决方案。事实上double-spending问题是我们在设计加密货币中面临的主要挑战。因此我们需要以某种方式改进GoofyCoin。这样，我们可以设计出另一套货币，我将它叫做ScroogeCoin。

![ScroogeCoin](/images/Bitcoin/ScroogeCoin.jpg)

&nbsp;&nbsp;&nbsp;&nbsp;ScroogeCoin将会更像GoofyCoin，除开它能用一种特别的方法解决double-spending问题。这种货币是被Scrooge创造的。

![ScroogeCoin-BlockChain](/images/Bitcoin/ScroogeCoin-BlockChain.jpg)

&nbsp;&nbsp;&nbsp;&nbsp;ScroogeCoin的数据结构会更加复杂，但它的中心观点之一是公开发表所有已经发生的交易的历史，交易历史将存储在一条区块链（block chain）中，并由Scrooge数字签名。每一次交易都有它的交易ID（根据交易顺序），链中每个元素的hash pointer 在它之前的一次交易。Scrooge会持有（并维护）代表整个数据结构的hash pointer，在其上签名并发表。然后所有人都可以确定Scrooge在这个hash pointer上签了名，并可以沿着整条链查看所有的ScroogeCoin的交易历史。虽然在这里我们将一个交易放在一个区块上，但这是我们为了解释而简化了操作，但是在实践中，为了优化，我们会将多个交易放进一个区块里。就像BitCoin一样。

&nbsp;&nbsp;&nbsp;&nbsp;这样，Scrooge发表了交易历史，然而交易历史又有什么用处呢？我们可以用交易历史检测double-spending，因为假设Alice拥有了一个货币，她将把货币支付给Bob，在这之后她又试图将同一个货币支付给Charlie，Charlie会发现事情出现了问题，因为Charlie可以查看交易历史，并发现Alice已经将货币支付给了Bob（事实上所有人都会知道Alice将其支付给了Bob）。然后Charlie（所有人包括Scrooge）会拒绝交易并意识到他们不能相信Alice。

![Scrooge-CreateCoins](/images/Bitcoin/Scrooge-CreateCoins.jpg)

&nbsp;&nbsp;&nbsp;&nbsp;因此，在ScroogeCoin中有两种交易。第一种交易是CreateCoins transaction，它创建新的货币（就像Goofy在GoofyCoin中创建货币一样），但在这里我们会允许一次交易中创建多个货币。就像上图中看到的一样。本次的交易ID为73,类型是CreateCoins，然后下方是一个被创建的货币的列表。每一个货币都有连续的标号像1、2…… 每个货币都有一个value，它价值一定的ScroogeCoins，而后每个货币还有一个接收者（货币被创建时接收的人的公钥）。这样，这种交易创建了一些新的货币并将它们分配给了最初的所有者。现在我们可以提出ScroogeCoin中货币ID的概念了。它代表了一个在特定的CreateCoins交易中被创建的具有特定number的货币。所以，ScroogeCoin中的货币也有了自己的ID。

&nbsp;&nbsp;&nbsp;&nbsp;一个CreateCoins交易永远是合法的，because Scrooge said so，Scrooge总会将它放进交易历史里，所以我们永远不要担心Scrooge是否有资格创建货币。

![Scrooge-PayCoins](/images/Bitcoin/Scrooge-PayCoins.jpg)

&nbsp;&nbsp;&nbsp;&nbsp;ScroogeCoin中的第二种交易是PayCoins trasaction。在这种交易中会消费（摧毁）货币、重新创建总价值相同但属于不同的人的货币。上图的左边是一个PayCoins的样例。交易ID是73,类型是PayCoins，这里我们有一份它消费的货币的列表，在这次交易中所有的这些货币都被消费并摧毁了。因此我们需要添加与所有被消费货币总价值等价的货币，在消费列表的下方列出的就是我们创建的货币（与在CreateCoins交易中的含义一样）。在最低部有数字签名，这交易需要被所有支付了货币的人签署。如果你是其中需要支付的货币的所有者，你需要在其上数字签名（如果你拥有两个货币，你需要签署两次），表示你同意了此次消费。只有（图中的）四个条件都满足，它才是一次合法的交易，才会被Scrooge写入交易历史。

&nbsp;&nbsp;&nbsp;&nbsp;在整个方案中需要提到的是，货币是不变的。货币不会被划分或合并，它们只会在一次交易中创建，之后在另一次消费中被消费（销毁）。但你可以用交易达到相同的效果，比如说消费一个货币并支付给自己总价值相同的两个货币。

&nbsp;&nbsp;&nbsp;&nbsp;但在ScroogeCoin中也存在了一个巨大问题：Scrooge本身。Scrooge的行为是无人监管的，当Scrooge开始misbehaving或停止工作，系统将会瘫痪。所以我们最大的问题是Centralization（系统集中）。虽然Scrooge可能会喜欢这种系统，但其他用户也许不会喜欢。所以，改善ScroogeCoin的主要技术挑战在于我们是否能去除集中的Scrooge控制。我们是否能找到像ScroogeCoin一样好的，但没有中央信任权威的货币系统呢？在这样的系统中也就意味着每个人都可以认同一个维护着所有交易历史的block chain。我们需要找到让每一个人都判断一次交易是否合法和是否已经发生的方法。我们也需要知道如何用一种非集中的方式分配ID。当然，BitCoin解决了所有的这些问题。





















