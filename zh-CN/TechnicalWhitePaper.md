# EOS.IO 技术白皮书

**草案：2017 年 6 月 26 日**

**摘要：** EOS.IO 软件引入一种新的区块链架构设计，它使得去中心化的应用可以横向和纵向的扩展。 这通过构建一个仿操作系统的方式来实现，在它之上可以构建应用程序。 该软件提供帐户、身份验证、数据库、异步通信和跨越数百个 CPU 内核或集群的应用程序调度。 由此产生的技术是一种区块链架构，它可以扩展至每秒处理百万级交易，消除用户的手续费，并且允许快速和轻松的部署去中心化的应用。

**PLEASE NOTE: CRYPTOGRAPHIC TOKENS REFERRED TO IN THIS WHITE PAPER REFER TO CRYPTOGRAPHIC TOKENS ON A LAUNCHED BLOCKCHAIN THAT ADOPTS THE EOS.IO SOFTWARE. THEY DO NOT REFER TO THE ERC-20 COMPATIBLE TOKENS BEING DISTRIBUTED ON THE ETHEREUM BLOCKCHAIN IN CONNECTION WITH THE EOS TOKEN DISTRIBUTION.**

Copyright © 2017 block.one

未经允许，在非用于商业和教育用途的前提下 (即，除了收取费用或商业目的)，如果注明原始出处并适用声明的版权，任何人可以使用、复制或发布本白皮书内的任何内容。

**免责声明：** 本 EOS.IO 技术白皮书草案仅供参考。 block.one does not guarantee the accuracy of or the conclusions reached in this white paper, and this white paper is provided “as is”. block.one does not make and expressly disclaims all representations and warranties, express, implied, statutory or otherwise, whatsoever, including, but not limited to: (i) warranties of merchantability, fitness for a particular purpose, suitability, usage, title or noninfringement; (ii) that the contents of this white paper are free from error; and (iii) that such contents will not infringe third-party rights. block.one and its affiliates shall have no liability for damages of any kind arising out of the use, reference to, or reliance on this white paper or any of the content contained herein, even if advised of the possibility of such damages. In no event will block.one or its affiliates be liable to any person or entity for any damages, losses, liabilities, costs or expenses of any kind, whether direct or indirect, consequential, compensatory, incidental, actual, exemplary, punitive or special for the use of, reference to, or reliance on this white paper or any of the content contained herein, including, without limitation, any loss of business, revenues, profits, data, use, goodwill or other intangible losses.

