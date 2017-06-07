# EOS.IO 技术白皮书

**草案：2017 年 6 月 5 日**

**摘要：** EOS.IO 软件引入一种新的区块链架构设计，它使得去中心化的应用可以横向和纵向的扩展。这通过构建一个仿操作系统的方式来实现，在它之上可以构建应用程序。该软件提供账户、身份验证、数据库、异步通信和跨越数百个 CPU 内核或集群的应用程序调度。由此产生的技术是一种区块链架构，它可以扩展至每秒处理百万级交易，消除用户的手续费，并且允许快速和轻松的部署去中心化的应用。

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
- [账户](#accounts)
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
  * [多账户的原子化交易](#atomic-transactions-with-multiple-accounts)
  * [区块链状态的部分评估](#partial-evaluation-of-blockchain-state)
  * [主观尽力而为调度](#subjective-best-effort-scheduling)
- [Token 模型与资源使用](#token-model-and-resource-usage)
  * [客观与主观的度量](#objective-and-subjective-measurements)
  * [Receiver Pays](#receiver-pays)
  * [Delegating Capacity](#delegating-capacity)
  * [Separating Transaction costs from Token Value](#separating-transaction-costs-from-token-value)
  * [State Storage Costs](#state-storage-costs)
  * [Block Rewards](#block-rewards)
  * [Community Benefit Applications](#community-benefit-applications)
- [Governance](#governance)
  * [Freezing Accounts](#freezing-accounts)
  * [Changing Account Code](#changing-account-code)
  * [Constitution](#constitution)
  * [Upgrading the Protocol & Constitution](#upgrading-the-protocol---constitution)
    + [Emergency Changes](#emergency-changes)
- [Scripts & Virtual Machines](#scripts---virtual-machines)
  * [Schema Defined Messages](#schema-defined-messages)
  * [Schema Defined Database](#schema-defined-database)
  * [Separating Authentication from Application](#separating-authentication-from-application)
  * [Virtual Machine Independent Architecture](#virtual-machine-independent-architecture)
    + [Web Assembly](#web-assembly)
    + [Ethereum Virtual Machine (EVM)](#ethereum-virtual-machine--evm-)
- [Inter Blockchain Communication](#inter-blockchain-communication)
  * [Merkle Proofs for Light Client Validation (LCV)](#merkle-proofs-for-light-client-validation--lcv-)
  * [Latency of Interchain Communication](#latency-of-interchain-communication)
  * [Proof of Completeness](#proof-of-completeness)
- [Conclusion](#conclusion)

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

# 账户

EOS.IO 软件允许所有的账户使用一个唯一的人类可读的名称来索引，长度在 2 到 32 个字符之间。这个名称由账户创建者自己选择。所有的账户必须在创建时用极少的账户余额来注资，从而覆盖存储账户信息的成本。账户名称也支持命名空间，比如 @domain 这个账户的拥有者是唯一可以创建 @user.domain 账户的人。

在一个去中心化的场景中，应用开发者将会为新用户注册成本买单。传统的企业已经为了获客而花费大量的前，比如广告、免费服务等。比起来，资助一个新的区块链账户的花费简直微不足道。值得庆幸的是，对一个已经在另一个应用注册过的用户并不需要再创建新的账户。

## 消息 & 处理

每个账户可以发送结构化的消息给其他的账户，并且可以定义脚本来处理他们接收到的消息。EOS.IO 软件给每个账户提供了只有自己的消息处理脚本能访问的私有数据库。消息处理脚本同样可以给其他账户发送消息。消息和自动化的消息处理的结合决定了 EOS.IO 如何定义智能合约的。

## 基于角色的权限管理

权限管理涉及判定一条消息是否被正确的授权。权限管理最简单的形式就是检查一个交易包含必须的签名，但这意味着必须的签名是已知的。一般情况下，权威必然是独立的个体或者个体组成的群体，并且是被划分开的。EOS.IO 软件提供了声明式的权限管理系统，通过管理谁可以在什么时间做什么来给用户细力度和高维度的控制。

授权和权限管理被标准化和脱离应用的商业逻辑是不可取的。这使得管理权限的工具得以被开发，既满足常规的需求又为性能优化提供了重要的可能性。

每一个账户可以被任何权重组合的其他账户和私钥管控。这创建了分层级的权利结构，这反映了现实中的权限分配方式，并且让多用户共同管理资产变得从未如此简单。多用户控制是安全最大的贡献者，并且，当用户使用得当，它可以极大的消除因被黑而导致被盗窃的风险。

EOS.IO 软件允许账户被定义为哪些密钥和／或其他账号的组合可以发送特定的消息类型给其他账户。举个例子，可以指定一个密钥给一个用户的社交媒体账号，同时另一个密钥访问交易所。甚至可以给其他账户权限来代表自己而无需分配给他们密钥。

### 命名的权限级别

<img align="right" src="http://eos.io/wpimg/diagram3.png" width="228.395px" height="300px" />

使用 EOS.IO 软件，账户可以定义

Using the EOS.IO software, accounts can define named permission levels each of which can be derived from higher level named permissions. Each named permission level defines an authority; an authority is a threshold multi-signature check consisting of keys and/or named permission levels of other accounts. For example, an account's "Friend" permission level can be set for the account to be controlled equally by any of the account's friends.

Another example is the Steem blockchain which has three hard-coded named permission levels: owner, active, and posting. The posting permission can only perform social actions such as voting and posting, while the active permission can do everything except change the owner.  The owner permission is meant for cold storage and is able to do everything.  EOS.IO generalizes this concept by allowing each account holder to define their own hierarchy as well as the grouping of actions.

### Named Message Handler Groups

The EOS.IO software allows each account to organize its own message handlers into named and nested groups. These named message handler groups can be referenced by other accounts when they configure their permission levels.

The highest level message handler group is the account name and the lowest level is the individual message type being received by the account. These groups can be referenced like so:  **@accountname.groupa.subgroupb.MessageType**.

Under this model it is possible for an exchange contract to group order creation and canceling separately from deposit and withdraw. This grouping by the exchange contract is a convenience for users of the exchange.

### Permission Mapping

EOS.IO software allows each account to define a mapping between a Named Message Handler Group of any account and their own Named Permission Level.  For example, an account holder could map the account holder's social media application to the account holder's "Friend" permission group. With this mapping, any friend could post as the account holder on the account holder's social media.  Even though they would post as the account holder, they would still use their own keys to sign the message. This means it is always possible to identify which friends used the account and in what way.

### Evaluating Permissions

When delivering a message of type "**Action**", from **@alice** to **@bob** the EOS.IO software will first check to see if **@alice** has defined a permission mapping for **@bob.groupa.subgroup.Action**.  If nothing is found then a mapping for **@bob.groupa.subgroup** then **@bob.groupa**, and lastly **@bob** will be checked. If no further match is found, then the assumed mapping will be to the named permission group **@alice.active**.  

Once a mapping is identified then signing authority is validated using the threshold multi-signature process and the authority associated with the named permission. If that fails, then it traverses up to the parent permission and ultimately to the owner permission, **@alice.owner**.  

<img align="center" src="http://eos.io/wpimg/diagram2grayscale2.jpg" width="845.85px" height="500px" />

#### Default Permission Groups

The technology also allows all accounts have an "owner" group which can do everything, and an "active" group which can do everything except change the owner group.  All other permission groups are derived from "active".  

#### Parallel Evaluation of Permissions

The permission evaluation process is "read-only" and changes to permissions made by transactions do not take effect until the end of a block. This means that all keys and permission evaluation for all transactions can be executed in parallel. Furthermore, this means that a rapid validation of permission is possible without starting the costly application logic that would have to be rolled back. Lastly, it means that transaction permissions can be evaluated as pending transactions are received and do not need to be re-evaluated as they are applied.

All things considered, permission verification represents a significant percentage of the computation required to validate transactions. Making this a read-only and trivially parallelizable process enables a dramatic increase in performance.

When replaying the blockchain to regenerate the deterministic state from the log of messages there is no need to evaluate the permissions again. The fact that a transaction is included in a known good block is sufficient to skip this step. This dramatically reduces the computational load associated with replaying an ever growing blockchain.

## Messages with Mandatory Delay

Time is a critical component of security. In most cases, it is not possible to know if a private key has been stolen until it has been used. Time based security is even more critical when people have applications that require keys be kept on computers connected to the internet for daily use.  The EOS.IO software enables application developers to indicate that certain messages must wait a minimum period of time after being included in a block before they can be applied. During this time they can be canceled.  

Users can then receive notice via email or text message when one of these messages is broadcast. If they did not authorize it, then they can use the account recovery process to recover their account and retract the message.

The required delay depends upon how sensitive an operation is. Paying for a coffee can have no delay and be irreversible in seconds, while buying a house may require a 72 hour clearing period. Transferring an entire account to new control may take up to 30 days. The exact delays chosen are up to application developers and users.

## Recovery from Stolen Keys

The EOS.IO software provides users a way to restore control of their account when their keys are stolen. An account owner can use any owner key that was active in the last 30 days along with approval from their designated account recovery partner to reset the owner key on their account. The account recovery partner cannot reset control of the account without the help of the owner.

There is nothing for the hacker to gain by attempting to go through the recovery process because they already "control" the account. Furthermore, if they did go through the process, the recovery partner would likely demand identification and multi-factor authentication (phone and email).  This would likely compromise the hacker or gain the hacker nothing in the process.

This process is also very different from a simple multi-signature arrangement. With a multi-signature transaction, there is another company that is party to every transaction that is executed, but with the recovery process the agent is only a party to the recovery process and has no power over the day-to-day transactions. This dramatically reduces costs and legal liabilities for everyone involved.

# Deterministic Parallel Execution of Applications

Blockchain consensus depends upon deterministic (reproducible) behavior. This means all parallel execution must be free from the use of mutexes or other locking primitives.  Without locks there must be some way to guarantee that all accounts can only read and write their own private database.  It also means that each account processes messages sequentially and that parallelism will be at the account level.  

Using the EOS.IO software, it is the job of the block producer to organize message delivery into independent threads so that they can be evaluated in parallel.  The state of each account depends only upon the messages delivered to it. The schedule is the output of a block producer and will be deterministically executed, but the process for generating the schedule need not be deterministic. This means that block producers can utilize parallel algorithms to schedule transactions.

Part of parallel execution means that when a script generates a new message it does not get delivered immediately, instead it is scheduled to be delivered in the next cycle. The reason it cannot be delivered immediately is because the receiver may be actively modifying its own state in another thread.

## Minimizing Communication Latency

Latency is the time it takes for one account to send a message to another account and then receive a response.  The goal is to enable two accounts to exchange messages back and forth within a single block without having to wait 3 seconds between each message. To enable this, the EOS.IO software divides each block into cycles.  Each cycle is divided into threads and each thread contains a list of transactions.  Each transaction contains a set of messages to be delivered. This structure can be visualized as a tree where alternating layers are processed sequentially and in parallel.

      Block

        Cycles (sequential)

          Threads (parallel)

            Transactions (sequential)

              Messages (sequential)

                Receiver and Notified Accounts (parallel)

Transactions generated in one cycle can be delivered in any subsequent cycle or block. Block producers will keep adding cycles to a block until the maximum wall clock time has passed or there are no new generated transactions to deliver.

It is possible to use static analysis of a block to verify that within a given cycle no two threads contain transactions that modify the same account. So long as that invariant is maintained a block can be processed by running all threads in parallel.

## Read-Only Message Handlers

Some accounts may be able to process a message on a pass/fail basis without modifying their internal state. If this is the case then these handlers can be executed in parallel so long as only read-only message handlers for a particular account are included in one or more threads within a particular cycle.

## Atomic Transactions with Multiple Accounts

Sometimes it is desirable to ensure that messages are delivered to and accepted by multiple accounts atomically. In this case both messages are placed in one transaction and both accounts will be assigned the same thread and the messages applied sequentially. This situation is not ideal for performance and when it comes to "billing" users for usage, they will get billed by the number of unique accounts referenced by a transaction.

For performance and cost reasons it is best to minimize atomic operations involving two or more heavily utilized accounts.  

## Partial Evaluation of Blockchain State

Scaling blockchain technology necessitates that components are modular. Everyone should not have to run everything, especially if they only need to use a small subset of the applications.

An exchange application developer runs full nodes for the purpose of displaying the exchange state to its users. This exchange application has no need for the state associated with social media applications. EOS.IO software allows any full node to pick any subset of applications to run. Messages delivered to other applications are safely ignored because an application's state is derived entirely from the messages that are delivered to it.  

This has some significant implications on communication with other accounts. Most significantly it cannot be assumed that the state of the other account is accessible on the same machine. It also means that while it is tempting to enable "locks" that allow one account to synchronously call another account, this design pattern breaks down if the other account is not resident in memory.

All state communication among accounts must be passed via messages included in the blockchain.

## Subjective Best Effort Scheduling

The EOS.IO software cannot obligate block producers to deliver any message to any other account. Each block producer makes their own subjective measurement of the computational complexity and time required to process a transaction. This applies whether a transaction is generated by a user or automatically by a script.

The EOS.IO software provides that at a network level all transactions are billed a fixed computational bandwidth cost regardless of whether it took .01ms or a full 10 ms to execute it. However, each individual block producer using the software may calculate resource usage using their own algorithm and measurements. When a block producer concludes that a transaction or account has consumed a disproportionate amount of the computational capacity they simply reject the transaction when producing their own block; however, they will still process the transaction if other block producers consider it valid.

In general so long as even 1 block producer considers a transaction as valid and under the resource usage limits then all other block producers will also accept it, but it may take up to 1 minute for the transaction to find that producer.

In some cases a producer may create a block that includes transactions that are an order of magnitude outside of acceptable ranges. In this case the next block producer may opt to reject the block and the tie will be broken by the third producer. This is no different than what would happen if a large block caused network propagation delays. The community would notice a pattern of abuse and eventually remove votes from the rogue producer.

This subjective evaluation of computational cost frees the blockchain from having to precisely and deterministically measure how long something takes to run. With this design there is no need to precisely count instructions which dramatically increases opportunities for optimization without breaking consensus.

# Token Model and Resource Usage

All blockchains are resource constrained and require a system to prevent abuse. With the EOS.IO software, there are three broad classes of resources that are consumed by applications:

1. Bandwidth and Log Storage (Disk);
2. Computation and Computational Backlog (CPU); and
3. State Storage (RAM).

Bandwidth and computation have two components, instantaneous usage and long-term usage. A blockchain maintains a log of all messages and this log is ultimately stored and downloaded by all full nodes. With the log of messages it is possible to reconstruct the state of all applications.

The computational debt is calculations that must be performed to regenerate state from the message log. If the computational debt grows too large then it becomes necessary to take snapshots of the blockchain's state and discard the blockchain's history. If computational debt grows too quickly then it may take 6 months to replay 1 year worth of transactions. It is critical, therefore, that the computational debt be carefully managed.

Blockchain state storage is information that is accessible from application logic. It includes information such as order books and account balances. If the state is never read by the application then it should not be stored. For example, blog post content and comments are not read by application logic so they should not be stored in the blockchain's state.  Meanwhile the existence of a post/comment, the number of votes, and other properties do get stored as part of the blockchain's state.

Block producers publish their available capacity for bandwidth, computation, and state. The EOS.IO software allows each account to consume a percentage of the available capacity proportional to the amount of tokens held in a 3-day staking contract. For example, if a blockchain based on the EOS.IO software is launched and if an account holds 1% of the total tokens distributable pursuant to that blockchain, then that account has the potential to utilize 1% of the state storage capacity.

Using the EOS.IO software, bandwidth and computational capacity are allocated on a fractional reserve basis because they are transient (unused capacity cannot be saved for future use). The algorithm used by EOS.IO is similar to the algorithm used by Steem to rate-limit bandwidth usage.

## Objective and Subjective Measurements

As discussed earlier, instrumenting computational usage has a significant impact on performance and optimization; therefore, all resource usage constraints are ultimately subjective and enforcement is done by block producers according to their individual algorithms and estimates.

That said, there are certain things that are trivial to measure objectively. The number of messages delivered and the size of the data stored in the internal database are cheap to measure objectively. The EOS.IO software enables block producers to apply the same algorithm over these objective measures but may choose to apply stricter subjective algorithms over subjective measurements.  

## Receiver Pays

Traditionally, it is the business that pays for office space, computational power, and other costs required to run the business. The customer buys specific products from the business and the revenue from those product sales is used to cover the business costs of operation. Similarly, no website obligates its visitors to make micropayments for visiting its website to cover hosting costs. Therefore, decentralized applications should not force its customers to pay the blockchain directly for the use of the blockchain.

The EOS.IO software does not require its users to pay directly to the blockchain for its use and therefore does not constrain or prevent t a business from determining its own monetization strategy for its products.

## Delegating Capacity

If a blockchain is launched using the EOS.IO software and tokens are held by a holder who may not have an immediate need to consume all or part of the available bandwidth, such holder can choose to give or rent the unconsumed bandwidth to others; the block producers running EOS.IO software will recognize this delegation of capacity and allocate bandwidth accordingly.

## Separating Transaction costs from Token Value

One of the major benefits of the EOS.IO software is that the amount of bandwidth available to an application is entirely independent of any token price. If an application owner holds a relevant number of tokens, then the application can run indefinitely within a fixed state and bandwidth usage. Developers and users are unaffected from any price volatility in the token market and therefore not reliant on a price feed. The EOS.IO software enables block producers to naturally increase bandwidth, computation, and storage available per token independent of the token's value.

The EOS.IO software awards block producers tokens every time they produce the block. The value of the tokens will impact the amount of bandwidth, storage, and computation a producer can afford to purchase; this model naturally leverages rising token values to increase network performance.

## State Storage Costs

While bandwidth and computation can be delegated, storage of application state will require an application developer to hold tokens until that state is deleted. If state is never deleted then the tokens are effectively removed from circulation.

Every user account requires a certain amount of storage; therefore, every account must maintain a minimum balance. As storage capacity of the network increases this minimum required balance will fall.

## Block Rewards

EOS.IO software awards new tokens to a block producer every time a block is produced.  The number of tokens created is determined by the median of the desired pay published by all block producers. The EOS.IO software may be configured to enforce a cap on producer awards such that the total annual increase in token supply does not exceed 5%.  

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
