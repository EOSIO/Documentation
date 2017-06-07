# EOS.IO 技术白皮书

**草案：2017 年 6 月 5 日**

**摘要：** EOS.IO 软件引入一种新的区块链架构设计，它使得去中心化的应用可以横向和纵向的扩展。这通过构建一个仿操作系统的方式来实现，在它之上可以构建应用程序。该软件提供帐户、身份验证、数据库、异步通信和跨越数百个 CPU 内核或集群的应用程序调度。由此产生的技术是一种区块链架构，它可以扩展至每秒处理百万级交易，消除用户的手续费，并且允许快速和轻松的部署去中心化的应用。

Copyright © 2017 block.one

未经允许，在非用于商业和教育用途的前提下（即，除了收取费用或商业目的），如果注明原始出处并适用声明的版权，任何人可以使用、复制或发布本白皮书内的任何内容。

**免责声明：** 本 EOS.IO 技术白皮书草案仅供参考。block.one 不保证本文结论的准确性，并且白皮书提供“是”没有任何陈述和保证，明示或暗示，任何，包括，但不限于：（i）保证的适销性，针对特定用途的适用性、标题或侵权；（ii）本白皮书的内容没有错误或合适与所有目的（iii）这样的内容不会侵犯第三方权利。所有的保证明确否认。block.one 及其子公司明确表示不承担使用造成的所有责任和赔偿，或依赖于包含在本白皮书的任何信息，即使告知此类损害的可能性。无论如何，block.one 或其附属机构有责任向任何人或实体的任何直接的、间接的、特殊的或间接的损害赔偿的使用，参考，或在该白皮书或任何内容所包含的依赖。

