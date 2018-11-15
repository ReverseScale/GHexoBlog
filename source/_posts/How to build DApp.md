---
title: 如何用 Swift 打造你的第一个区块链 App
layout: post
date: 2018-07-06 22:51:21
comments: true
categories:
	- Blockchain
keywords: 区块链
tags:
	- Blockchain
description: 

---
区块链(Blockchain) 是一种突破性技术(Disruptive Technologies)，近年渐获关注，号称将互联网从信息共享，推向价值传递，让我们一步一步的来探索它设计上的奇思妙想~                                                                                                                                                       
<!-- more -->
------

因为区块链是许多加密货币(Cryptocurrencies) 如比特币(Bitcoin)、以太坊(Ethereum)、莱特币(Litecoin) 的创始技术。那区块链是如何运作的呢？在本次的教学里，我将会谈到所有关于区块链技术的知识，以及如何用Swift 来制作自己的「区块链」。那么，让我们开始吧！

![](https://user-gold-cdn.xitu.io/2018/7/10/164819b1f9297878?w=1856&h=832&f=png&s=1551258)

## 区块链的运作

顾名思义，区块链就是一个由不同的区块串连在一起的「链」，每个区块包含三则资讯：资料(Data)、杂凑值(Hash)、和前一个区块的杂凑值。

1. 资料 ──依据使用情境，储存在区块的资料会因区块链的类型而不同。例如，在比特币区块链中，储存资料就是与交易有关的资讯，即是转帐的金额、以及参与交易二人的资讯。
2. 杂凑值 ──你可以把杂凑想成是一种数位指纹，它是用来识别一个区块及其资料。杂凑最重要的地方，就是它是一个独一无二的字母数字(Alphanumeric)程式码，通常会由64个字元组成。当一个区块被创造时，杂凑值也同时产生。当一个区块被更动时，杂凑值也会同时被更动。透过这种方式，当你想要查看区块上的任何变动时，杂凑值就非常重要。
3. 前一个区块的杂凑值 ──每个区块就是藉由储存前个区块的杂凑来链结在一起，组成一个区块链！这就是让区块链如此安全的原因。
看看这张图片：

![](https://user-gold-cdn.xitu.io/2018/7/6/1646d0fd1cf4ed02?w=2050&h=790&f=png&s=25544)

如你所见，每个区块由资料（未显示）、杂凑值、和前一个区块杂凑值所组成。举个例子，黄色区块包含了自己的杂凑值H7S6、和红色区块的杂凑值8SD9。以这种方式，它们组成了一个链结，并以此连结在一起。现在，假设有个骇客入侵并尝试更动红色区块。请记住，每次一个区块以任何方式被更动时，区块的杂值凑也会被更动！因此，当下一个区块执行确认、并看到前一个区块的杂凑值并不吻合时，它会有效地将自己从「链」中分离出来，而不会被骇客读取。

这就是区块练会如此安全的原因，你几乎不可能尝试回溯并更改任何资料。杂凑值提供了不错的保密及隐私，但还有两个安全机制来让区块链更加安全：验证(Proof-of-work)及智慧合约(Smart Contract)。虽然我不会详述，但你可以在这里了解更多。
区块链最后一个保全方式就是基于它的位置。与大多储存在伺服器或是资料库内的资料不同，区块链使用点对点网路(Peer-To-Peer, P2P)，它是一种网路型态，允许任何人加入并将该网路上的资料分发给每位接收者。
每当有人加入这个网路时，他们就会得到区块链的完整拷贝；每当有人建立一个新区块时，就会传送到网路内的所有人。然后，透过一些复杂的程式，让节点(Node)在加入这个区块到区块链之前，先确认该区块有否被窜改。就这样，任何地方的任何人都可以取得这些资讯。如果你是HBO矽谷群瞎传(Silicon Valley)的忠实粉丝，这听起来可能有点熟悉，因为在这出美剧里，主角就是用了类似的技术来建立一个全新网路。

因为每个人或节点都有一份区块链的拷贝，他们可以有共识并确认哪些区块是合法的。因此，如果你想要骇入一个区块，你必须骇入网路上超过50% 的区块来通过你的资讯。这就是为什么区块链或许是过去十年以来最安全的技术之一。

### 关于范例应用

现在你了解区块链是如何运作了，那么开始来制作我们的范例App吧！请先下载初始项目。
（https://raw.githubusercontent.com/appcoda/BlockchainDemo/master/BlockchainStarter.zip）

如你所见，我们有两个比特币钱包。第一个帐号Account 1065 拥有500 BTC，而第二个帐号0217 则什么都没有。我们利用传送按钮来传送比特币到其他帐号。为了赚取BTC，我们按下Mine 按钮就可以获得50 BTC 为奖励。基本来说，我们做的是当App 执行时，利用控制台来观察两个比特币帐号间的交易情况。

![](https://user-gold-cdn.xitu.io/2018/7/6/1646d11eca5d2f5f?w=1440&h=758&f=png&s=63032)

你会注意到，侧边栏里有两个重要的类别：Block及Blockchain。打开这些档案，你会看到档案是空的，那是因为我将会引导你写出这些类别的逻辑。那我们开始吧！

![](https://user-gold-cdn.xitu.io/2018/7/6/1646d123d37487fa?w=1398&h=729&f=png&s=59158)

### 在Swift 定义Block

前往Block.swift并添加程式码以定义一个区块。首先，让我们了解一下区块是什么。我们先前定义了一个区块由三个部分组成：杂凑值、记录的实际资料、以及前一个区块的杂凑值。当我们想建立自己的区块链时，必须要知道区块的排序，这一点可以很容易地在Swift中定义。添加以下程式码到类别里：

```swift
var hash: String!
var data: String!
var previousHash: String!
var index: Int!
```

现在，我们需要添加最后一个重要的程式码。我之前提到每次一个区块被更动，杂凑就会改变；这就是区块链如此安全的特色之一。所以我们必须建立一个函式来产生一个随机字母数字的杂凑。这个函式只需要几行程式码：

```swift
func generateHash() -> String {
    return NSUUID().uuidString.replacingOccurrences(of: "-", with: "")
}
```

NSUUID是一个物件，代表桥接UUID的通用唯一值。它内置于Swift中，而且非常适合用来产生32字元的字串。这个函式产生一个UUID，消除所有连字符(-)，然后回传String，即区块的杂凑。Block.swift现在应该看起来像这样：


![](https://user-gold-cdn.xitu.io/2018/7/6/1646d13a8baf7a67?w=1440&h=761&f=png&s=56820)

我们已经定义了Block类别，接着让我们定义Blockchain类别吧。切换到Blockchain.swift。

### 在Swift 定义Blockchain

如前文所说，让我们尝试从基本面来了解区块链。在基本的术语里，区块链只是区块串在一起组成的链；换句话说，它是一个包含所有项目的列表。听起来是不是有点熟悉呢？因为这就是阵列的定义，而这个阵列就是由区块所组成！让我们把下列的程式码加进去吧：

```swift
var chain = [Block]()
```

> 小提示：这几乎可以应用在电脑科学世界的所有事上。如果你曾遇过大问题，试着将它拆解成小组件，然后以自己的方法来解决问题；就像我们弄清楚如何在Swift中加入区块及区块链一样！

你会注意到在阵列里面包含了先前定义的Block类别，那是我们在区块链中需要的所有变数。加入两个函式到类别里，我们就完成了。试着用我前文所教的来回答这个问题：

在一个区块链中，两个主要的函式是什么？

希望你能够回答这个问题！这两个区块链拥有的主要函式，是用来建立初始区块，以及在后面新增新的区块。当然，现在我不会下放这个链并加入智慧合约，但是这些是基本函式！加入以下程式码到Blockchain.swift：


```swift
func createGenesisBlock(data:String) {
    let genesisBlock = Block()
    genesisBlock.hash = genesisBlock.generateHash()
    genesisBlock.data = data
    genesisBlock.previousHash = "0000"
    genesisBlock.index = 0
    chain.append(genesisBlock)
}
 
func createBlock(data:String) {
    let newBlock = Block()
    newBlock.hash = newBlock.generateHash()
    newBlock.data = data
    newBlock.previousHash = chain[chain.count-1].hash
    newBlock.index = chain.count
    chain.append(newBlock)
}
```

1. 我们加入的第一个函式是用来建立初始区块。为此，我们建立了一个函式来把区块的资料作为Input。然后，我们定义一个名为genesisBlock的变数，并将它设为Block型别。因为它是Block型别，所以它有我们之前在Block.swift定义的所有变数及函式。我们设定generateHash()为杂凑、 Input data为资料。因为这是第一个区块，所以我们将前一个区块的杂凑设定为000，好让我们知道这是初始区块。我们将它的索引值设为0 ，然后放到区块链chain。
2. 我们建立的下一个函式则适用于所有genesisBlock后的区块，而它会建立剩下的所有区块。你会注意到它跟之前的函式非常相似，唯一的不同的是我们将previousHash设定为前一个区块的杂凑，并将index设为它在区块练的位置。
完成了！我们已经成功定义自己的Blockchain！你的程式码应该如下图所示！

![](https://user-gold-cdn.xitu.io/2018/7/6/1646d1584e7538d5?w=1440&h=761&f=png&s=70645)

接着，我们将所有的部分连接到ViewController.swift档案，并看看执行成果吧！

### 钱包后台(Wallet Backend)

切换到ViewController.swift，我们可以看到所有的UI元件都已经连结完毕。我们所需要做的就是处理交易，并将交易列印到控制台上。
然而在开始之前，我们应该稍微探讨一下比特币区块链。比特币来自一个总帐号，假设这个帐号的编号是000。当你挖掘一颗比特币时，就表示你解答了数学问题，并获得一定数量的比特币作为奖励。这是发行货币一个很聪明的方法，同时也创造了让更多人去挖掘的动机。在我们的App 里，我们将100 BTC 作为奖励。首先，让我们在ViewController 添加需要的变数：

```swift
let firstAccount = 1065
let secondAccount = 0217
let bitcoinChain = Blockchain()
let reward = 100
var accounts: [String: Int] = ["0000": 10000000]
let invalidAlert = UIAlertController(title: "Invalid Transaction", message: "Please check the details of your transaction as we were unable to process this.", preferredStyle: .alert)
```

我们定义两个帐号：一个编号为1065，另一个编号为0217。我们同时新增一个bitcoinChain变数来作为我们的区块链，并将reward设定为100。我们需要一个作为比特币来源的主帐号：这是我们的初始帐号，编号为0000，它拥有一千万个比特币。你可以把这个帐号当成银行，在每一次的奖励中，就会从中取出100个比特币，并转至合法的帐号里。我们也定义一个警告，在每次交易无法完成时显示。

现在，让我们来写些将会执行的泛用函式。你可以猜到这些函式是什么吗？

1. 第一个函式是用来处理交易的。我们要确认传送者及接收者的帐号中，接收或扣除的金额是正确的，而且这个资讯会被记录在我们的区块链上。
2. 下一个函式是要在控制台里印出完整的纪录，它会显示每个区块及每个区块内的资料。
3. 最后一个函式是用来验证区块链是否合法，方法为确认前个区块的杂凑是否符合下一个区块的资讯。因为我们不会示范任何骇客方法，所以范例中的链永远都是合法的。

#### Transaction 函式

以下是我们的泛用交易函式。在定义变数之下输入以下程式码：

```swift
func transaction(from: String, to: String, amount: Int, type: String) {
    // 1
    if accounts[from] == nil {
        self.present(invalidAlert, animated: true, completion: nil)
        return
    } else if accounts[from]!-amount < 0 {
        self.present(invalidAlert, animated: true, completion: nil)
        return
    } else {
        accounts.updateValue(accounts[from]!-amount, forKey: from)
    }
    
    // 2
    if accounts[to] == nil {
        accounts.updateValue(amount, forKey: to)
    } else {
        accounts.updateValue(accounts[to]!+amount, forKey: to)
    }
    
    // 3
    if type == "genesis" {
        bitcoinChain.createGenesisBlock(data: "From: \(from); To: \(to); Amount: \(amount)BTC")
    } else if type == "normal" {
        bitcoinChain.createBlock(data: "From: \(from); To: \(to); Amount: \(amount)BTC")
    }
}
```

看起来程式码很多，但是它的核心只是为每次的交易定义一些规则。在开头的地方，我们有四个参数：to、from、amount以及type。To、From、及Amount的含义一目了然，而Type基本上就是定义交易的类型。这里有两种Type：Normal和Genesis。一个Normal的交易类型会是在帐号1065与0217之间进行，而Genesis交易类型则会涉及到帐号0000。
1. 第一个if-else条件式是关于来源帐号。如果来源帐号不存在或金额不足，我们会显示交易无效的警告，然后结束函式。而如果通过的话，我们会更新数值。
2.     第二个if-else条件式是关于接收帐号。如果接收帐号不存在，那么我们随它而去，然后结束函式。要不然，我们就会传送正确的比特币数量到帐号。
3. 第三个if-else条件式处理交易的类型。如果一个交易涉及初始区块，我们就建立一个新的初始区块；反之我们建立一个新区块来储存资料。

#### Printing 函式

在每次交易的最后，我们想要看到一个清单列出所有交易，来确保我们知道所有发生的事情。以下是我们在transaction函式下输入的程式码：

```swift
func chainState() {
    for i in 0...bitcoinChain.chain.count-1 {
        print("\tBlock: \(bitcoinChain.chain[i].index!)\n\tHash: \(bitcoinChain.chain[i].hash!)\n\tPreviousHash: \(bitcoinChain.chain[i].previousHash!)\n\tData: \(bitcoinChain.chain[i].data!)")
    }
    redLabel.text = "Balance: \(accounts[String(describing: firstAccount)]!) BTC"
    blueLabel.text = "Balance: \(accounts[String(describing: secondAccount)]!) BTC"
    print(accounts)
    print(chainValidity())
}
```

这是一个简单的for回圈，包含bitcoinChain的每个区块。我们印出区块的编号、杂凑值、前个区块的杂凑、以及储存的资料，再更新UILabel来显示每个帐号内正确的BTC数目。最后，印出一个列出每个帐号的清单（应该会有三个），并验证链的合法性。
现在，你应该会在函式最后一行中发生错误。这是因为我们还没定义chainValidity()函式，那么就来开始吧！

#### Validity 函式

记住，如果前一个区块的杂凑值符合目前区块所描述的内容，那么这一个链就是合法的。我们可以轻易地用另一个for 回圈来重复验证每个区块。

```swift
func chainValidity() -> String {
    var isChainValid = true
    for i in 1...bitcoinChain.chain.count-1 {
        if bitcoinChain.chain[i].previousHash != bitcoinChain.chain[i-1].hash {
            isChainValid = false
        }
    }
    return "Chain is valid: \(isChainValid)\n"
}
```

跟之前有点相似，我们在bitcoinChain中重复验证每个区块，来确认前一个区块的杂凑值是否与目前区块所描述的内容符合。
这样就完成了！我们已经定义了函式，并将会每次都用到它们！你的ViewController.swift现在应该看起来像这样：

![](https://user-gold-cdn.xitu.io/2018/7/6/1646d182e224eb54?w=1440&h=900&f=png&s=56076)

现在我们只需要将按钮连接到函式就完成了，来开始最后的篇章吧！

### 将所有东西连结在一起

当我们的App首次启动时，我们想让初始帐号0000传送50 BTC到我们的第一个帐号。然后，我们将让第一个帐号传送10 BTC到第二个帐号。这个步骤仅需三行程式码就可以完成。如此更改你的viewDidLoad函式：

```swift
override func viewDidLoad() {
    super.viewDidLoad()
    transaction(from: "0000", to: "\(firstAccount)", amount: 50, type: "genesis")
    transaction(from: "\(firstAccount)", to: "\(secondAccount)", amount: 10, type: "normal")
    chainState()
    self.invalidAlert.addAction(UIAlertAction(title: "OK", style: .default, handler: nil))
}
```

我们使用先前定义好的函式，并在最后呼叫chainState()。同时，我们新增一个OK按钮到交易无效的警告中。
现在让我们来看看剩下的四个函式里要加入什么：redMine()、blueMine()、redSend()及blueSend()。

#### 挖矿函式

挖矿函式非常地简单，只要三行程式码就行了。这就是我们要添加的程式码：

```swift
@IBAction func redMine(_ sender: Any) {
    transaction(from: "0000", to: "\(firstAccount)", amount: 100, type: "normal")
    print("New block mined by: \(firstAccount)")
    chainState()
}
    
@IBAction func blueMine(_ sender: Any) {
    transaction(from: "0000", to: "\(secondAccount)", amount: 100, type: "normal")
    print("New block mined by: \(secondAccount)")
    chainState()
}
```

在第一个挖矿函式中，我们使用transaction函式从初始帐号传送100 BTC到第一个帐号，先印出一个区块被挖出，再印出chainState。同样地，我们在blueMine函式里将100 BTC传送到第二个帐号。

#### 传送函式

传送函式与先前的函式也稍微相似。

```swift
@IBAction func redSend(_ sender: Any) {
    if redAmount.text == "" {
        present(invalidAlert, animated: true, completion: nil)
    } else {
        transaction(from: "\(firstAccount)", to: "\(secondAccount)", amount: Int(redAmount.text!)!, type: "normal")
        print("\(redAmount.text!) BTC sent from \(firstAccount) to \(secondAccount)")
        chainState()
        redAmount.text = ""
    }
}
    
@IBAction func blueSend(_ sender: Any) {
    if blueAmount.text == "" {
        present(invalidAlert, animated: true, completion: nil)
    } else {
        transaction(from: "\(secondAccount)", to: "\(firstAccount)", amount: Int(blueAmount.text!)!, type: "normal")
        print("\(blueAmount.text!) BTC sent from \(secondAccount) to \(firstAccount)")
        chainState()
        blueAmount.text = ""
    }
}
```

首先，我们确认redAmount或blueAmount中的文字栏位是否为空值。如果是，我们会显示一个交易无效的警告。如果不是，我们就可以继续。我们使用transaction函式输入金额，并把交易设为normal型态，以将第一个帐号的金额传送到第二个帐号（或相反）。我们印出被传送的金额，然后呼叫chainState()函式。最后，把文字栏位清空。
这样我们就完成啰！确认一下你的程式码是否符合下图所示。

![](https://user-gold-cdn.xitu.io/2018/7/6/1646d19c12ace9cc?w=1440&h=764&f=png&s=91014)

执行App 试试看！从前端来说，它看起来就如一个普通的交易App，但你会知道它后台的运作。试试使用App 将BTC 从一个帐号交易给另一个帐号、并试着欺骗App 吧！

![](https://user-gold-cdn.xitu.io/2018/7/6/1646d1a0e4f97390?w=1920&h=1080&f=png&s=103382)

## 结论

在这次的教学中，你学到了如何使用Swift 来建立一个区块链，并建立自己的比特币交易。请注意在真实的加密货币后台里，实作部分是跟上文是完全不一样的东西，因为它需要藉由智慧合约来分散，但是上面的示范内容用来学习的。
在这个范例中，我们运用了比特币来当加密货币，但你能想到区块链还的其他用途吗？欢迎在下面留言分享你的看法！希望你在此学到新的东西！
你可以在Github下载完整项目作参考。

完整项目：https://github.com/appcoda/BlockchainDemo 
原文链接：https://www.appcoda.com/blockchain-introduction/