- [背景](#background)
- [区块链应用的要求](#requirements-for-blockchain-applications) 
  - [支持成百上千的用户](#support-millions-of-users)
  - [免费的使用](#free-usage)
  - [简单升级和 bug 修复](#easy-upgrades-and-bug-recovery)
  - [低延时](#low-latency)
  - [时序性能](#sequential-performance)
  - [并发性能](#parallel-performance)
- [共识算法 (DPOS)](#consensus-algorithm--dpos-) 
  - [交易确认](#transaction-confirmation)
  - [股权证明的交易 (TaPoS)](#transaction-as-proof-of-stake--tapos-)
- [帐户](#accounts) 
  - [消息 & 处理](#messages---handlers)
  - [基于角色的权限管理](#role-based-permission-management) 
    - [命名的权限级别](#named-permission-levels)
    - [命名的消息处理群组](#named-message-handler-groups)
    - [权限映射](#permission-mapping)
    - [评估权限](#evaluating-permissions) 
      - [默认权限群组](#default-permission-groups)
      - [权限并行评估](#parallel-evaluation-of-permissions)
  - [带强制性延时的消息](#messages-with-mandatory-delay)
  - [恢复被盗窃的密钥](#recovery-from-stolen-keys)
- [应用程序的确定性并行执行](#deterministic-parallel-execution-of-applications) 
  - [最小化通信延迟](#minimizing-communication-latency)
  - [只读信息的处理](#read-only-message-handlers)
  - [多帐户的原子化交易](#atomic-transactions-with-multiple-accounts)
  - [区块链状态的部分评估](#partial-evaluation-of-blockchain-state)
  - [自主最优调度](#subjective-best-effort-scheduling)
- [Token 模型与资源使用](#token-model-and-resource-usage) 
  - [客观与主观的度量](#objective-and-subjective-measurements)
  - [接收方付费](#receiver-pays)
  - [委托能力](#delegating-capacity)
  - [分离交易成本与 Token 价值](#separating-transaction-costs-from-token-value)
  - [状态存储成本](#state-storage-costs)
  - [区块奖励](#block-rewards)
  - [社区效益应用](#community-benefit-applications)
- [治理](#governance) 
  - [冻结帐户](#freezing-accounts)
  - [更改帐户代码](#changing-account-code)
  - [宪法](#constitution)
  - [升级协议 & 宪法](#upgrading-the-protocol---constitution) 
    - [紧急变更](#emergency-changes)
- [脚本 & 虚拟机](#scripts---virtual-machines) 
  - [模式定义的消息](#schema-defined-messages)
  - [模式定义的数据库](#schema-defined-database)
  - [分离授权与应用](#separating-authentication-from-application)
  - [虚拟机独立架构](#virtual-machine-independent-architecture) 
    - [Web 组建 (WASM)](#web-assembly)
    - [以太访虚拟机 (EVM)](#ethereum-virtual-machine--evm-)
- [跨链通信](#inter-blockchain-communication) 
  - [用于轻客户端的 Merkle 证明 (LCV)](#merkle-proofs-for-light-client-validation--lcv-)
  - [跨链通信的延时](#latency-of-interchain-communication)
  - [完备性证明](#proof-of-completeness)
- [结论](#conclusion)

# 背景

区块链技术是通过 2008 年诞生的比特币货币得以被认知，自从那之后企业家和开发者就不断的尝试推广这一技术，以便在单一的区块链平台上支持更为广泛的应用程序。

而一些区块链平台努力的支持可运作的去中心化应用，具体的应用比如 BitShares 去中心化交易所 (2014) 和 Steem 社交媒体平台 (2016) 已经成为每天被成千上万活跃用户重度使用的区块链。 他们能做到这些，是通过性能的提升达到每秒处理上千交易，消除手续费和提供堪比已经存在的中心化服务的用户体验。

已存在的区块链平台承担着大量的交易费和有限的可计算能力，这都阻碍了区块链技术的大面积应用。

# 区块链应用的要求

为了赢得广泛的应用，构建在区块链之上的应用需要一个灵活性足以满足以下要求的平台：

## 支持成百上千的用户

像 Ebay、Uber、AirBnB 和 Facebook 这样企业，他们需要区块链技术能处理每日数以千万的活跃用户。 在某些情况下，除非用户群体达到一个极庞大的量级否则应用并无用武之地，因此一个可以处理极其庞大用户的平台是至关重要的。

## 免费的使用

Application developers need the flexibility to offer users free services; users should not have to pay in order to use the platform or benefit from its services. 一个可以免费供用户使用的区块链平台或许将赢得更为广泛的使用。 开发者和企业可以制订有效的货币化战略。

## 简单升级和 bug 修复

企业构建区块链基础的应用需要能够为应用增加新特性的灵活性。

所有非同凡响的软件都会受到 bug 的影响，即便是经过了最严格意义上的验证。这个平台必须具有足够的鲁棒性以便应对不可避免出现的 bug。

## 低延时

一个好的用户体验需要延时时间在数秒内就能收到可靠的反馈。 高延时会阻碍用户，并且会让构建在区块链上的应用比已有的非区块链应用缺乏竞争力。

## 时序性能

一些应用因为顺序依赖关系的执行步骤而不能使用并发算法实现。 比如交易所就需要足够的时序性能来处理很高的交易量，因此高时序性能处理的平台是必须的。

## 并发性能

大型可扩展应用需要将工作量分配到多 CPU 和计算机之上。

# 共识算法 (DPOS)

EOS.IO 软件使用唯一能满足区块链之上应用性能需求的去中心化共识算法，[委托股权证明 (DPOS)](https://steemit.com/dpos/@dantheman/dpos-consensus-algorithm-this-missing-white-paper)。 Under this algorithm, those who hold tokens on a blockchain adopting the EOS.IO software may select block producers through a continuous approval voting system and anyone may choose to participate in block production and will be given an opportunity to produce blocks proportional to the total votes they have received relative to all other producers. For private blockchains the management could use the tokens to add and remove IT staff.

EOS.IO 软件使得区块准确的每 3 秒生成一个并且在任何时间点都只有一个被授权的生产者来生成区块。 如果一个区块在规定时间之内未被生产出来则这一区块将被跳过。 当一个或多个区块被跳过发生时，在区块链中会有一个 6 秒及以上的间隔。

在 EOS.IO 软件中，区块通过 21 名生产者轮流产生。 在每一轮的开始时，21 个唯一的区块生产者被选出。 获票最高的前 20 名自动在没轮被选中，剩余的一个生产者通过得票比例选出。 被选中的生产者通过从区块取到的时间作为伪随机数来打乱其顺序。 打乱顺序是为确保这些生产者与其他生产者保持均衡的连通性。

如果一个生产者错过了一个区块并且在过去的 24 小时内没有生产任何的区块，那么它将被从候选中移除，直到它在区块链中通知它要开始再次生产区块的意图。 这样通过最小化区块丢失数量（因被证实不可靠的节点不作为导致）来确保网络操作的稳定性。

在一般情况下，一个 DPOS 区块链不会经历任何的分叉，因为区块生产者是通过合作而非竞争的方式来生产区块。 即便真的出现了分叉，共识也将自动的切换到最长的链上。 之所以会这样运作，是因为区块添加到一个区块链分叉的速率与公用同一共识的区块生产者比例是相关的。 换句话说，具有更多生产者的区块链分叉会比拥有较少生产的那一个条增长的速度更快。 而且，没有一个生产者会同时在两个分叉上同时生产区块。 如果一个区块生产者被抓到做这样的事儿，那么这个生产者将很可能被投票投出。 这些双重生产行为对应密码学凭证可以用来自动的删除这些滥用者。

## 交易确认

通常 DPOS 区块链 100% 会有区块生产者参与。一个交易从广播开始后平均 1.5 秒就可以 99.9% 被认为是确认了。

在一些特殊情况下例外，软件出现 bug，网络拥塞，或一个恶意的区块生产者制造了两个或更多的分叉。 为了确保一个交易绝对是不可逆的，一个节点可以选择等待 21 个区块生产者中的 15 个给出确认。 基于通常的 EOS.IO 软件配置，在一般情况下这需要平均 45 秒的时间。 默认情况下，所有的节点将认为当 21 个生产者中有 15 个给出确认后这一区块就是不可逆的了，并且不管长度如何都不会切换到没有这一区块的分叉。

在分叉开始的 9 秒内，一个节点就可以警告用户他们极可能正处于分叉中。 在连续丢失 2 个区块后，有 95% 的概率可以确认一个节点处于分叉中。 在连续丢失 3 个区块后就有 99% 的概率确认。 可以通过节点丢失、近期参与比率和其他参数来构建鲁棒性预测模型，从而快速的警告操作者出现了问题。

对于这种警告的反应完全取决于商业交易的性质，但最简单的做法就是等待 15/21 的确认直到警告消失。

## 股权证明的交易 (TaPoS)

EOS.IO 软件需要每一个交易包含最近一个区块头的哈希值。这个哈希值有两个目的：

  1. 防止不包含区块引用的交易在分叉时重放发生；和
  2. 通知网络对应的用户和他们的股份当前在某个具体的分叉上。

随着时间的推移，所有的用户直接确认区块链，在这一链条上难以伪造假的链条，因为假的链条根本无法从合法链条上迁移交易。

# 帐户

EOS.IO 软件允许所有的帐户使用一个唯一的人类可读的名称来索引，长度在 2 到 32 个字符之间。 这个名称由帐户创建者自己选择。 所有的帐户必须在创建时用极少的帐户余额来注资，从而覆盖存储帐户信息的成本。 帐户名称也支持命名空间，比如 @domain 这个帐户的拥有者是唯一可以创建 @user.domain 帐户的人。

在一个去中心化的场景中，应用开发者将会为新用户注册成本买单。 Traditional businesses already spend significant sums of money per customer they acquire in the form of advertising, free services, etc. 比起来，资助一个新的区块链帐户的花费简直微不足道。 值得庆幸的是，对一个已经在另一个应用注册过的用户并不需要再创建新的帐户。

## 消息 & 处理

每个帐户可以发送结构化的消息给其他的帐户，并且可以定义脚本来处理他们接收到的消息。 EOS.IO 软件给每个帐户提供了只有自己的消息处理脚本能访问的私有数据库。 消息处理脚本同样可以给其他帐户发送消息。 消息和自动化的消息处理的结合决定了 EOS.IO 如何定义智能合约的。

## 基于角色的权限管理

权限管理涉及判定一条消息是否被正确的授权。 权限管理最简单的形式就是检查一个交易包含必须的签名，但这意味着必须的签名是已知的。 一般情况下，权威必然是独立的个体或者个体组成的群体，并且是被划分开的。 EOS.IO 软件提供了声明式的权限管理系统，通过管理谁可以在什么时间做什么来给用户细力度和高维度的控制。

授权和权限管理被标准化和脱离应用的商业逻辑是不可取的。 这使得管理权限的工具得以被开发，既满足常规的需求又为性能优化提供了重要的可能性。

每一个帐户可以被任何权重组合的其他帐户和私钥管控。 这创建了分层级的权利结构，这反映了现实中的权限分配方式，并且让多用户共同管理资产变得从未如此简单。 多用户控制是安全最大的贡献者，并且，当用户使用得当，它可以极大的消除因被黑而导致被盗窃的风险。

EOS.IO software allows accounts to define what combination of keys and/or accounts can send a particular message type to another account. 举个例子，可以指定一个密钥给一个用户的社交媒体账号，同时另一个密钥访问交易所。 甚至可以给其他帐户权限来代表自己而无需分配给他们密钥。

### 命名的权限级别

<img align="right" src="http://eos.io/wpimg/diagram3.png" width="228.395px" height="300px" />

在 EOS.IO 软件中，帐户可以定义命名的权限级别，每一个是由更高级别的命名权限派生而来。 每一个命名的权限级别定义了一个权威；一个权威是多重签名阈值校验，它包含密钥和／或其他帐户的命名权限级别。 打个比方，一个帐户的“朋友”权限级别可以被设置为由该帐户的任何一个朋友无差别的控制。

另一个例子在 Steem 区块链中，它包含三个硬编码的命名权限级别：拥有，活跃和发帖。 发帖权限就只能进行如投票和发帖的社交活动，而活跃权限可以做除了变更拥有之外的所有的事情。 拥有权限的意思是冷存储并且有能力做任何事。 The EOS.IO software generalizes this concept by allowing each account holder to define their own hierarchy as well as the grouping of actions.

### 命名的消息处理群组

EOS.IO 软件允许每个帐户将他们自己的消息组织到一个命名和嵌套的群组中。 这个命名的消息处理群组可以在其他帐户配置他们权限级别时被引用。

最高级别的消息处理群组是帐户名称，最低级别的是一个帐户接收到的单独的消息类型。 这些群组可以被这样的方式引用： **@accountname.groupa.subgroupb.MessageType**.

在这样的模型之下，交易所合约可以通过将挂单的创建和取消分组，从而与充值提现分离开。 交易所合约的这样分组对用户而言带来了方便。

### 权限映射

EOS.IO 软件允许每个帐户定义从任意帐户的一个命名的消息处理群组与自己的命名的权限级别之间建立映射。 举个例子，一个帐户所有者可以将自己社交媒体应用与自己的“朋友”权限群组建立映射。 有了这个映射，任何朋友可以以这一帐户的身份在这一帐户的社交媒体上发帖。 尽管他们将以帐户所有者的身份发帖，他们仍然使用自己的密钥来签名消息。 这意味着总是可以辨识出是哪一个朋友在以何种方式使用帐户。

### 评估权限

当 **@alice** 以 "**Action**" 类型发送一条消息给 **@bob** 时，EOS.IO 软件首先会检查 **@alice** 是否为 **@bob.groupa.subgroup.Action** 定义过权限映射。 如果什么都没有找到，紧接着检查 **@bob.groupa.subgroup** 映射，然后是 **@bob.groupa**，最后 **@bob** 将被检查。 如果都没有找到，那么假定映射为命名的权限群组 **@alice.active**。

一旦一个映射被识别，则通过阈值多签名流程验证签名权威，并且关联权威与命名的权限。 如果失败了，则跃迁至父权限，直至拥有者权限，**@alice.owner**。

<img align="center" src="http://eos.io/wpimg/diagram2grayscale2.jpg" width="845.85px" height="500px" />

#### 默认权限群组

The EOS.IO technology also allows all accounts to have an "owner" group which can do everything, and an "active" group which can do everything except change the owner group. 所有其他的全新群组派生自“活动”群组。

#### 权限并行评估

权限评估过程是“只读”的，并且通过交易对权限的变更在一个区块结束之前不会起作用。 这意味着对所有的交易对应的密钥和权限评估可以被并行执行。 此外，这意味着一个快速的权限验证是可行的，它无需启动会引起回滚需求的高成本的应用逻辑。 最后，这意味着交易权限可以被评估即便接收到等待的交易，并且之后无需再重新评估。

从各方面考虑，权限验证占据了验证交易计算量的很大比例。 让其只读和普遍的并发处理将会使得性能有一个质的飞跃。

当从消息日志中重新生成确定性状态时不再需要重复的权限验证。 事实是一个交易如果被包含近了一个被认为不存在问题的区块时它就有足够的理由跳过这 步这将极大减少因为区块链增长拉去过去记录时的计算量。

## 带强制性延时的消息

时间是安全中的一个关键组成部分。 在大多数情况下，一个私钥在没有被使用前都无从知晓它是否被偷窃。 当人们有需要密钥的应用在每天联网使用的电脑上运行时，基于时间的安全会更为重要。 EOS.IO 软件让应用开发者可以指明消息必须在被加到一个区块之前等待最小的时间间隙。 During this time they can be cancelled.

用户可以在消息广播出去后通过邮件或者文字消息的形式收到通知。 如果他们没有授权，那么他们可以使用帐户恢复流程来恢复帐户，并收回消息。

这个必须的延时由操作敏感性决定。 为一杯咖啡付款可以没有任何的延时，几秒之内就不可逆了，而购买一个房子也许需要 72 消失的结算期。 转移整个帐户到一个新的控制可能需要长达 30 天。 具体的延时选择由开发者和用户自己来做选择。

## 恢复被盗窃的密钥

EOS.IO 软件提供给用户一种找回自己失窃密钥控制权的方式。 一个帐户的所有者可以使用过去 30 天任何活跃的拥有者密钥与事先指定的合作者帐户给出的批准来重置自己帐户的密钥。 帐户的恢复合作者在没有所有人帮助的情况下无法重置帐户的控制权。

黑客尝试进行恢复流程是无意义的，因为他们已经“控制”了帐户。 此外，就算他们真的进行这一流程，恢复合作者也会询问身份证明和多因素认证 (手机和邮件)。 这会让黑客脱作出让步或者无功而返。

这一流程与简单的多重签名有很大差异。 在多重签名中，另一个公司要参与所有转账的执行，但在恢复流程中，它却只在恢复时才起作用对每天的转账无从干预。 这大大的降低了参与者的成本和法律责任。

# 应用程序的确定性并行执行

区块链共识取决于确定性 (可重现的) 的行为。 这意味着所有的并行计算必须是不能互斥或者具有其他锁特性的。 没有了锁就必须有一些方式可以确保所有的帐户只可以读取和写入他们自己的私有数据库。 这也意味着每个帐户处理消息是顺序的，而并发只能在帐户层面进行。

In an EOS.IO software-based blockchain, it is the job of the block producer to organize message delivery into independent threads so that they can be evaluated in parallel. 每个帐户的状态由且只由发送给它的消息决定。 进度表由区块生产者输出并且会被确定性的执行，但是生成进度表的过程却不一定是确定性的。 这意味着区块生产者可以使用并发算法来调度交易。

Part of parallel execution means that when a script generates a new message it does not get delivered immediately, instead it is scheduled to be delivered in the next cycle. The reason it cannot be delivered immediately is because the receiver may be actively modifying its own state in another thread.

## 最小化通信延迟

Latency is the time it takes for one account to send a message to another account and then receive a response. The goal is to enable two accounts to exchange messages back and forth within a single block without having to wait 3 seconds between each message. To enable this, the EOS.IO software divides each block into cycles. Each cycle is divided into threads and each thread contains a list of transactions. Each transaction contains a set of messages to be delivered. This structure can be visualized as a tree where alternating layers are processed sequentially and in parallel.

        区块
    
          循环 (顺序)
    
            线程 (并行)
    
              交易 (顺序)
    
                消息 (顺序)
    
                  接受者和被通知帐户 (并行)
    

Transactions generated in one cycle can be delivered in any subsequent cycle or block. Block producers will keep adding cycles to a block until the maximum wall clock time has passed or there are no new generated transactions to deliver.

It is possible to use static analysis of a block to verify that within a given cycle no two threads contain transactions that modify the same account. So long as that invariant is maintained a block can be processed by running all threads in parallel.

## 只读消息的处理

Some accounts may be able to process a message on a pass/fail basis without modifying their internal state. If this is the case then these handlers can be executed in parallel so long as only read-only message handlers for a particular account are included in one or more threads within a particular cycle.

## 多帐户的原子化交易

Sometimes it is desirable to ensure that messages are delivered to and accepted by multiple accounts atomically. In this case both messages are placed in one transaction and both accounts will be assigned the same thread and the messages applied sequentially. This situation is not ideal for performance and when it comes to "billing" users for usage, they will get billed by the number of unique accounts referenced by a transaction.

For performance and cost reasons it is best to minimize atomic operations involving two or more heavily utilized accounts.

## 区块链状态的部分评估

Scaling blockchain technology necessitates that components are modular. Everyone should not have to run everything, especially if they only need to use a small subset of the applications.

An exchange application developer runs full nodes for the purpose of displaying the exchange state to its users. This exchange application has no need for the state associated with social media applications. EOS.IO software allows any full node to pick any subset of applications to run. Messages delivered to other applications are safely ignored because an application's state is derived entirely from the messages that are delivered to it.

This has some significant implications on communication with other accounts. Most significantly it cannot be assumed that the state of the other account is accessible on the same machine. It also means that while it is tempting to enable "locks" that allow one account to synchronously call another account, this design pattern breaks down if the other account is not resident in memory.

All state communication among accounts must be passed via messages included in the blockchain.

## 自主最优调度

The EOS.IO software cannot obligate block producers to deliver any message to any other account. Each block producer makes their own subjective measurement of the computational complexity and time required to process a transaction. This applies whether a transaction is generated by a user or automatically by a script.

On a launched blockchain adopting the EOS.IO software, at a network level all transactions are billed a fixed computational bandwidth cost regardless of whether it took .01ms or a full 10 ms to execute it. However, each individual block producer using the software may calculate resource usage using their own algorithm and measurements. When a block producer concludes that a transaction or account has consumed a disproportionate amount of the computational capacity they simply reject the transaction when producing their own block; however, they will still process the transaction if other block producers consider it valid.

In general so long as even 1 block producer considers a transaction as valid and under the resource usage limits then all other block producers will also accept it, but it may take up to 1 minute for the transaction to find that producer.

In some cases a producer may create a block that includes transactions that are an order of magnitude outside of acceptable ranges. In this case the next block producer may opt to reject the block and the tie will be broken by the third producer. This is no different than what would happen if a large block caused network propagation delays. The community would notice a pattern of abuse and eventually remove votes from the rogue producer.

This subjective evaluation of computational cost frees the blockchain from having to precisely and deterministically measure how long something takes to run. With this design there is no need to precisely count instructions which dramatically increases opportunities for optimization without breaking consensus.

# Token 模型与资源使用

**PLEASE NOTE: CRYPTOGRAPHIC TOKENS REFERRED TO IN THIS WHITE PAPER REFER TO CRYPTOGRAPHIC TOKENS ON A LAUNCHED BLOCKCHAIN THAT ADOPTS THE EOS.IO SOFTWARE. THEY DO NOT REFER TO THE ERC-20 COMPATIBLE TOKENS BEING DISTRIBUTED ON THE ETHEREUM BLOCKCHAIN IN CONNECTION WITH THE EOS TOKEN DISTRIBUTION.**

All blockchains are resource constrained and require a system to prevent abuse. With a blockchain that uses EOS.IO software, there are three broad classes of resources that are consumed by applications:

  1. 带宽和日志存储 (磁盘)；
  2. 计算与计算储备 (中央处理器)；
  3. 状态存储 (内存)。

Bandwidth and computation have two components, instantaneous usage and long-term usage. A blockchain maintains a log of all messages and this log is ultimately stored and downloaded by all full nodes. With the log of messages it is possible to reconstruct the state of all applications.

The computational debt is calculations that must be performed to regenerate state from the message log. If the computational debt grows too large then it becomes necessary to take snapshots of the blockchain's state and discard the blockchain's history. If computational debt grows too quickly then it may take 6 months to replay 1 year worth of transactions. It is critical, therefore, that the computational debt be carefully managed.

Blockchain state storage is information that is accessible from application logic. It includes information such as order books and account balances. If the state is never read by the application then it should not be stored. For example, blog post content and comments are not read by application logic so they should not be stored in the blockchain's state. Meanwhile the existence of a post/comment, the number of votes, and other properties do get stored as part of the blockchain's state.

Block producers publish their available capacity for bandwidth, computation, and state. The EOS.IO software allows each account to consume a percentage of the available capacity proportional to the amount of tokens held in a 3-day staking contract. For example, if a blockchain based on the EOS.IO software is launched and if an account holds 1% of the total tokens distributable pursuant to that blockchain, then that account has the potential to utilize 1% of the state storage capacity.

Adopting the EOS.IO software on a launched blockchain means bandwidth and computational capacity are allocated on a fractional reserve basis because they are transient (unused capacity cannot be saved for future use). The algorithm used by EOS.IO software is similar to the algorithm used by Steem to rate-limit bandwidth usage.

## 客观与主观的度量

As discussed earlier, instrumenting computational usage has a significant impact on performance and optimization; therefore, all resource usage constraints are ultimately subjective and enforcement is done by block producers according to their individual algorithms and estimates.

That said, there are certain things that are trivial to measure objectively. The number of messages delivered and the size of the data stored in the internal database are cheap to measure objectively. The EOS.IO software enables block producers to apply the same algorithm over these objective measures but may choose to apply stricter subjective algorithms over subjective measurements.

## 接收方付费

Traditionally, it is the business that pays for office space, computational power, and other costs required to run the business. The customer buys specific products from the business and the revenue from those product sales is used to cover the business costs of operation. Similarly, no website obligates its visitors to make micropayments for visiting its website to cover hosting costs. Therefore, decentralized applications should not force its customers to pay the blockchain directly for the use of the blockchain.

A launched blockchain that uses the EOS.IO software does not require its users to pay the blockchain directly for its use and therefore does not constrain or prevent a business from determining its own monetization strategy for its products.

## 委托能力

A holder of tokens on a blockchain launched adopting the EOS.IO software who may not have an immediate need to consume all or part of the available bandwidth, can give or rent such unconsumed bandwidth to others; the block producers running EOS.IO software on such blockchain will recognize this delegation of capacity and allocate bandwidth accordingly.

## 分离交易成本与 Token 价值

One of the major benefits of the EOS.IO software is that the amount of bandwidth available to an application is entirely independent of any token price. If an application owner holds a relevant number of tokens on a blockchain adopting EOS.IO software, then the application can run indefinitely within a fixed state and bandwidth usage. In such case, developers and users are unaffected from any price volatility in the token market and therefore not reliant on a price feed. In other words, a blockchain that adopts the EOS.IO software enables block producers to naturally increase bandwidth, computation, and storage available per token independent of the token's value.

A blockchain using EOS.IO software also awards block producers tokens every time they produce a block. The value of the tokens will impact the amount of bandwidth, storage, and computation a producer can afford to purchase; this model naturally leverages rising token values to increase network performance.

## 状态存储成本

While bandwidth and computation can be delegated, storage of application state will require an application developer to hold tokens until that state is deleted. If state is never deleted then the tokens are effectively removed from circulation.

Every user account requires a certain amount of storage; therefore, every account must maintain a minimum balance. As storage capacity of the network increases this minimum required balance will fall.

## 块奖励

A blockchain that adopts the EOS.IO software will award new tokens to a block producer every time a block is produced. In these circumstances, the number of tokens created is determined by the median of the desired pay published by all block producers. The EOS.IO software may be configured to enforce a cap on producer awards such that the total annual increase in token supply does not exceed 5%.

## 社区效益应用

In addition to electing block producers, pursuant to a blockchain based on the EOS.IO software, users can elect 3 community benefit applications also known as smart contracts. These 3 applications will receive tokens of up to a configured percent of the token supply per annum minus the tokens that have been paid to block producers. These smart contracts will receive tokens proportional to the votes each application has received from token holders. The elected applications or smart contracts can be replaced by newly elected applications or smart contracts by token holders.

# 治理

Governance is the process by which people reach consensus on subjective matters that cannot be captured entirely by software algorithms. An EOS.IO software-based blockchain implements a governance process that efficiently directs the existing influence of block producers. Absent a defined governance process, prior blockchains relied ad hoc, informal, and often controversial governance processes that result in unpredictable outcomes.

A blockchain based on the EOS.IO software recognizes that power originates with the token holders who delegate that power to the block producers. The block producers are given limited and checked authority to freeze accounts, update defective applications, and propose hard forking changes to the underlying protocol.

Embedded into the EOS.IO software is the election of block producers. Before any change can be made to the blockchain these block producers must approve it. If the block producers refuse to make changes desired by the token holders then they can be voted out. If the block producers make changes without permission of the token holders then all other non-producing full-node validators (exchanges, etc) will reject the change.

## 冻结帐户

Sometimes a smart contact behaves in an aberrant or unpredictable manner and fails to perform as intended; other times an application or account may discover an exploit that enables it to consume an unreasonable amount of resources. When such issues inevitably occur, the block producers have the power to rectify such situations.

The block producers on all blockchains have the power to select which transactions are included in blocks which gives them the ability to freeze accounts. A blockchain using EOS.IO software formalizes this authority by subjecting the process of freezing an account to a 17/21 vote of active producers. If the producers abuse the power they can be voted out and an account will be unfrozen.

## 更改帐户代码

When all else fails and an "unstoppable application" acts in an unpredictable manner, a blockchain using EOS.IO software allows the block producers to replace the account's code without hard forking the entire blockchain. Similar to the process of freezing an account, this replacement of the code requires a 17/21 vote of elected block producers.

## 宪法

The EOS.IO software enables blockchains to establish a peer-to-peer terms of service agreement or a binding contract among those users who sign it, referred to as a "constitution". The content of this constitution defines obligations among the users which cannot be entirely enforced by code and facilitates dispute resolution by establishing jurisdiction and choice of law along with other mutually accepted rules. Every transaction broadcast on the network must incorporate the hash of the constitution as part of the signature and thereby explicitly binds the signer to the contract.

The constitution also defines the human-readable intent of the source code protocol. This intent is used to identify the difference between a bug and a feature when errors occur and guides the community on what fixes are proper or improper.

## 升级协议 & 宪法

The EOS.IO software defines a process by which the protocol as defined by the canonical source code and its constitution, can be updated using the following process:

  1. 区块生产者对宪法提出改建意见并获得 17/21 批准。
  2. 区块生产者持续 17/21 品准连续 30 天。
  3. 所有用户需要使用新的宪法来做签名。
  4. 区块生产通过变更代码的方式来影响宪法并且提交一个 git 记录的哈希值。
  5. 区块生产者持续 17/21 品准连续 30 天。
  6. 7 天后改为会起影响的代码，给所有完整节点 1 周时间在确认源码后进行升级。
  7. 所有未升级到最新代码的节点被自动关掉。

By default configuration of the EOS.IO software, the process of updating the blockchain to add new features takes 2 to 3 months, while updates to fix non-critical bugs that do not require changes to the constitution can take 1 to 2 months.

### 紧急变更

The block producers may accelerate the process if a software change is required to fix a harmful bug or security exploit that is actively harming users. Generally speaking it could be against the constitution for accelerated updates to introduce new features or fix harmless bugs.

# 脚本 & 虚拟机

The EOS.IO software will be first and foremost a platform for coordinating the delivery of authenticated messages to accounts. The details of scripting language and virtual machine are implementation specific details that are mostly independent from the design of the EOS.IO technology. Any language or virtual machine that is deterministic and properly sandboxed with sufficient performance can be integrated with the EOS.IO software API.

## 模式定义的消息

All messages sent between accounts are defined by a schema which is part of the blockchain consensus state. This schema allows seamless conversion between binary and JSON representation of the messages.

## 模式定义的数据库

Database state is also defined using a similar schema. This ensures that all data stored by all applications is in a format that can be interpreted as human readable JSON but stored and manipulated with the efficiency of binary.

## 分离授权与应用

To maximize parallelization opportunities and minimize the computational debt associated with regenerating application state from the transaction log, EOS.IO software separates validation logic into three sections:

  1. 验证消息是否内部一致；
  2. 验证所有前提条件是否有效；
  3. 修改应用程序状态。

Validating the internal consistency of a message is read-only and requires no access to blockchain state. This means that it can be performed with maximum parallelism. Validating preconditions, such as required balance, is read-only and therefore can also benefit from parallelism. Only modification of application state requires write access and must be processed sequentially for each application.

Authentication is the read-only process of verifying that a message can be applied. Application is actually doing the work. In real time both calculations are required to be performed, however once a transaction is included in the blockchain it is no longer necessary to perform the authentication operations.

## 虚拟机独立架构

It is the intention of the EOS.IO software-based blockchain that multiple virtual machines can be supported and new virtual machines added over time as necessary. For this reason, this paper will not discuss the details of any particular language or virtual machine. That said, there are two virtual machines that are currently being evaluated for use with an EOS.IO software-based blockchain.

### Web 组建 (WASM)

Web Assembly is an emerging web standard for building high performance web applications. With a few small modifications Web Assembly can be made deterministic and sandboxed. The benefit of Web Assembly is the widespread support from industry and that it enables contracts to be developed in familiar languages such as C or C++.

Ethereum developers have already begun modifying Web Assembly to provide suitable sandboxing and determinism in with their [Ethereum flavored Web Assembly (WASM)](https://github.com/ewasm/design). This approach can be easily adapted and integrated with EOS.IO software.

### 以太访虚拟机 (EVM)

This virtual machine has been used for most existing smart contracts and could be adapted to work within an EOS.IO blockchain. It is conceivable that EVM contracts could be run within their own sandbox inside an EOS.IO software-based blockchain and that with some adaptation EVM contracts could communicate with other EOS.IO software blockchain applications.

# 跨链通信

EOS.IO software is designed to facilitate inter-blockchain communication. This is achieved by making it easy to generate proof of message existence and proof of message sequence. These proofs combined with an application architecture designed around message passing enables the details of inter-blockchain communication and proof validation to be hidden from application developers.

<img align="right" src="http://eos.io/wpimg/Diagram1.jpg" width="362.84px" height="500px" />

## 用于轻客户端的 Merkle 证明 (LCV)

Integrating with other blockchains is much easier if clients do not need to process all transactions. After all, an exchange only cares about transfers in and out of the exchange and nothing more. It would also be ideal if the exchange chain could utilize lightweight merkle proofs of deposit rather than having to trust its own block producers entirely. At the very least a chain's block producers would like to maintain the smallest possible overhead when synchronizing with another blockchain.

The goal of LCV is to enable the generation of relatively light-weight proof of existence that can be validated by anyone tracking a relatively light-weight data set. In this case the objective is to prove that a particular transaction was included in a particular block and that the block is included in the verified history of a particular blockchain.

Bitcoin supports validation of transactions assuming all nodes have access to the full history of block headers which amounts to 4MB of block headers per year. At 10 transactions per second, a valid proof requires about 512 bytes. This works well for a blockchain with a 10 minute block interval, but is no longer "light" for blockchains with a 3 second block interval.

The EOS.IO software enables lightweight proofs for anyone who has any irreversible block header after the point in which the transaction was included. Using the hash-linked structure shown below it is possible to prove the existence of any transaction with a proof less than 1024 bytes in size. If it is assumed that validating nodes are keeping up with all block headers in the past day (2 MB of data), then proving these transactions will only require proofs 200 bytes long.

There is little incremental overhead associated with producing blocks with the proper hash-linking to enable these proofs which means there is no reason not to generate blocks this way.

When it comes time to validate proofs on other chains there are a wide variety of time/ space/ bandwidth optimizations that can be made. Tracking all block headers (420 MB/year) will keep proof sizes small. Tracking only recent headers can offer a trade off between minimal long-term storage and proof size. Alternatively, a blockchain can use a lazy evaluation approach where it remembers intermediate hashes of past proofs. New proofs only have to include links to the known sparse tree. The exact approach used will necessarily depend upon the percentage of foreign blocks that include transactions referenced by merkle proof.

After a certain density of interconnectedness it becomes more efficient to simply have one chain contain the entire block history of another chain and eliminate the need for proofs all together. For performance reasons, it is ideal to minimize the frequency of inter-chain proofs.

## 跨链通信的延时

When communicating with another outside blockchain, block producers must wait until there is 100% certainty that a transaction has been irreversibly confirmed by the other blockchain before accepting it as a valid input. Using an EOS.IO software-based blockchain and DPOS with 3 second blocks and 21 producers, this takes approximately 45 seconds. If a chain's block producers do not wait for irreversibility it would be like an exchange accepting a deposit that was later reversed and could impact the validity of the blockchain's consensus.

## 完备性证明

When using merkle proofs from outside blockchains, there is a significant difference between knowing that all transactions processed are valid and knowing that no transactions have been skipped or omitted. While it is impossible to prove that all of the most recent transactions are known, it is possible to prove that there have been no gaps in the transaction history. The EOS.IO software facilitates this by assigning a sequence number to every message delivered to every account. A user can use these sequence numbers to prove that all messages intended for a particular account have been processed and that they were processed in order.

# 总结

The EOS.IO software is designed from experience with proven concepts and best practices, and represents fundamental advancements in blockchain technology. The software is part of a holistic blueprint for a globally scalable blockchain society in which decentralised applications can be easily deployed and governed.