- [背景](#background)
- [区块链应用的要求](#requirements-for-blockchain-applications)
  * [支持成百上千的用户](#support-millions-of-users)
  * [免费的使用](#free-usage)
  * [简单升级和 bug 修复](#easy-upgrades-and-bug-recovery)
  * [低延时](#low-latency)
  * [时序性能](#sequential-performance)
  * [并发性能](#parallel-performance)
- [共识算法 (DPOS)](#consensus-algorithm--dpos-)
  * [交易确认](#transaction-confirmation)
  * [股权证明的交易 (TaPoS)](#transaction-as-proof-of-stake--tapos-)
- [帐户](#accounts)
  * [消息 & 处理](#messages---handlers)
  * [基于角色的权限管理](#role-based-permission-management)
    + [命名的权限级别](#named-permission-levels)
    + [命名的消息处理群组](#named-message-handler-groups)
    + [权限映射](#permission-mapping)
    + [评估权限](#evaluating-permissions)
      - [默认权限群组](#default-permission-groups)
      - [权限并行评估](#parallel-evaluation-of-permissions)
  * [带强制性延时的消息](#messages-with-mandatory-delay)
  * [恢复被盗窃的密钥](#recovery-from-stolen-keys)
- [应用程序的确定性并行执行](#deterministic-parallel-execution-of-applications)
  * [最小化通信延迟](#minimizing-communication-latency)
  * [只读信息的处理](#read-only-message-handlers)
  * [多帐户的原子化交易](#atomic-transactions-with-multiple-accounts)
  * [区块链状态的部分评估](#partial-evaluation-of-blockchain-state)
  * [自主最优调度](#subjective-best-effort-scheduling)
- [Token 模型与资源使用](#token-model-and-resource-usage)
  * [客观与主观的度量](#objective-and-subjective-measurements)
  * [接收方付费](#receiver-pays)
  * [委托能力](#delegating-capacity)
  * [分离交易成本与 Token 价值](#separating-transaction-costs-from-token-value)
  * [状态存储成本](#state-storage-costs)
  * [区块奖励](#block-rewards)
  * [社区效益应用](#community-benefit-applications)
- [治理](#governance)
  * [冻结帐户](#freezing-accounts)
  * [更改帐户代码](#changing-account-code)
  * [宪法](#constitution)
  * [升级协议 & 宪法](#upgrading-the-protocol---constitution)
    + [紧急变更](#emergency-changes)
- [脚本 & 虚拟机](#scripts---virtual-machines)
  * [模式定义的消息](#schema-defined-messages)
  * [模式定义的数据库](#schema-defined-database)
  * [分离授权与应用](#separating-authentication-from-application)
  * [虚拟机独立架构](#virtual-machine-independent-architecture)
    + [Web 组建](#web-assembly)
    + [以太访虚拟机 (EVM)](#ethereum-virtual-machine--evm-)
- [跨链通信](#inter-blockchain-communication)
  * [用于轻客户端的 Merkle 证明 (LCV)](#merkle-proofs-for-light-client-validation--lcv-)
  * [跨链通信的延时](#latency-of-interchain-communication)
  * [完备性证明](#proof-of-completeness)
- [结论](#conclusion)

# 背景

区块链技术是通过 2008 年诞生的比特币货币得以被认知，自从那之后企业家和开发者就不断的尝试推广这一技术，以便在单一的区块链平台上支持更为广泛的应用程序。

而一些区块链平台努力的支持可运作的去中心化应用，具体的应用比如 BitShares 去中心化交易所（2014）和 Steem 社交媒体平台（2016）已经成为每天被成千上万活跃用户重度使用的区块链。他们能做到这些，是通过性能的提升达到每秒处理上千交易，消除手续费和提供堪比已经存在的中心化服务的用户体验。

已存在的区块链平台承担着大量的交易费和有限的可计算能力，这都阻碍了区块链技术的大面积应用。

# 区块链应用的要求

为了赢得广泛的应用，构建在区块链之上的应用需要一个灵活性足以满足以下要求的平台：

## 支持成百上千的用户

像 Ebay、Uber、AirBnB 和 Facebook 这样企业，他们需要区块链技术能处理每日数以千万的活跃用户。在某些情况下，除非用户群体达到一个极庞大的量级否则应用并无用武之地，因此一个可以处理极其庞大用户的平台是至关重要的。

## 免费的使用

应用开发者需要提供给用户免费服务的灵活性；用户并不一定因为使用平台或从中受益就一定要付费。一个可以免费供用户使用的区块链平台或许将赢得更为广泛的使用。开发者和企业可以制订有效的货币化战略。

## 简单升级和 bug 修复

企业构建区块链基础的应用需要能够为应用增加新特性的灵活性。

所有非同凡响的软件都会受到 bug 的影响，即便是经过了最严格意义上的验证。这个平台必须具有足够的鲁棒性以便应对不可避免出现的 bug。

## 低延时

一个好的用户体验需要延时时间在数秒内就能收到可靠的反馈。高延时会阻碍用户，并且会让构建在区块链上的应用比已有的非区块链应用缺乏竞争力。

## 时序性能

一些应用因为顺序依赖关系的执行步骤而不能使用并发算法实现。比如交易所就需要足够的时序性能来处理很高的交易量，因此高时序性能处理的平台是必须的。

## 并发性能

大型可扩展应用需要将工作量分配到多 CPU 和计算机之上。

# 共识算法 (DPOS)

EOS.IO 软件使用唯一能满足区块链之上应用性能需求的去中心化共识算法，[委托股权证明（DPOS）](https://steemit.com/dpos/@dantheman/dpos-consensus-algorithm-this-missing-white-paper)。在这种算法中，持有区块链中 token 的人可以通过持续批准的投票系统选择区块生产者，任何人可以选择参与区块的生产，并且将按照其获得总票数在所有生产者获得票数的比例来赋予参与的机会。对于私有区块链管理人员可以使用这些 token 来添加和删除 IT 职员。

EOS.IO 软件使得区块准确的每 3 秒生成一个并且在任何时间点都只有一个被授权的生产者来生成区块。如果一个区块在规定时间之内未被生产出来则这一区块将被跳过。当一个或多个区块被跳过发生时，在区块链中会有一个 6 秒及以上的间隔。

在 EOS.IO 软件中，区块通过 21 名生产者轮流产生。在每一轮的开始时，21 个唯一的区块生产者被选出。获票最高的前 20 名自动在没轮被选中，剩余的一个生产者通过得票比例选出。被选中的生产者通过从区块取到的时间作为伪随机数来打乱其顺序，打乱顺序是为确保这些生产者与其他生产者保持均衡的连通性。

如果一个生产者错过了一个区块并且在过去的 24 小时内没有生产任何的区块，那么它将被从候选中移除，直到它在区块链中通知它要开始再次生产区块的意图。这样通过最小化区块丢失数量（因被证实不可靠的节点不作为导致）来确保网络操作的稳定性。

在一般情况下，一个 DPOS 区块链不会经历任何的分叉，因为区块生产者是通过合作而非竞争的方式来生产区块。即便真的出现了分叉，共识也将自动的切换到最长的链上。之所以会这样运作，是因为区块添加到一个区块链分叉的速率与公用同一共识的区块生产者比例是相关的。换句话说，具有更多生产者的区块链分叉会比拥有较少生产的那一个条增长的速度更快。而且，没有一个生产者会同时在两个分叉上同时生产区块。如果一个区块生产者被抓到做这样的事儿，那么这个生产者将很可能被投票投出。这些双重生产行为对应密码学凭证可以用来自动的删除这些滥用者。

## 交易确认

通常 DPOS 区块链 100% 会有区块生产者参与。一个交易从广播开始后平均 1.5 秒就可以 99.9% 被认为是确认了。

在一些特殊情况下例外，软件出现 bug，网络拥塞，或一个恶意的区块生产者制造了两个或更多的分叉。为了确保一个交易绝对是不可逆的，一个节点可以选择等待 21 个区块生产者中的 15 个给出确认。基于通常的 EOS.IO 软件配置，在一般情况下这需要平均 45 秒的时间。默认情况下，所有的节点将认为当 21 个生产者中有 15 个给出确认后这一区块就是不可逆的了，并且不管长度如何都不会切换到没有这一区块的分叉。

在分叉开始的 9 秒内，一个节点就可以警告用户他们极可能正处于分叉中。在连续丢失 2 个区块后，有 95% 的概率可以确认一个节点处于分叉中。在连续丢失 3 个区块后就有 99% 的概率确认。可以通过节点丢失、近期参与比率和其他参数来构建鲁棒性预测模型，从而快速的警告操作者出现了问题。

对于这种警告的反应完全取决于商业交易的性质，但最简单的做法就是等待 15/21 的确认直到警告消失。

## 股权证明的交易 (TaPoS)

EOS.IO 软件需要每一个交易包含最近一个区块头的哈希值。这个哈希值有两个目的：

1. 防止不包含区块引用的交易在分叉时重放发生；和
2. 通知网络对应的用户和他们的股份当前在某个具体的分叉上。

随着时间的推移，所有的用户直接确认区块链，在这一链条上难以伪造假的链条，因为假的链条根本无法从合法链条上迁移交易。

# 帐户

EOS.IO 软件允许所有的帐户使用一个唯一的人类可读的名称来索引，长度在 2 到 32 个字符之间。这个名称由帐户创建者自己选择。所有的帐户必须在创建时用极少的帐户余额来注资，从而覆盖存储帐户信息的成本。帐户名称也支持命名空间，比如 @domain 这个帐户的拥有者是唯一可以创建 @user.domain 帐户的人。

在一个去中心化的场景中，应用开发者将会为新用户注册成本买单。传统的企业已经为了获客而花费大量的前，比如广告、免费服务等。比起来，资助一个新的区块链帐户的花费简直微不足道。值得庆幸的是，对一个已经在另一个应用注册过的用户并不需要再创建新的帐户。

## 消息 & 处理

每个帐户可以发送结构化的消息给其他的帐户，并且可以定义脚本来处理他们接收到的消息。EOS.IO 软件给每个帐户提供了只有自己的消息处理脚本能访问的私有数据库。消息处理脚本同样可以给其他帐户发送消息。消息和自动化的消息处理的结合决定了 EOS.IO 如何定义智能合约的。

## 基于角色的权限管理

权限管理涉及判定一条消息是否被正确的授权。权限管理最简单的形式就是检查一个交易包含必须的签名，但这意味着必须的签名是已知的。一般情况下，权威必然是独立的个体或者个体组成的群体，并且是被划分开的。EOS.IO 软件提供了声明式的权限管理系统，通过管理谁可以在什么时间做什么来给用户细力度和高维度的控制。

授权和权限管理被标准化和脱离应用的商业逻辑是不可取的。这使得管理权限的工具得以被开发，既满足常规的需求又为性能优化提供了重要的可能性。

每一个帐户可以被任何权重组合的其他帐户和私钥管控。这创建了分层级的权利结构，这反映了现实中的权限分配方式，并且让多用户共同管理资产变得从未如此简单。多用户控制是安全最大的贡献者，并且，当用户使用得当，它可以极大的消除因被黑而导致被盗窃的风险。

EOS.IO 软件允许帐户被定义为哪些密钥和／或其他账号的组合可以发送特定的消息类型给其他帐户。举个例子，可以指定一个密钥给一个用户的社交媒体账号，同时另一个密钥访问交易所。甚至可以给其他帐户权限来代表自己而无需分配给他们密钥。

### 命名的权限级别

<img align="right" src="http://eos.io/wpimg/diagram3.png" width="228.395px" height="300px" />

在 EOS.IO 软件中，帐户可以定义命名的权限级别，每一个是由更高级别的命名权限派生而来。每一个命名的权限级别定义了一个权威；一个权威是多重签名阈值校验，它包含密钥和／或其他帐户的命名权限级别。打个比方，一个帐户的“朋友”权限级别可以被设置为由该帐户的任何一个朋友无差别的控制。

另一个例子在 Steem 区块链中，它包含三个硬编码的命名权限级别：拥有，活跃和发帖。发帖权限就只能进行如投票和发帖的社交活动，而活跃权限可以做除了变更拥有之外的所有的事情。拥有权限的意思是冷存储并且有能力做任何事。EOS.IO 通过允许帐户所有者定义他们自己的分级权限和行为的组合来推广这一概念。

### 命名的消息处理群组

EOS.IO 软件允许每个帐户将他们自己的消息组织到一个命名和嵌套的群组中。这个命名的消息处理群组可以在其他帐户配置他们权限级别时被引用。

最高级别的消息处理群组是帐户名称，最低级别的是一个帐户接收到的单独的消息类型。这些群组可以被这样的方式引用：
 **@accountname.groupa.subgroupb.MessageType**.

在这样的模型之下，交易所合约可以通过将挂单的创建和取消分组，从而与充值提现分离开。交易所合约的这样分组对用户而言带来了方便。

### 权限映射

EOS.IO 软件允许每个帐户定义从任意帐户的一个命名的消息处理群组与自己的命名的权限级别之间建立映射。举个例子，一个帐户所有者可以将自己社交媒体应用与自己的“朋友”权限群组建立映射。有了这个映射，任何朋友可以以这一帐户的身份在这一帐户的社交媒体上发帖。尽管他们将以帐户所有者的身份发帖，他们仍然使用自己的密钥来签名消息。这意味着总是可以辨识出是哪一个朋友在以何种方式使用帐户。

### 评估权限

当 **@alice** 以 "**Action**" 类型发送一条消息给 **@bob** 时，EOS.IO 软件首先会检查 **@alice** 是否为 **@bob.groupa.subgroup.Action** 定义过权限映射。如果什么都没有找到，紧接着检查 **@bob.groupa.subgroup** 映射，然后是 **@bob.groupa**，最后 **@bob** 将被检查。如果都没有找到，那么假定映射为命名的权限群组 **@alice.active**。

一旦一个映射被识别，则通过阈值多签名流程验证签名权威，并且关联权威与命名的权限。如果失败了，则跃迁至父权限，直至拥有者权限，**@alice.owner**。

<img align="center" src="http://eos.io/wpimg/diagram2grayscale2.jpg" width="845.85px" height="500px" />

#### 默认权限群组

这一技术同样允许所有的帐户有一个“拥有者”群组，可以做任何的事，并且还有一个“活动”群组可以做出了变更拥有者群组之外的所有事。所有其他的全新群组派生自“活动”群组。

#### 权限并行评估

权限评估过程是“只读”的，并且通过交易对权限的变更在一个区块结束之前不会起作用。这意味着对所有的交易对应的密钥和权限评估可以被并行执行。此外，这意味着一个快速的权限验证是可行的，它无需启动会引起回滚需求的高成本的应用逻辑。最后，这意味着交易权限可以被评估为待接收交易，

The permission evaluation process is "read-only" and changes to permissions made by transactions do not take effect until the end of a block. This means that all keys and permission evaluation for all transactions can be executed in parallel. Furthermore, this means that a rapid validation of permission is possible without starting the costly application logic that would have to be rolled back. Lastly, it means that transaction permissions can be evaluated as pending transactions are received and do not need to be re-evaluated as they are applied.

All things considered, permission verification represents a significant percentage of the computation required to validate transactions. Making this a read-only and trivially parallelizable process enables a dramatic increase in performance.

When replaying the blockchain to regenerate the deterministic state from the log of messages there is no need to evaluate the permissions again. The fact that a transaction is included in a known good block is sufficient to skip this step. This dramatically reduces the computational load associated with replaying an ever growing blockchain.

## 带强制性延时的消息

时间是安全中的一个关键组成部分。在大多数情况下，一个私钥在没有被使用前都无从知晓它是否被偷窃。当人们有需要密钥的应用在每天联网使用的电脑上运行时，基于时间的安全会更为重要。EOS.IO 软件让应用开发者可以指明消息必须在被加到一个区块之前等待最小的时间间隙。在这个时间段内消息可以被取消。

用户可以在消息广播出去后通过邮件或者文字消息的形式收到通知。如果他们没有授权，那么他们可以使用帐户恢复流程来恢复帐户，并收回消息。

这个必须的延时由操作敏感性决定。为一杯咖啡付款可以没有任何的延时，几秒之内就不可逆了，而购买一个房子也许需要 72 消失的结算期。转移整个帐户到一个新的控制可能需要长达 30 天。具体的延时选择由开发者和用户自己来做选择。

## 恢复被盗窃的密钥

EOS.IO 软件提供给用户一种找回自己失窃密钥控制权的方式。一个帐户的所有者可以使用过去 30 天任何活跃的拥有者密钥与事先指定的合作者帐户给出的批准来重置自己帐户的密钥。帐户的恢复合作者在没有所有人帮助的情况下无法重置帐户的控制权。

黑客尝试进行恢复流程是无意义的，因为他们已经“控制”了帐户。此外，就算他们真的进行这一流程，恢复合作者也会询问身份证明和多因素认证（手机和邮件）。这会让黑客脱作出让步或者无功而返。

这一流程与简单的多重签名有很大差异。在多重签名中，另一个公司要参与所有转账的执行，但在恢复流程中，它却只在恢复时才起作用对每天的转账无从干预。这大大的降低了参与者的成本和法律责任。

# 应用程序的确定性并行执行

区块链共识取决于确定性（可重现的）的行为。这意味着所有的并行计算必须是不能互斥或者具有其他锁特性的。没有了锁就必须有一些方式可以确保所有的帐户只可以读取和写入他们自己的私有数据库。这也意味着每个帐户处理消息是顺序的，而并发只能在帐户层面进行。

EOS.IO 软件中，将消息传递到不同的线程是区块生产者的职责，这样他们就可以被平行的评估。每个帐户的状态由且只由发送给它的消息决定。进度表由区块生产者输出并且会被确定性的执行，但是生成进度表的过程却不一定是确定性的。这意味着区块生产者可以使用并发算法来调度交易。

并行执行的一方面意味着当一个脚本生成了一个新的消息，它不会立即被发送，而被安排在下一个轮训中发送。不能立马发出的原因是接受者可能在另一个线程中活跃的变更自己的状态。

## 最小化通信延迟

延迟是一个帐户从发出一条消息给另一个帐户，直到收到回应的这段时间。我们的目标是在一个单独的区块中包含两个帐户交换消息的来去信息，而不用在每条消息间等待 3 秒钟。为了做到这一点，EOS.IO 软件将每个区块划分为循环。每个循环划分为线程，每个线程包含了交易的一个列表。每一个交易包含了待发送的消息集合。这个结构可以被可视化为一个树，其中交互层彼此并行，各自被顺序的执行。

      区块

        循环（顺序）

          线程（并行）

            交易（顺序）

              消息（顺序）

                接受者和被通知帐户（并行）

在一个循环中生成的交易可以在后续的任何一个循环或者区块中被发送。区块生产者会持续不断的向区块中添加循环直到最大的墙上时间到了或者没有更多的新交易要发送。

可以对一个区块使用静态分析来验证同一个循环内不存在两个线程包含同一帐户下对交易的变更。只要保持不变一个区块就可以并行的运行所有的线程。

## 只读消息的处理

有些帐户可以在传递/失败的基础上处理消息而不修改内部状态。如果是这样的话，那么这些处理程序可以并行执行，只要只有一个特定的帐户的只读消息处理程序包含在一个或多个线程在一个特定的周期。

## 多帐户的原子化交易

有时我们需要确保消息自动的被多个账户传递和接收。在这种情况下，消息会被放在同一个交易内，账户会被分配到同一个线程，并且消息被顺序的添加。这种情况对性能是不理想的，当用户使用涉及到“账单”时，他们将在交易内以账户唯一索引被列入其中。

基于性能和成本原因最好减少涉及两个或多个重度帐户的原子性操作。

## 区块链状态的部分评估

扩展区块链技术使得组件化成为必要。每个人不应该执行所有的事务，尤其是当其只需要运行应用的一个小的子集。

一个交易所应用开发者运行一个完整节点位的是为其用户展现所有的状态。这个交易所应用没有与社交网络建立关联的必要性。EOS.IO 软件允许任何的完整节点选择应用的任何子集来执行。传递给其他应用的消息可以被安全的忽略掉，因为应用程序的状态完全由传递给它的消息派生。

这与其他帐户的沟通有一些重要的影响。最重要的是，不能假定其他帐户的状态可以在同一台机器上访问。这也意味着，虽然很容易启用“锁”来允许一个帐户同步调用另一个帐户，如果其他帐户不驻留在内存中，这种设计模式就会出现问题。

所有账户帐户间的状态通信必须通过包含在区块链中的消息进行。

## 自主最优调度

EOS.IO 软件并不能为区块生产生者为任何其他帐户送达的任何信息负责。每个区块生产者要对计算的发杂读和处理一个消息的时间自己进行主观上的预测。这同时适用于用户生成的和脚本自动生成的交易。

EOS.IO 软件在网络层面通过所有列出的交易给出固定计算带宽成本，无论它是需要 .01ms 还是足足 10ms 来执行。然而，每个单独的区块生产者要通过自己的算法来计算资源的消耗。当一个区块生产者断定一个交易或者帐户消耗了不相称的大量的计算资源时，他们可以在生成自己的区块时拒绝该交易；但是，如果其他区块生产者认为交易是有效的，他们就仍需要处理交易。

一般而言，只要一个区块生产者认为交易在资源使用限度内是有效的，那么其他区块生产者就也要接受，但可能交易传递给生产者就要花费 1 分钟。

在某些情况下，生产者可以创建包含可接受范围之外的数量级的块。在这种情况下，下一个区块生产者可能会选择拒绝区块和束缚将被第三个生产者打破。这和因为区块过大导致的网络延时没什么打不同。社区会注意到模式的异常并最终会将票从流氓生产者哪里删掉。

这种对计算成本的主观评估将区块链从必须精确和确定的预测一些东西要花多长时间来运行这一问题中解放出来。有了这一设计就不需要精确的数指令，将极大的增加优化的可能性又不必打破共识。

# Token 模型与资源使用

所有的区块链都受资源约束并且需要一个系统来防止滥用。EOS.IO 中，有三个宽泛的类别的资源供应用程序消耗：

1. 带宽和日志存储（磁盘）；
2. 计算与计算储备（中央处理器）；
3. 状态存储（内存）；

带宽和计算有两部分，瞬时使用和长期使用。一个区块链维持着所有消息的日志，这些日志最终由完全节点存储和下载。通过消息日志可以重现所有应用的状态。

可计算债务是一个必须通过消息日志重新构建状态的计算结果。如果可计算债务增长变得臃肿则有必要通过快照方式记录区块链状态，并丢弃区块链历史。如果可计算债务增长过快，则它需要花费 6 个月时间来重放等值与 1 年的交易。这很不可取，因此，可计算债务需要被细心的管理。

区块链状态存储是通过访问应用逻辑获取的信息。它包括诸如挂单和账户余额等信息。如果状态从未被应用读取则它不会被存储。比如，博客发布的内容和评论如未被应用逻辑读取则他们就不应该存储在区块链状态中。同时，发布的内容／评论的存在、投票的数量和其他属性要作为区块链状态的部分被存储下来。

区块生产者对外发布她们可用的带宽，计算能力和状态。EOS.IO 允许帐户按比例消耗一个 3 天对赌合约中的可用资源。举个例子，如果一个基于 EOS.IO 的区块链启动了，一个帐户持有所有 token 发行总量的 1%，那么帐号就具有使用 1% 状态存储空间的能力。

EOS.IO 软件中，带宽和计算能力分配在部分准备的基础中，因为他们是瞬态的（未使用的容量不能为了之后使用而存储下来）。EOS.IO 中使用的算法类似于 Steem 中限制带宽使用而用的算法。

## 客观与主观的度量

如前所述，检测计算使用的性能和优化的影响很大；因此，所有资源的使用限制，最终都是主观的，执行依靠个人的算法和区块生产者进行估计。

也就是说，有一些事情是微不足道的客观衡量。发送的消息数和存储在内部数据库中的数据的大小是便宜的客观衡量。的 EOS.IO 软件让区块生产者采用相同的算法应对客观的量，但可以在主观量上选择采用更严格的主观测量算法。

## 接收方付费

传统上来说，企业为办公场地、计算力和其他为了运行企业而需要的成本买单。客户从企业购买具体的产品，产品销售产生的利润来盖过企业运作的成本。类似的，没有哪个网站要求来访者为盖过运作成本而支付。因此，去中心化应用也不应该强制用户因为使用了区块链而直接为区块链支付。

EOS.IO 软件不需要其用户为使用区块链而付费，因此不限制或阻止企业确定其产品自身的盈利策略。

## 委托能力

如果一个区块链使用 EOS.IO 软件启动，一个 token 持有者也许并不想立即消耗所有或者部分可用带宽，这个持有者可以选择将可用带宽送给或者租给其他人；运行 EOS.IO 的区块生产者将根据识别到的被委托方的能力给其分配对应的带宽。

## 分离交易成本与 Token 价值

EOS.IO 软件的一个主要优点就是应用可用的带宽完全独立于 token 的价格。如果一个应用所有者持有相应数量的 token，那么应用就可以在固定的状态和带宽使用下运行。开发者和用户不会收到 token 市场价格波动的任何影响，因此不依赖于价格反馈。EOS.IO 软件可以依据 token 让区块生产者自然的增加带宽、计算力和存储空间，而与 token 价值彼此独立。

EOS.IO 软件在区块生产者每次生产区块时给予其奖励。token 的值将影响其能购买的带宽、存储和计算资源；这一模型会自然的利用 token 值的上涨来增加网络的性能。

## 状态存储成本

由于带宽和计算资源可以被委托，因此应用的状态存储需要应用程序的开发者持有 token 直到状态被删除。如果状态永远不会被删除那么 token 实质上从流通中被抹除。

每一个用户帐户需要一个确定数量的存储；因此每一个帐户必须保持一个最小的余额。随着网络存储能力的不断提升，余额的最小余额需求将会下降。

## 块奖励

每次区块生产生造出新的区块时 EOS.IO 软件会给予其 token 奖励。奖励的 token 数量由所有的区块生产者发布的期望回报取中值得到。

EOS.IO 软件可以配置限定生产者回报的上限从而确保 token 的每年增长比例不会超过 5%。

## Community Benefit Applications

In addition to electing block producers, based on the EOS.IO software, users can elect 3 community benefit applications also known as smart contracts. These 3 applications will receive tokens of up to a configured percent of the token supply per annum minus the tokens that have been paid to block producers. These smart contracts will receive tokens proportional to the votes each application has received from token holders. The elected applications or smart contracts can be replaced by newly elected applications or smart contracts by token holders.

# Governance

Governance is the process by which people reach consensus on subjective matters that cannot be captured entirely by software algorithms. The EOS.IO software implements a governance process that efficiently directs the existing influence of block producers. Absent a defined governance process, prior blockchains relied ad hoc, informal, and often controversial governance processes that result in unpredictable outcomes.

The EOS.IO software recognizes that power originates with the token holders who delegate that power to the block producers. The block producers are given limited and checked authority to freeze accounts, update defective applications, and propose hard forking changes to the underlying protocol.    

Part of the EOS.IO software is the election of block producers. Before any change can be made to the blockchain these block producers must approve it. If the block producers refuse to make changes desired by the token holders then they can be voted out. If the block producers make changes without permission of the token holders then all other non-producing full-node validators (exchanges, etc) will reject the change.

## Freezing Accounts

Sometimes a smart contact behaves in an aberrant or unpredictable manner and fails to perform as intended; other times an application or account may discover an exploit that enables it to consume an unreasonable amount of resources. When such issues inevitably occur, the block producers have the power to rectify such situations.

The block producers on all blockchains have the power to select which transactions are included in blocks which gives them the ability to freeze accounts.  EOS.IO software formalizes this authority by subjecting the process of freezing an account to a 17/21 vote of active producers. If the producers abuse the power they can be voted out and an account will be unfrozen.

## Changing Account Code

When all else fails and an "unstoppable application" acts in an unpredictable manner, the EOS.IO software allows the block producers to replace the account's code without hard forking the entire blockchain. Similar to the process of freezing an account, this replacement of the code requires a 17/21 vote of elected block producers.

## Constitution

The EOS.IO software enables blockchains to establish a peer-to-peer terms of service agreement or a binding contract among those users who sign it, referred to as a "constitution". The content of this constitution defines obligations among the users which cannot be entirely enforced by code and facilitates dispute resolution by establishing jurisdiction and choice of law along with other mutually accepted rules. Every transaction broadcast on the network must incorporate the hash of the constitution as part of the signature and thereby explicitly binds the signer to the contract.

The constitution also defines the human-readable intent of the source code protocol. This intent is used to identify the difference between a bug and a feature when errors occur and guides the community on what fixes are proper or improper.   

## Upgrading the Protocol & Constitution

The EOS.IO software define a process by which the protocol as defined by the canonical source code and its constitution, can be updated using the following process:

1. Block producers propose a change to the constitution and obtains 17/21 approval.
2. Block producers maintain 17/21 approval for 30 consecutive days.
3. All users are required to sign transactions using the hash of the new constitution.
4. Block producers adopt changes to the source code to reflect the change in the constitution and propose it to the blockchain using the hash of a git commit.
5. Block producers maintain 17/21 approval for 30 consecutive days.
6. Changes to the code take effect 7 days later, giving all full nodes 1 week to upgrade after ratification of the source code.
7. All nodes that do not upgrade to the new code shut down automatically.

By default configuration of the EOS.IO software, the process of updating the blockchain to add new features takes 2 to 3 months, while updates to fix non-critical bugs that do not require changes to the constitution can take 1 to 2 months.

### Emergency Changes

The block producers may accelerate the process if a software change is required to fix a harmful bug or security exploit that is actively harming users. Generally speaking it could be against the constitution for accelerated updates to introduce new features or fix harmless bugs.

# Scripts & Virtual Machines

The EOS.IO software will be first and foremost a platform for coordinating the delivery of authenticated messages to accounts. The details of scripting language and virtual machine are implementation specific details that are mostly independent from the design of the EOS.IO technology.  Any language or virtual machine that is deterministic and properly sandboxed with sufficient performance can be integrated with the EOS.IO software API.

## Schema Defined Messages

All messages sent between accounts are defined by a schema which is part of the blockchain consensus state. This schema allows seamless conversion between binary and JSON representation of the messages.  

## Schema Defined Database

Database state is also defined using a similar schema. This ensures that all data stored by all applications is in a format that can be interpreted as human readable JSON but stored and manipulated with the efficiency of binary.

## Separating Authentication from Application

To maximize parallelization opportunities and minimize the computational debt associated with regenerating application state from the transaction log, EOS.IO separates validation logic into three sections:

1. Validating that a message is internally consistent;
2. Validating that all preconditions are valid; and
3. Modifying the application state.

Validating the internal consistency of a message is read-only and requires no access to blockchain state. This means that it can be performed with maximum parallelism. Validating preconditions, such as required balance, is read-only and therefore can also benefit from parallelism. Only modification of application state requires write access and must be processed sequentially for each application.



Authentication is the read-only process of verifying that a message can be applied. Application is actually doing the work. In real time both calculations are required to be performed, however once a transaction is included in the blockchain it is no longer necessary to perform the authentication operations.

## Virtual Machine Independent Architecture

It is the intention of the EOS.IO software that multiple virtual machines can be supported and new virtual machines added over time as necessary. For this reason, this paper will not discuss the details of any particular language or virtual machine. That said, there are two virtual machines that are currently being evaluated for use within EOS.IO.

### Web Assembly (WASM)

Web Assembly is an emerging web standard for building high performance web applications. With a few small modifications Web Assembly can be made deterministic and sandboxed. The benefit of Web Assembly is the widespread support from industry and that it enables contracts to be developed in familiar languages such as C or C++.

Ethereum developers have already begun modifying Web Assembly to provide suitable sandboxing and determinism in with their [Ethereum flavored Web Assembly (WASM)](https://github.com/ewasm/design). This approach can be easily adapted and integrated with EOS.IO software.  

### Ethereum Virtual Machine (EVM)

This virtual machine has been used for most existing smart contracts and could be adapted to work within an EOS.IO blockchain.  It is conceivable that EVM contracts could be run within their own sandbox inside an EOS.IO blockchain and that with some adaptation EVM contracts could communicate with other EOS.IO blockchain applications.

# Inter Blockchain Communication

EOS.IO software is designed to facilitate inter-blockchain communication. This is achieved by making it easy to generate proof of message existence and proof of message sequence. These proofs combined with an application architecture designed around message passing enables the details of inter-blockchain communication and proof validation to be hidden from application developers.

<img align="right" src="http://eos.io/wpimg/Diagram1.jpg" width="362.84px" height="500px" />

## Merkle Proofs for Light Client Validation (LCV)

Integrating with other blockchains is much easier if clients do not need to process all transactions.  After all, an exchange only cares about transfers in and out of the exchange and nothing more.  It would also be ideal if the exchange chain could utilize lightweight merkle proofs of deposit rather than having to trust its own block producers entirely. At the very least a chain's block producers would like to maintain the smallest possible overhead when synchronizing with another blockchain.

The goal of LCV is to enable the generation of relatively light-weight proof of existence that can be validated by anyone tracking a relatively light-weight data set. In this case the objective is  to prove that a particular transaction was included in a particular block and that the block is included in the verified history of a particular blockchain.  

Bitcoin supports validation of transactions assuming all nodes have access to the full history of block headers which amounts to 4MB of block headers per year. At 10 transactions per second, a valid proof requires about 512 bytes. This works well for a blockchain with a 10 minute block interval, but is no longer "light" for blockchains with a 3 second block interval.  
 
The EOS.IO software enables lightweight proofs for anyone who has any irreversible block header after the point in which the transaction was included. Using the hash-linked structure shown below it is possible to prove the existence of any transaction with a proof less than 1024 bytes in size.  If it is  assumed that validating nodes are keeping up with all block headers in the past day (2 MB of data), then proving these transactions will only require proofs 200 bytes long.

There is little incremental overhead associated with producing blocks with the proper hash-linking to enable these proofs which means there is no reason not to generate blocks this way.

When it comes time to validate proofs on other chains there are a wide variety of time/ space/ bandwidth optimizations that can be made. Tracking all block headers (420 MB/year) will keep proof sizes small.  Tracking only recent headers can offer a trade off between minimal long-term storage and proof size. Alternatively, a blockchain can use a lazy evaluation approach where it remembers intermediate hashes of past proofs. New proofs only have to include links to the known sparse tree. The exact approach used will necessarily depend upon the percentage of foreign blocks that include transactions referenced by merkle proof.  

After a certain density of interconnectedness it becomes more efficient to simply have one chain contain the entire block history of another chain and eliminate the need for proofs all together. For performance reasons, it is ideal to minimize the frequency of inter-chain proofs.

## Latency of Interchain Communication   

When communicating with another outside blockchain, block producers must wait until there is 100% certainty that a transaction has been irreversibly confirmed by the other blockchain before accepting it as a valid input. Using EOS.IO software and DPOS with 3 second blocks and 21 producers, this takes approximately 45 seconds.  If a chain's block producers do not wait for irreversibility it would be like an exchange accepting a deposit that was later reversed and could impact the validity of the a chain's consensus.

## Proof of Completeness

When using merkle proofs from outside blockchains, there is a significant difference between knowing that all transactions processed are valid and knowing that no transactions have been skipped or omitted.  While it is impossible to prove that all of the most recent transactions are known, it is possible to prove that there have been no gaps in the transaction history.  The EOS.IO software facilitates this by assigning a sequence number to every message delivered to every account. A user can use these sequence numbers to prove that all messages intended for a particular account have been processed and that they were processed in order.

# Conclusion

The EOS.IO software is designed from experience with proven concepts and best practices, and represents fundamental advancements in blockchain technology. The software is part of a holistic blueprint for a globally scalable blockchain society in which decentralised applications can be easily deployed and governed.
