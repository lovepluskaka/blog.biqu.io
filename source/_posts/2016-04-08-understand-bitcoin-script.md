title: 理解比特币脚本
date: 2016-04-08 12:00
---

作者：[汪海波](http://weibo.com/highball)

其实我们可以这样看待比特币的交易：『交易的发起者悬赏若干比特币，在网络上贴出了一到数学题，谁解出了这道数学题，悬赏就归谁了』。 顺着这个思路，Alice对Bob的转账可以理解为『Alice把一道只有Bob才能解开的数学题发到网络上，Bob解出题并拿走了悬赏』。那么，每个交易数据中都会出现的『脚本』就是题和解，『脚本语言』就是用来描述题和解的工具。

![图01](https://cloud.githubusercontent.com/assets/514951/14377566/e6d18d00-fda2-11e5-9c90-a237ca251bff.png)

## 『输入脚本』和『输出脚本』

在这里我们先讨论单输入单输出的比特币交易，因为这样描述起来更方便且不影响对『脚本』的理解。
[9c50cee8d50e273100987bb12ec46208cb04a1d5b68c9bea84fd4a04854b5eb1](https://blockchain.info/zh-cn/tx/9c50cee8d50e273100987bb12ec46208cb04a1d5b68c9bea84fd4a04854b5eb1) 这是一个单输入单输出交易，看下我们要关注的数据：

```
Hash:
9c50cee8d50e273100987bb12ec46208cb04a1d5b68c9bea84fd4a04854b5eb1

输入交易:
  前导输入的Hash:
  437b95ae15f87c7a8ab4f51db5d3c877b972ef92f26fbc6d3c4663d1bc750149
  
  输入脚本 scriptSig:
  3045022100efe12e2584bbd346bccfe67fd50a54191e4f45f945e3853658284358d9c062ad02200121e00b6297c0874650d00b786971f5b4601e32b3f81afa9f9f8108e93c752201
  038b29d4fbbd12619d45c84c83cb4330337ab1b1a3737250f29cec679d7551148a

输出交易:
  转账值:
  0.05010000 btc

  输出脚本 scriptPubKey:
  OP_DUP OP_HASH160 be10f0a78f5ac63e8746f7f2e62a5663eed05788 OP_EQUALVERIFY OP_CHECKSIG
```

假设Alice是转账发送者，Bob是接受者。那么『输入交易』表明了Alice要动用的比特币的来源，『输出交易』表明了Alice要转账的数额和转账对象——Bob。那么，你可能要问，数据中的『输入脚本』和『输出脚本』是不是就是题和解？对了一半！

在[Bitcoin Wiki][1]中提到：

> 原先发送币的一方，控制脚本运行，以便比特币在下一个交易中使用。想花掉币的另一方必须把以前记录的运行为真的脚本，放到输入区。

换句话说，在一个交易中，『输出脚本』是数学题，『输入脚本』是题解，**但不是这道数学题的题解。**我开始看Wiki的时候，在这里遇到了一些障碍，没法理解『输入脚本』和『输出脚本』的联系。但是在考虑交易间的关系后，就明白了。

假设有这么一系列交易：

![图02](https://cloud.githubusercontent.com/assets/514951/14377658/8eb0c22a-fda3-11e5-8d64-d6a6ab4d361a.png)

1. 上图的三个交易都是单输入单输出交易
1. 每个『输入交易』『输出交易』中，都包含对应的『脚本』
1. **交易a**，Alice转账给Bob；**交易b**，Bob转账给Carol；**交易c**，Carol转账给Dave
1. 当前交易的『输入』都引用前一个交易的『输出』，如交易b的『输入』引用交易a的『输出』

按照之前的说法，**交易a**中的『输出脚本』就是Alice为Bob出的数学题。那么，Bob想要引用**交易a**『输出交易』的比特币，就要解开这道数学题。题解是在**交易b**的『输入脚本』里给出的！Bob解开了这道题，获得了奖金，然后在**交易b**中为Carol出一道数学题，等待Carol来解...

所以说，下图中相同颜色的『输出』和『输入』才是一对题和解：

![图03](https://cloud.githubusercontent.com/assets/514951/14377680/9a8ac6b8-fda3-11e5-965e-80bda1aed8c6.png)

## 脚本语言
[Bitcoin Wiki][1]给出的对脚本的解释:

> 比特币在交易中使用脚本系统，与FORTH(一种编译语言)一样，脚本是简单的、基于堆栈的、并且从左向右处理，它特意设计成非图灵完整，没有LOOP语句。

要理解比特币脚本，先要了解『堆栈』，这是一个后进先出(Last In First Out )的容器，脚本系统对数据的操作都是通过它完成的。比特币脚本系统中有两个堆栈：主堆栈和副堆栈，一般来说主要使用主堆栈。举几个简单的例子，看下指令是如何对堆栈操作的（完整的指令集在[Wiki][1]里可以找到）:

* 常数入栈：把一段常数压入到堆栈中，这个常数成为了栈顶元素
![图04](https://cloud.githubusercontent.com/assets/514951/14377676/9a6d1d16-fda3-11e5-9f83-275c6f95bd61.png)

* OP_DUP：复制栈顶元素
![图05](https://cloud.githubusercontent.com/assets/514951/14377672/9a4d5b5c-fda3-11e5-8365-72ff3034ec4d.png)

* OP_EQUALVERIFY：检查栈顶两个元素是否相等
![图06](https://cloud.githubusercontent.com/assets/514951/14377674/9a4db156-fda3-11e5-9fbc-74c6119ddb58.png)

## 标准交易脚本
也就是P2PKH(Pay To Public Key Hash)，我们常用的转账方式。Alice在转账给Bob的时候，『输出交易』中给出了Bob的『钱包地址』(等价于『公钥哈希』)；当Bob想要转账给Carol的时候，他要证明自己拥有这个『钱包地址』对应的『私钥』，所以在『输入交易』中给出了自己的『公钥』以及使用『私钥』对交易的签名。看个实例：

* 交易a: [9c50cee8d50e273100987bb12ec46208cb04a1d5b68c9bea84fd4a04854b5eb1](https://blockchain.info/zh-cn/tx/9c50cee8d50e273100987bb12ec46208cb04a1d5b68c9bea84fd4a04854b5eb1)
* 交易b: [62fadb313b74854a818de4b4c0dc2e2049282b28ec88091a9497321203fb016e](https://blockchain.info/tx/62fadb313b74854a818de4b4c0dc2e2049282b28ec88091a9497321203fb016e)

交易b中有一个『输入交易』引用了交易a的『输出交易』，它们的脚本是一对题与解：

**题：**交易a的『输出脚本』，若干个脚本指令和转账接收方的『公钥哈希』
```
OP_DUP OP_HASH160 be10f0a78f5ac63e8746f7f2e62a5663eed05788 OP_EQUALVERIFY OP_CHECKSIG 
```

**解：**交易b的『输入脚本』，这么一长串只是两个元素，『签名』和『公钥』（sig & pubkey）
```
3046022100ba1427639c9f67f2ca1088d0140318a98cb1e84f604dc90ae00ed7a5f9c61cab02210094233d018f2f014a5864c9e0795f13735780cafd51b950f503534a6af246aca301
03a63ab88e75116b313c6de384496328df2656156b8ac48c75505cd20a4890f5ab
```
下面来看下这两段脚本是如何执行，来完成『解题』过程的。

1. 首先执行的是『输入脚本』。因为脚本是从左向右执行的，那么先入栈的是『签名』，随后是『公钥』
![图07](https://cloud.githubusercontent.com/assets/514951/14377675/9a4ecf50-fda3-11e5-88b3-4bb3d5bdb538.png)

2. 接着，执行的是『输出脚本』。从左向右执行，第一个指令是`OP_DUP `——复制栈顶元素
![图08](https://cloud.githubusercontent.com/assets/514951/14377673/9a4d4734-fda3-11e5-8ff3-ab099274fe1e.png)

3. `OP_HASH160`: 计算栈顶元素Hash，得到`pubkeyhash `
![图09](https://cloud.githubusercontent.com/assets/514951/14377677/9a7c5538-fda3-11e5-82ee-7cbb5789e62f.png)

4. 将『输出脚本』中的『公钥哈希』入栈，为了和前面计算得到的哈希区别，称它为`pubkeyhash`
![图10](https://cloud.githubusercontent.com/assets/514951/14377678/9a7cee4e-fda3-11e5-8745-3cdf582a0b77.png)

5. `OP_EQUALVERIFY`: 检查栈顶前两元素是否相等，如果相等继续执行，否则中断执行，返回失败
![图11](https://cloud.githubusercontent.com/assets/514951/14377679/9a7ed808-fda3-11e5-887e-d2dd7d9c5143.png)

6. `OP_CHECKSIG`: 使用栈顶前两元素执行签名校验操作，如果相等，返回成功，否则返回失败
![图12](https://cloud.githubusercontent.com/assets/514951/14377682/9a9f8878-fda3-11e5-9de7-6c7544f12107.png)

这样一串指令执行下来，就可以验证这道数学题是否做对了，也就是说验明了想要花费『钱包地址』中比特币的人是否拥有对应的『私钥』。上面的执行过程是可以在[脚本模拟器](http://webbtc.com/script)中执行的，能够看到每一步执行的状态，感兴趣的童鞋可以尝试一下。

其实除了标准的P2PKH交易脚本，还有P2SH的Multi-Sig脚本以及真正的『解谜交易』脚本，我们可以在今后接着讨论。

___
## 参考
 \[1] [比特币脚本][1]
 \[2] [申屠青春(我看比特币][3]
 \[3] [Bitcoin Wiki, Script][2]

[1]:http://www.8btc.com/bitcoin_scripts
[2]:https://en.bitcoin.it/wiki/Script
[3]:http://weibo.com/MyBitcoin
