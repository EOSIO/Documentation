# EOS.IO 기술 백서

**초안 작성일: 2017년 6월 26일, 번역: 이태민 (taeminlee), 감수: 조재우 (@clayop (https://steemit.com/@clayop))**

**초록:** EOS.IO 소프트웨어는 탈중앙화 애플리케이션의 수직 및 수평 확장이 가능하도록 디자인된 새로운 블록체인 아키텍처를 소개합니다. 이는 애플리케이션을 구축할 수 있는 운영체제와 유사한 구조를 생성함으로 완성됩니다. 본 소프트웨어는 수백 개의 CPU 코어 또는 클러스터에 계정(accounts), 인증(authentication), 데이터베이스(databases), 비동기 통신(asynchronous communication), 애플리케이션 스케쥴링(application scheduling) 기능을 제공합니다. 그 결과 초당 수백만 건의 트랜잭션 처리 능력을 갖추면서도, 수수료가 없고, 빠르고 쉽게 애플리케이션을 개발할 수 있는 블록체인 아키텍처 기술이 탄생했습니다.

**PLEASE NOTE: CRYPTOGRAPHIC TOKENS REFERRED TO IN THIS WHITE PAPER REFER TO CRYPTOGRAPHIC TOKENS ON A LAUNCHED BLOCKCHAIN THAT ADOPTS THE EOS.IO SOFTWARE. THEY DO NOT REFER TO THE ERC-20 COMPATIBLE TOKENS BEING DISTRIBUTED ON THE ETHEREUM BLOCKCHAIN IN CONNECTION WITH THE EOS TOKEN DISTRIBUTION.**

저작권 소유 © 2017 block.one

누구든지 허가 없이 원래의 출처와 해당 저작권 고지가 언급된 경우 비영리적이고 교육적인 용도 (즉, 유료 또는 상업적 목적 이외의 목적)로 본 백서의 자료를 사용, 복제 또는 배포할 수 있습니다.

**면책 조항:** EOS.IO 기술 백서 초안은 오직 정보 제공의 목적으로써 제공됩니다. block.one does not guarantee the accuracy of or the conclusions reached in this white paper, and this white paper is provided “as is”. block.one does not make and expressly disclaims all representations and warranties, express, implied, statutory or otherwise, whatsoever, including, but not limited to: (i) warranties of merchantability, fitness for a particular purpose, suitability, usage, title or noninfringement; (ii) that the contents of this white paper are free from error; and (iii) that such contents will not infringe third-party rights. block.one and its affiliates shall have no liability for damages of any kind arising out of the use, reference to, or reliance on this white paper or any of the content contained herein, even if advised of the possibility of such damages. In no event will block.one or its affiliates be liable to any person or entity for any damages, losses, liabilities, costs or expenses of any kind, whether direct or indirect, consequential, compensatory, incidental, actual, exemplary, punitive or special for the use of, reference to, or reliance on this white paper or any of the content contained herein, including, without limitation, any loss of business, revenues, profits, data, use, goodwill or other intangible losses.

- [탄생 배경 (Background)](#background)
- [블록체인 애플리케이션의 요구사항 (Requirements for Blockchain Application)](#requirements-for-blockchain-applications) 
  - [수백만의 사용자 허용 (Support Millions of Users)](#support-millions-of-users)
  - [무료 사용 (Free Usage)](#free-usage)
  - [간편한 업그레이드 및 버그 해소 (Easy upgrades and Bug Recovery)](#easy-upgrades-and-bug-recovery)
  - [짧은 지연 시간 (Low Latency)](#low-latency)
  - [순차(sequential) 처리 성능 (Sequential Performance)](#sequential-performance)
  - [병렬 처리 성능 (Parallel Performance)](#parallel-performance)
- [합의 알고리즘 (DPOS) (Consensus Algorithm)](#consensus-algorithm--dpos-) 
  - [트랜잭션 확인 (Transaction Confirmation)](#transaction-confirmation)
  - [트랜잭션 기반 지분 증명 (Transaction as Proof of Stake, TaPoS)](#transaction-as-proof-of-stake--tapos-)
- [계정 (Accounts)](#accounts) 
  - [메시지와 처리기 (Messages & Handlers)](#messages---handlers)
  - [역할 기반 권한 관리 (Role Based Permission Management)](#role-based-permission-management) 
    - [명명된 권한 수준 (Named Permission Levels)](#named-permission-levels)
    - [명명된 메시지 처리기 그룹 (Named Message Handler Groups)](#named-message-handler-groups)
    - [권한 매핑 (Permission Mapping)](#permission-mapping)
    - [권한 검사 (Evaluating Permissions)](#evaluating-permissions) 
      - [기본 권한 그룹( Default Permission Groups)](#default-permission-groups)
      - [권한 검사의 병렬화 (Parallel Evaluation of Permissions)](#parallel-evaluation-of-permissions)
  - [메시지의 필수 지연 시간 (Messages with Mandatory Delay)](#messages-with-mandatory-delay)
  - [키 도난 상태에서의 복구 (Recovery from Stolen Keys)](#recovery-from-stolen-keys)
- [애플리케이션의 결정론적 병렬 실행 (Deterministic Parallel Execution of Applications)](#deterministic-parallel-execution-of-applications) 
  - [통신 지연 최소화 (Minimizing Communication Latency)](#minimizing-communication-latency)
  - [읽기 전용 메시지 처리기 (Read-Only Message Handlers)](#read-only-message-handlers)
  - [다중 계정의 원자적 트랜잭션 (Atomic Transactions with Multiple Accounts)](#atomic-transactions-with-multiple-accounts)
  - [블록체인 상태의 부분 검사 (Partial Evaluation of Blockchain State)](#partial-evaluation-of-blockchain-state)
  - [주관적 최선 스케쥴링 (Subjective Best Effort Scheduling)](#subjective-best-effort-scheduling)
- [토큰 모델과 리소스 사용 (Token Model and Resource Usage)](#token-model-and-resource-usage) 
  - [객관적 측정과 주관적 측정 (Objective and Subjective Measurements)](#objective-and-subjective-measurements)
  - [수취인 부담 (Receiver Pays)](#receiver-pays)
  - [리소스 허용량 위임 (Delegating Capacity)](#delegating-capacity)
  - [토큰의 가치와 트랜잭션 비용의 분리 (Separating Transaction costs from Token Value)](#separating-transaction-costs-from-token-value)
  - [상태 저장 비용 (State Storage Costs)](#state-storage-costs)
  - [블록 보상 (Block Rewards)](#block-rewards)
  - [커뮤니티 혜택 애플리케이션 (Community Benefit Applications)](#community-benefit-applications)
- [거버넌스 (Governance)](#governance) 
  - [계정 동결 (Freezing Accounts)](#freezing-accounts)
  - [계정 코드 변경 (Changing Account Code)](#changing-account-code)
  - [약관 (Constitution)](#constitution)
  - [프로토콜과 약관의 개정 (Upgrading the Protocol & Constitution)](#upgrading-the-protocol---constitution) 
    - [응급 변경 (Emergency Changes)](#emergency-changes)
- [스크립트와 가상 머신 (Scripts & Virtual Machines)](#scripts---virtual-machines) 
  - [스키마 정의 메시지 (Schema Defined Messages)](#schema-defined-messages)
  - [스키마 정의 데이터베이스 (Schema Defined Database)](#schema-defined-database)
  - [애플리케이션과 인증 분리 (Separating Authentication from Application)](#separating-authentication-from-application)
  - [가상 머신 독립 아키텍처 (Virtual Machine Independent Architecture)](#virtual-machine-independent-architecture) 
    - [웹어셈블리 (WASM; Web Assembly)](#web-assembly)
    - [이더리움 가상 머신 (EVM; Ethereum Virtual Machine)](#ethereum-virtual-machine--evm-)
- [블록체인 간 통신 (Inter Blockchain Communication)](#inter-blockchain-communication) 
  - [경량화된 클라이언트 검증(LCV)을 위한 머클 증명 (Merkle Proofs for Light Client Validation)](#merkle-proofs-for-light-client-validation--lcv-)
  - [체인 간 통신의 지연 시간 (Latency of Interchain Communication)](#latency-of-interchain-communication)
  - [완전성 증명 (Proof of Completeness)](#proof-of-completeness)
- [결론 (Conclusion)](#conclusion)

# 탄생 배경 (Background)

블록체인 기술은 2008년 비트코인 화폐의 출현과 함께 시작되었으며, 이후 기업가와 개발자는 하나의 블록체인 플랫폼에서 다양한 애플리케이션을 지원하기 위해 기술의 일반화를 시도해 왔습니다.

다수의 블록체인 플랫폼은 제대로 작동하는 탈중앙화 애플리케이션을 지원하기 위해 노력하는 동안, BitShares 탈중앙화 거래소(2014) 및 Steem 소셜 미디어 플랫폼(2016)과 같은 애플리케이션 특화 블록체인은 이미 수만 명의 일일 사용자를 가진 블록체인으로 성장하였습니다. 이는 초당 수천 건의 트랜잭션 지원과 1.5초의 지연시간과 같은 성능 향상, 사용 수수료의 제거, 현재 서비스되는 중앙 집중형 서비스와 유사한 수준의 사용자 경험 제공을 통해 가능하게 되었습니다.

현존하는 블록체인 플랫폼들은 비싼 수수료와 연산능력의 한계 때문에 블록체인의 광범위한 사용에 있어서 어려움을 겪고 있습니다.

# 블록체인 애플리케이션의 요구사항 (Requirements for Blockchain Application)

블록체인 위에서 돌아가는 애플리케이션이 대중적으로 사용되기 위해서는 다음의 요구사항을 만족하는 유연한 플랫폼을 갖춰야 합니다.

## 수백만의 사용자 허용 (Support Millions of Users)

Ebay, Uber, AirBnB, Facebook 과 같은 기존 서비스와 경쟁력을 갖추기 위해서 수천만의 일일 사용자를 수용할 수 있는 블록체인 기술이 필요합니다. 또한 많은 사용자가 이용하지 않을 경우 작동하지 않는 애플리케이션도 있으므로 많은 사용자를 수용하는 플랫폼은 무엇보다 중요합니다.

## 무료 사용 (Free Usage)

Application developers need the flexibility to offer users free services; users should not have to pay in order to use the platform or benefit from its services. 사용자가 무료로 이용할 수 있는 블록체인 플랫폼이 더 널리 전파될 것입니다. 빠른 대중화로 인해 기업가와 개발자는 효율적인 수익 창출 전략을 만들어 낼 수 있을 것입니다.

## 간편한 업그레이드 및 버그 해소 (Easy upgrades and Bug Recovery)

블록체인 기반 애플리케이션을 만드는 기업은 그들의 애플리케이션에 새로운 기능을 추가하고 개선할 수 있어야 합니다.

많은 소프트웨어들은 엄격한 공식적인 검사를 진행함에도 버그가 발생합니다. 플랫폼은 애플리케이션에서 버그가 발생하였을 때 버그를 수정할 수 있을 만큼 안정적이어야 합니다.

## 짧은 지연 시간 (Low Latency)

좋은 사용자 경험은 수 초 이하의 지연시간을 통한 안정적인 피드백을 필요로 합니다. 긴 지연 시간은 사용자의 불만을 일으키며, 이러한 블록체인 애플리케이션은 블록체인을 사용하지 않는 기성 시장의 애플리케이션에 비해 경쟁력이 떨어집니다.

## 순차(sequential) 처리 성능 (Sequential Performance)

몇몇 애플리케이션은 순차적인 처리 단계를 거쳐야 하기 때문에 병렬 알고리즘으로 구현될 수 없습니다. 거래소(exchange)와 같은 애플리케이션들은 많은 양의 거래를 순차적으로 처리하는 충분한 성능을 요구하므로, 플랫폼은 빠른 순차 처리 성능이 필요합니다.

## 병렬 처리 성능 (Parallel Performance)

거대 규모의 애플리케이션은 하나의 작업을 다수의 CPU와 컴퓨터에 분배하여 처리할 수 있어야 합니다.

# 합의 알고리즘 (DPOS) (Consensus Algorithm)

EOS.IO 소프트웨어는 블록체인 애플리케이션의 성능 요구사항을 충족할 수 있는 유일한 탈중앙화 합의 알고리즘인 [지분 위임 증명(DPOS; Deleteged Proof-Of-Stake)을 사용합니다](https://steemit.com/dpos/@dantheman/dpos-consensus-algorithm-this-missing-white-paper). Under this algorithm, those who hold tokens on a blockchain adopting the EOS.IO software may select block producers through a continuous approval voting system and anyone may choose to participate in block production and will be given an opportunity to produce blocks proportional to the total votes they have received relative to all other producers. For private blockchains the management could use the tokens to add and remove IT staff.

EOS.IO 소프트웨어는 정확히 3초마다 블록이 만들어질 수 있게 하며, 각 시점마다 오직 한 명의 블록 생산자만이 블록을 생성할 수 있습니다. 만약 정해진 시간에 블록이 생산되지 않을 경우 해당 시점의 블록은 무시됩니다. 1개 혹은 그 이상의 블록이 무시될 경우 블록체인에는 6초 혹은 그 이상의 간격(gap)이 나타납니다.

EOS.IO 소프트웨어를 이용하여 블록들은 21번의 단계로 구성되는 라운드로 생성되며. 각 라운드가 시작될 때 21명의 블록 생산자가 정해집니다. 라운드마다 많은 득표를 받은 상위 20명의 블록 생산자가 자동으로 배정되며, 마지막 생산자는 다른 생산자와의 상대적인 투표수에 비례하여 선출됩니다. 블록 시간으로 유도되는 의사 난수(pseudorandom number)에 따라 선출된 생산자들의 블록 생성 순서를 랜덤하게 섞습니다. 블록 생성 순서를 섞는 것은 모든 생산자가 다른 생산자들과 균형적인 연결(balanced connectivity)을 유지하도록 진행합니다.

생산자가 블록 생성에 실패하고 지난 24시간 동안 어떠한 블록을 생성하지 않는다면, 블록체인에 블록 생성 참여 의사를 알려주기 전까지 후보군에서 제외됩니다. 신뢰할 수 없는 사람을 참가시키지 않으므로 놓치는 블록의 수를 최소화하고 네트워크가 원활하게 동작하도록 보장합니다.

일반적인 상황에서 지분 위임 증명(DPOS) 알고리즘을 사용하는 블록체인은 어떠한 포크(fork)도 일어나지 않습니다. 이는 블록 생성자가 경쟁이 아닌 협력을 하기 때문입니다. 포크가 일어난 경우, 합의 알고리즘은 자동으로 가장 긴 블록 체인(chain)을 선택합니다. 이 방법이 동작하는 이유는, 특정 블록체인 포크에 블록들이 추가되는 속도는 같은 합의를 공유하는 블록 생성자의 비율과 직접 연관되기 때문입니다. 다수의 생산자 존재하는 블록체인 포크는 적은 생산자를 가진 것에 비하여 빠르게 증가합니다. 추가로, 어떠한 블록 생성자도 동시에 두 개의 포크에 블록을 생성할 수 없습니다. 이러한 경우가 적발될 경우 해당 블록 생성자는 탄핵당할 것입니다. 이러한 이중 생산에 대한 암호학적 증거(cryptographic evidence)는 정당하지 않은 방법으로 이득을 취한 사람을 자동으로 제거하는 데 사용될 수 있습니다.

## 트랜잭션 확인 (Transaction Confirmation)

일반적인 DPOS 블록체인은 100%의 블록 생산자 참여율을 가집니다. 전파(braodcasting) 시간부터 평균 1.5초의 시간이 흐르면 트랜잭션은 99.9%의 신뢰도로 확인(confirm)되었다 판단할 수 있습니다.

소프트웨어 버그, 인터넷 속도 저하, 비정상적 블록 생산자와 같은 특수한 상황에서 두 개 혹은 그 이상의 포크가 생성될 수 있습니다. 트랜잭션의 바뀌지 않음(irreversible)을 확정하기 위해서, 노드는 21명의 블록 생산자 중 15명의 확인(confirmation)을 기다릴 수 있습니다. EOS.IO 소프트웨어의 기본 설정에 따르면 보통 상황에서 45초의 시간이 소요됩니다. 기본적으로 모든 노드는 21명 중 15명이 확인할 경우 해당 블록이 바뀌지 않음을 확정하고 블록의 길이와 상관없이 다른 포크로 전환하지 않습니다.

노드는 포크가 분기된 후 9초 이내에 속한 노드가 소수 포크(minority fork)에 속해있음을 높은 확률로 사용자에게 알려줄 수 있습니다. 노드가 속한 블록체인에 2개의 블록이 연속적으로 추가되지 않을 경우 95% 확률로 소수 포크에 속하게 됩니다. 3번 연속으로 추가되지 않을 경우 99%의 소수 포크 확률을 가집니다. 블록이 Qkwls 노드, 최근 참여 비율 등의 정보를 토대로 관리자에게 잘못된 상황에 대한 경보를 신속하게 제공하는 안정적인 예측 모형을 만들 수 있습니다.

경보에 대한 대응은 비즈니스 트랜잭션의 성격에 따라 다르며, 가장 간단한 대응 방법은 15/21 확인이 이루어져 경보가 끝나는 시점까지 기다리는 것입니다.

## 트랜잭션 기반 지분 증명 (Transaction as Proof of Stake, TaPoS)

EOS.IO 소프트웨어는 모든 트랜잭션이 최근 블록 헤더의 해쉬값을 포함하도록 요구합니다. 해쉬 값은 두 가지 용도로 사용됩니다.

  1. 참조 블록(referenced block)이 포함되지 않은 포크에서 트랜잭션이 재실행 되는 것을 방지합니다.
  2. 특정 사용자가 가진 자산이 어떤 포크에서 존재하는지 네트워크에 알려줍니다.

시간이 지날수록 모든 사용자는 직접 블록체인을 확인(confirm)하게 되며, 합법적 체인의 거래를 위조 체인으로 옮길 수 없으므로 위조 체인을 만드는 것은 어렵게 됩니다.

# 계정 (Accounts)

EOS.IO 소프트웨어는 모든 계정이 2~32글자의 읽을 수 있는 고유 이름으로 참조되도록 합니다. 계정 이름은 생성자가 선택합니다. 모든 계정은 생성되는 시점에 계정 정보를 담는 저장 비용을 초과하는 잔액을 보유해야 합니다. 계정 이름은 네임스페이스를 지원합니다. 예를 들어, @domain 계정의 소유자만이 @user.domain을 생성할 수 있도록 합니다.

탈중앙화 환경에서, 애플리케이션 개발자는 새로운 사용자가 가입하여 계정을 생성하는 최소한의 비용을 부담해야 합니다. Traditional businesses already spend significant sums of money per customer they acquire in the form of advertising, free services, etc. 이와 비교할 때 새로운 블록체인 계정에 부담하는 비용은 상대적으로 미미할 것입니다. 이미 다른 애플리케이션에 가입한 사용자의 계정을 중복하여 가입할 필요는 없습니다.

## 메시지와 처리기 (Messages & Handlers)

한 계정에서 다른 계정으로 구조화된 메시지(structured message)를 전송할 수 있으며, 이를 송신하였을 때 처리하는 스크립트(script)를 정의할 수 있습니다. EOS.IO 소프트웨어는 각각의 계정이 독립된 프라이빗 데이터베이스를 가지도록 하며, 자신의 메시지 처리기만이 접근하도록 허용합니다. 메시지 처리 스크립트에서 다른 계정으로 메시지를 전송할 수도 있습니다. EOS.IO는 스마트 컨트렉트(smart contract)를 메시지와 자동화된 메시지 처리기의 조합으로 정의합니다.

## 역할 기반 권한 관리 (Role Based Permission Management)

권한 관리는 메시지가 정상적으로 인증(authorized)되었는지 결정하는 것을 포함됩니다. 가장 단순한 형태의 권한 관리는 트랜잭션의 서명 여부를 검사하는 것이나, 이는 이미 필요한 서명을 알고 있을 때 가능합니다. 일반적으로 권한은 사용자와 그룹에 부여되며 이는 종종 구분됩니다. EOS.IO 소프트웨어는 계정별로 누가, 언제, 어떤 작업을 할 수 있는지에 대하여 세부적으로 제어하는 선언적 권한 관리 시스템을 제공합니다.

인증과 권한 관리를 표준화하고 애플리케이션의 비즈니스 로직과 분리하는 것이 중요합니다. 이것은 권한 관리가 범용적인 관점에서 이루어지도록 하는 도구를 개발할 수 있게 하며, 또한 성능 최적화의 큰 가능성이 열리게 합니다.

모든 계정은 다른 계정들(other accounts)과 개인키들(private keys)의 가중치 조합(weighted combination)으로 제어될 수 있습니다. 이를 통해 현실의 권한 구성 방식과 유사한 위계적 인증 구조(hierarchical authority structure)를 생성할 수 있으며, 또한 기금(funds)에 대한 다중 사용자 제어(multi-user control)를 보다 쉽게 하도록 합니다. Multi-user control is the single biggest contributor to security, and, when used properly, it can greatly eliminate the risk of theft due to hacking.

EOS.IO software allows accounts to define what combination of keys and/or accounts can send a particular message type to another account. For example, it is possible to have one key for a user's social media account and another for access to the exchange. It is even possible to give other accounts permission to act on behalf of a user's account without assigning them keys.

### 명명된 권한 수준 (Named Permission Levels)

<img align="right" src="http://eos.io/wpimg/diagram3.png" width="228.395px" height="300px" />

Using the EOS.IO software, accounts can define named permission levels each of which can be derived from higher level named permissions. Each named permission level defines an authority; an authority is a threshold multi-signature check consisting of keys and/or named permission levels of other accounts. For example, an account's "Friend" permission level can be set for the account to be controlled equally by any of the account's friends.

Another example is the Steem blockchain which has three hard-coded named permission levels: owner, active, and posting. The posting permission can only perform social actions such as voting and posting, while the active permission can do everything except change the owner. The owner permission is meant for cold storage and is able to do everything. The EOS.IO software generalizes this concept by allowing each account holder to define their own hierarchy as well as the grouping of actions.

### 명명된 메시지 처리기 그룹 (Named Message Handler Groups)

The EOS.IO software allows each account to organize its own message handlers into named and nested groups. These named message handler groups can be referenced by other accounts when they configure their permission levels.

The highest level message handler group is the account name and the lowest level is the individual message type being received by the account. These groups can be referenced like so: **@accountname.groupa.subgroupb.MessageType**.

Under this model it is possible for an exchange contract to group order creation and canceling separately from deposit and withdraw. This grouping by the exchange contract is a convenience for users of the exchange.

### 권한 매핑 (Permission Mapping)

EOS.IO software allows each account to define a mapping between a Named Message Handler Group of any account and their own Named Permission Level. For example, an account holder could map the account holder's social media application to the account holder's "Friend" permission group. With this mapping, any friend could post as the account holder on the account holder's social media. Even though they would post as the account holder, they would still use their own keys to sign the message. This means it is always possible to identify which friends used the account and in what way.

### 권한 검사 (Evaluating Permissions)

When delivering a message of type "**Action**", from **@alice** to **@bob** the EOS.IO software will first check to see if **@alice** has defined a permission mapping for **@bob.groupa.subgroup.Action**. If nothing is found then a mapping for **@bob.groupa.subgroup** then **@bob.groupa**, and lastly **@bob** will be checked. If no further match is found, then the assumed mapping will be to the named permission group **@alice.active**.

Once a mapping is identified then signing authority is validated using the threshold multi-signature process and the authority associated with the named permission. If that fails, then it traverses up to the parent permission and ultimately to the owner permission, **@alice.owner**.

<img align="center" src="http://eos.io/wpimg/diagram2grayscale2.jpg" width="845.85px" height="500px" />

#### 기본 권한 그룹( Default Permission Groups)

The EOS.IO technology also allows all accounts to have an "owner" group which can do everything, and an "active" group which can do everything except change the owner group. All other permission groups are derived from "active".

#### 권한 검사의 병렬화 (Parallel Evaluation of Permissions)

The permission evaluation process is "read-only" and changes to permissions made by transactions do not take effect until the end of a block. This means that all keys and permission evaluation for all transactions can be executed in parallel. Furthermore, this means that a rapid validation of permission is possible without starting the costly application logic that would have to be rolled back. Lastly, it means that transaction permissions can be evaluated as pending transactions are received and do not need to be re-evaluated as they are applied.

All things considered, permission verification represents a significant percentage of the computation required to validate transactions. Making this a read-only and trivially parallelizable process enables a dramatic increase in performance.

When replaying the blockchain to regenerate the deterministic state from the log of messages there is no need to evaluate the permissions again. The fact that a transaction is included in a known good block is sufficient to skip this step. This dramatically reduces the computational load associated with replaying an ever growing blockchain.

## 메시지의 필수 지연 시간 (Messages with Mandatory Delay)

Time is a critical component of security. In most cases, it is not possible to know if a private key has been stolen until it has been used. Time based security is even more critical when people have applications that require keys be kept on computers connected to the internet for daily use. The EOS.IO software enables application developers to indicate that certain messages must wait a minimum period of time after being included in a block before they can be applied. During this time they can be cancelled.

Users can then receive notice via email or text message when one of these messages is broadcast. If they did not authorize it, then they can use the account recovery process to recover their account and retract the message.

The required delay depends upon how sensitive an operation is. Paying for a coffee can have no delay and be irreversible in seconds, while buying a house may require a 72 hour clearing period. Transferring an entire account to new control may take up to 30 days. The exact delays chosen are up to application developers and users.

## 키 도난 상태에서의 복구 (Recovery from Stolen Keys)

The EOS.IO software provides users a way to restore control of their account when their keys are stolen. An account owner can use any owner key that was active in the last 30 days along with approval from their designated account recovery partner to reset the owner key on their account. The account recovery partner cannot reset control of the account without the help of the owner.

There is nothing for the hacker to gain by attempting to go through the recovery process because they already "control" the account. Furthermore, if they did go through the process, the recovery partner would likely demand identification and multi-factor authentication (phone and email). This would likely compromise the hacker or gain the hacker nothing in the process.

This process is also very different from a simple multi-signature arrangement. With a multi-signature transaction, there is another company that is party to every transaction that is executed, but with the recovery process the agent is only a party to the recovery process and has no power over the day-to-day transactions. This dramatically reduces costs and legal liabilities for everyone involved.

# 애플리케이션의 결정론적 병렬 실행 (Deterministic Parallel Execution of Applications)

Blockchain consensus depends upon deterministic (reproducible) behavior. This means all parallel execution must be free from the use of mutexes or other locking primitives. Without locks there must be some way to guarantee that all accounts can only read and write their own private database. It also means that each account processes messages sequentially and that parallelism will be at the account level.

In an EOS.IO software-based blockchain, it is the job of the block producer to organize message delivery into independent threads so that they can be evaluated in parallel. The state of each account depends only upon the messages delivered to it. The schedule is the output of a block producer and will be deterministically executed, but the process for generating the schedule need not be deterministic. This means that block producers can utilize parallel algorithms to schedule transactions.

Part of parallel execution means that when a script generates a new message it does not get delivered immediately, instead it is scheduled to be delivered in the next cycle. The reason it cannot be delivered immediately is because the receiver may be actively modifying its own state in another thread.

## 통신 지연 최소화 (Minimizing Communication Latency)

Latency is the time it takes for one account to send a message to another account and then receive a response. The goal is to enable two accounts to exchange messages back and forth within a single block without having to wait 3 seconds between each message. To enable this, the EOS.IO software divides each block into cycles. Each cycle is divided into threads and each thread contains a list of transactions. Each transaction contains a set of messages to be delivered. This structure can be visualized as a tree where alternating layers are processed sequentially and in parallel.

        블록(Block)
    
          사이클(Cycles) (순차 처리)
    
            쓰레드(Threads) (병렬 처리)
    
              트랜잭션(Transactions) (순차 처리)
    
                메시지(Messages) (순차 처리)
    
                  수신자 및 계정 알림(Receiver and Notified Accounts) (병렬 처리)
    

Transactions generated in one cycle can be delivered in any subsequent cycle or block. Block producers will keep adding cycles to a block until the maximum wall clock time has passed or there are no new generated transactions to deliver.

It is possible to use static analysis of a block to verify that within a given cycle no two threads contain transactions that modify the same account. So long as that invariant is maintained a block can be processed by running all threads in parallel.

## 읽기 전용 메시지 처리기 (Read-Only Message Handlers)

Some accounts may be able to process a message on a pass/fail basis without modifying their internal state. If this is the case then these handlers can be executed in parallel so long as only read-only message handlers for a particular account are included in one or more threads within a particular cycle.

## 다중 계정의 원자적 트랜잭션 (Atomic Transactions with Multiple Accounts)

Sometimes it is desirable to ensure that messages are delivered to and accepted by multiple accounts atomically. In this case both messages are placed in one transaction and both accounts will be assigned the same thread and the messages applied sequentially. This situation is not ideal for performance and when it comes to "billing" users for usage, they will get billed by the number of unique accounts referenced by a transaction.

For performance and cost reasons it is best to minimize atomic operations involving two or more heavily utilized accounts.

## 블록체인 상태의 부분 검사 (Partial Evaluation of Blockchain State)

Scaling blockchain technology necessitates that components are modular. Everyone should not have to run everything, especially if they only need to use a small subset of the applications.

An exchange application developer runs full nodes for the purpose of displaying the exchange state to its users. This exchange application has no need for the state associated with social media applications. EOS.IO software allows any full node to pick any subset of applications to run. Messages delivered to other applications are safely ignored because an application's state is derived entirely from the messages that are delivered to it.

This has some significant implications on communication with other accounts. Most significantly it cannot be assumed that the state of the other account is accessible on the same machine. It also means that while it is tempting to enable "locks" that allow one account to synchronously call another account, this design pattern breaks down if the other account is not resident in memory.

All state communication among accounts must be passed via messages included in the blockchain.

## 주관적 최선 스케쥴링 (Subjective Best Effort Scheduling)

The EOS.IO software cannot obligate block producers to deliver any message to any other account. Each block producer makes their own subjective measurement of the computational complexity and time required to process a transaction. This applies whether a transaction is generated by a user or automatically by a script.

On a launched blockchain adopting the EOS.IO software, at a network level all transactions are billed a fixed computational bandwidth cost regardless of whether it took .01ms or a full 10 ms to execute it. However, each individual block producer using the software may calculate resource usage using their own algorithm and measurements. When a block producer concludes that a transaction or account has consumed a disproportionate amount of the computational capacity they simply reject the transaction when producing their own block; however, they will still process the transaction if other block producers consider it valid.

In general so long as even 1 block producer considers a transaction as valid and under the resource usage limits then all other block producers will also accept it, but it may take up to 1 minute for the transaction to find that producer.

In some cases a producer may create a block that includes transactions that are an order of magnitude outside of acceptable ranges. In this case the next block producer may opt to reject the block and the tie will be broken by the third producer. This is no different than what would happen if a large block caused network propagation delays. The community would notice a pattern of abuse and eventually remove votes from the rogue producer.

This subjective evaluation of computational cost frees the blockchain from having to precisely and deterministically measure how long something takes to run. With this design there is no need to precisely count instructions which dramatically increases opportunities for optimization without breaking consensus.

# 토큰 모델과 리소스 사용 (Token Model and Resource Usage)

**PLEASE NOTE: CRYPTOGRAPHIC TOKENS REFERRED TO IN THIS WHITE PAPER REFER TO CRYPTOGRAPHIC TOKENS ON A LAUNCHED BLOCKCHAIN THAT ADOPTS THE EOS.IO SOFTWARE. THEY DO NOT REFER TO THE ERC-20 COMPATIBLE TOKENS BEING DISTRIBUTED ON THE ETHEREUM BLOCKCHAIN IN CONNECTION WITH THE EOS TOKEN DISTRIBUTION.**

All blockchains are resource constrained and require a system to prevent abuse. With a blockchain that uses EOS.IO software, there are three broad classes of resources that are consumed by applications:

  1. 대역폭과 로그 저장소 (디스크)
  2. 연산과 연산 로그 (CPU)
  3. 상태 저장소 (램)

Bandwidth and computation have two components, instantaneous usage and long-term usage. A blockchain maintains a log of all messages and this log is ultimately stored and downloaded by all full nodes. With the log of messages it is possible to reconstruct the state of all applications.

The computational debt is calculations that must be performed to regenerate state from the message log. If the computational debt grows too large then it becomes necessary to take snapshots of the blockchain's state and discard the blockchain's history. If computational debt grows too quickly then it may take 6 months to replay 1 year worth of transactions. It is critical, therefore, that the computational debt be carefully managed.

Blockchain state storage is information that is accessible from application logic. It includes information such as order books and account balances. If the state is never read by the application then it should not be stored. For example, blog post content and comments are not read by application logic so they should not be stored in the blockchain's state. Meanwhile the existence of a post/comment, the number of votes, and other properties do get stored as part of the blockchain's state.

Block producers publish their available capacity for bandwidth, computation, and state. The EOS.IO software allows each account to consume a percentage of the available capacity proportional to the amount of tokens held in a 3-day staking contract. For example, if a blockchain based on the EOS.IO software is launched and if an account holds 1% of the total tokens distributable pursuant to that blockchain, then that account has the potential to utilize 1% of the state storage capacity.

Adopting the EOS.IO software on a launched blockchain means bandwidth and computational capacity are allocated on a fractional reserve basis because they are transient (unused capacity cannot be saved for future use). The algorithm used by EOS.IO software is similar to the algorithm used by Steem to rate-limit bandwidth usage.

## 객관적 측정과 주관적 측정 (Objective and Subjective Measurements)

As discussed earlier, instrumenting computational usage has a significant impact on performance and optimization; therefore, all resource usage constraints are ultimately subjective and enforcement is done by block producers according to their individual algorithms and estimates.

That said, there are certain things that are trivial to measure objectively. The number of messages delivered and the size of the data stored in the internal database are cheap to measure objectively. The EOS.IO software enables block producers to apply the same algorithm over these objective measures but may choose to apply stricter subjective algorithms over subjective measurements.

## 수취인 부담 (Receiver Pays)

Traditionally, it is the business that pays for office space, computational power, and other costs required to run the business. The customer buys specific products from the business and the revenue from those product sales is used to cover the business costs of operation. Similarly, no website obligates its visitors to make micropayments for visiting its website to cover hosting costs. Therefore, decentralized applications should not force its customers to pay the blockchain directly for the use of the blockchain.

A launched blockchain that uses the EOS.IO software does not require its users to pay the blockchain directly for its use and therefore does not constrain or prevent a business from determining its own monetization strategy for its products.

## 리소스 허용량 위임 (Delegating Capacity)

A holder of tokens on a blockchain launched adopting the EOS.IO software who may not have an immediate need to consume all or part of the available bandwidth, can give or rent such unconsumed bandwidth to others; the block producers running EOS.IO software on such blockchain will recognize this delegation of capacity and allocate bandwidth accordingly.

## 토큰의 가치와 트랜잭션 비용의 분리 (Separating Transaction costs from Token Value)

One of the major benefits of the EOS.IO software is that the amount of bandwidth available to an application is entirely independent of any token price. If an application owner holds a relevant number of tokens on a blockchain adopting EOS.IO software, then the application can run indefinitely within a fixed state and bandwidth usage. In such case, developers and users are unaffected from any price volatility in the token market and therefore not reliant on a price feed. In other words, a blockchain that adopts the EOS.IO software enables block producers to naturally increase bandwidth, computation, and storage available per token independent of the token's value.

A blockchain using EOS.IO software also awards block producers tokens every time they produce a block. The value of the tokens will impact the amount of bandwidth, storage, and computation a producer can afford to purchase; this model naturally leverages rising token values to increase network performance.

## 상태 저장 비용 (State Storage Costs)

While bandwidth and computation can be delegated, storage of application state will require an application developer to hold tokens until that state is deleted. If state is never deleted then the tokens are effectively removed from circulation.

Every user account requires a certain amount of storage; therefore, every account must maintain a minimum balance. As storage capacity of the network increases this minimum required balance will fall.

## 블록 보상 (Block Rewards)

A blockchain that adopts the EOS.IO software will award new tokens to a block producer every time a block is produced. In these circumstances, the number of tokens created is determined by the median of the desired pay published by all block producers. The EOS.IO software may be configured to enforce a cap on producer awards such that the total annual increase in token supply does not exceed 5%.

## 커뮤니티 혜택 애플리케이션 (Community Benefit Applications)

In addition to electing block producers, pursuant to a blockchain based on the EOS.IO software, users can elect 3 community benefit applications also known as smart contracts. These 3 applications will receive tokens of up to a configured percent of the token supply per annum minus the tokens that have been paid to block producers. These smart contracts will receive tokens proportional to the votes each application has received from token holders. The elected applications or smart contracts can be replaced by newly elected applications or smart contracts by token holders.

# 거버넌스 (Governance)

Governance is the process by which people reach consensus on subjective matters that cannot be captured entirely by software algorithms. An EOS.IO software-based blockchain implements a governance process that efficiently directs the existing influence of block producers. Absent a defined governance process, prior blockchains relied ad hoc, informal, and often controversial governance processes that result in unpredictable outcomes.

A blockchain based on the EOS.IO software recognizes that power originates with the token holders who delegate that power to the block producers. The block producers are given limited and checked authority to freeze accounts, update defective applications, and propose hard forking changes to the underlying protocol.

Embedded into the EOS.IO software is the election of block producers. Before any change can be made to the blockchain these block producers must approve it. If the block producers refuse to make changes desired by the token holders then they can be voted out. If the block producers make changes without permission of the token holders then all other non-producing full-node validators (exchanges, etc) will reject the change.

## 계정 동결 (Freezing Accounts)

Sometimes a smart contact behaves in an aberrant or unpredictable manner and fails to perform as intended; other times an application or account may discover an exploit that enables it to consume an unreasonable amount of resources. When such issues inevitably occur, the block producers have the power to rectify such situations.

The block producers on all blockchains have the power to select which transactions are included in blocks which gives them the ability to freeze accounts. A blockchain using EOS.IO software formalizes this authority by subjecting the process of freezing an account to a 17/21 vote of active producers. If the producers abuse the power they can be voted out and an account will be unfrozen.

## 계정 코드 변경 (Changing Account Code)

When all else fails and an "unstoppable application" acts in an unpredictable manner, a blockchain using EOS.IO software allows the block producers to replace the account's code without hard forking the entire blockchain. Similar to the process of freezing an account, this replacement of the code requires a 17/21 vote of elected block producers.

## 약관 (Constitution)

The EOS.IO software enables blockchains to establish a peer-to-peer terms of service agreement or a binding contract among those users who sign it, referred to as a "constitution". The content of this constitution defines obligations among the users which cannot be entirely enforced by code and facilitates dispute resolution by establishing jurisdiction and choice of law along with other mutually accepted rules. Every transaction broadcast on the network must incorporate the hash of the constitution as part of the signature and thereby explicitly binds the signer to the contract.

The constitution also defines the human-readable intent of the source code protocol. This intent is used to identify the difference between a bug and a feature when errors occur and guides the community on what fixes are proper or improper.

## 프로토콜과 약관의 개정 (Upgrading the Protocol & Constitution)

The EOS.IO software defines a process by which the protocol as defined by the canonical source code and its constitution, can be updated using the following process:

  1. 블록 생산자들은 약관의 개정을 제안하고 17/21 승인을 받습니다.
  2. 블록 생산자들은 17/21 승인을 30일간 유지합니다.
  3. 모든 사용자는 새 약관의 해시를 사용하여 거래에 서명해야 합니다.
  4. 블록 생산자들은 약관의 변화를 반영하도록 소스 코드의 변경을 채택하며, git 커밋의 해시값을 이용하여 블록체인에 제안합니다.
  5. 블록 생산자들은 17/21 승인을 30일간 유지합니다.
  6. 코드 변경은 7일간의 소스코드 적용 유예기간을 주며, 7일이 지난 이후 적용됩니다.
  7. 새 코드로 판올림하지 않은 노드는 강제로 종료됩니다.

By default configuration of the EOS.IO software, the process of updating the blockchain to add new features takes 2 to 3 months, while updates to fix non-critical bugs that do not require changes to the constitution can take 1 to 2 months.

### 응급 변경 (Emergency Changes)

The block producers may accelerate the process if a software change is required to fix a harmful bug or security exploit that is actively harming users. Generally speaking it could be against the constitution for accelerated updates to introduce new features or fix harmless bugs.

# 스크립트와 가상 머신 (Scripts & Virtual Machines)

The EOS.IO software will be first and foremost a platform for coordinating the delivery of authenticated messages to accounts. The details of scripting language and virtual machine are implementation specific details that are mostly independent from the design of the EOS.IO technology. Any language or virtual machine that is deterministic and properly sandboxed with sufficient performance can be integrated with the EOS.IO software API.

## 스키마 정의 메시지 (Schema Defined Messages)

All messages sent between accounts are defined by a schema which is part of the blockchain consensus state. This schema allows seamless conversion between binary and JSON representation of the messages.

## 스키마 정의 데이터베이스 (Schema Defined Database)

Database state is also defined using a similar schema. This ensures that all data stored by all applications is in a format that can be interpreted as human readable JSON but stored and manipulated with the efficiency of binary.

## 애플리케이션과 인증 분리 (Separating Authentication from Application)

To maximize parallelization opportunities and minimize the computational debt associated with regenerating application state from the transaction log, EOS.IO software separates validation logic into three sections:

  1. 메시지의 내적 일관성(internal consistency) 검증
  2. 모든 전제 조건의 유효성 검증
  3. 애플리케이션 상태의 변경

Validating the internal consistency of a message is read-only and requires no access to blockchain state. This means that it can be performed with maximum parallelism. Validating preconditions, such as required balance, is read-only and therefore can also benefit from parallelism. Only modification of application state requires write access and must be processed sequentially for each application.

Authentication is the read-only process of verifying that a message can be applied. Application is actually doing the work. In real time both calculations are required to be performed, however once a transaction is included in the blockchain it is no longer necessary to perform the authentication operations.

## 가상 머신 독립 아키텍처 (Virtual Machine Independent Architecture)

It is the intention of the EOS.IO software-based blockchain that multiple virtual machines can be supported and new virtual machines added over time as necessary. For this reason, this paper will not discuss the details of any particular language or virtual machine. That said, there are two virtual machines that are currently being evaluated for use with an EOS.IO software-based blockchain.

### 웹어셈블리 (WASM; Web Assembly)

Web Assembly is an emerging web standard for building high performance web applications. With a few small modifications Web Assembly can be made deterministic and sandboxed. The benefit of Web Assembly is the widespread support from industry and that it enables contracts to be developed in familiar languages such as C or C++.

Ethereum developers have already begun modifying Web Assembly to provide suitable sandboxing and determinism in with their [Ethereum flavored Web Assembly (WASM)](https://github.com/ewasm/design). This approach can be easily adapted and integrated with EOS.IO software.

### 이더리움 가상 머신 (EVM; Ethereum Virtual Machine)

This virtual machine has been used for most existing smart contracts and could be adapted to work within an EOS.IO blockchain. It is conceivable that EVM contracts could be run within their own sandbox inside an EOS.IO software-based blockchain and that with some adaptation EVM contracts could communicate with other EOS.IO software blockchain applications.

# 블록체인 간 통신 (Inter Blockchain Communication)

EOS.IO software is designed to facilitate inter-blockchain communication. This is achieved by making it easy to generate proof of message existence and proof of message sequence. These proofs combined with an application architecture designed around message passing enables the details of inter-blockchain communication and proof validation to be hidden from application developers.

<img align="right" src="http://eos.io/wpimg/Diagram1.jpg" width="362.84px" height="500px" />

## 경량화된 클라이언트 검증(LCV)을 위한 머클 증명 (Merkle Proofs for Light Client Validation)

Integrating with other blockchains is much easier if clients do not need to process all transactions. After all, an exchange only cares about transfers in and out of the exchange and nothing more. It would also be ideal if the exchange chain could utilize lightweight merkle proofs of deposit rather than having to trust its own block producers entirely. At the very least a chain's block producers would like to maintain the smallest possible overhead when synchronizing with another blockchain.

The goal of LCV is to enable the generation of relatively light-weight proof of existence that can be validated by anyone tracking a relatively light-weight data set. In this case the objective is to prove that a particular transaction was included in a particular block and that the block is included in the verified history of a particular blockchain.

Bitcoin supports validation of transactions assuming all nodes have access to the full history of block headers which amounts to 4MB of block headers per year. At 10 transactions per second, a valid proof requires about 512 bytes. This works well for a blockchain with a 10 minute block interval, but is no longer "light" for blockchains with a 3 second block interval.

The EOS.IO software enables lightweight proofs for anyone who has any irreversible block header after the point in which the transaction was included. Using the hash-linked structure shown below it is possible to prove the existence of any transaction with a proof less than 1024 bytes in size. If it is assumed that validating nodes are keeping up with all block headers in the past day (2 MB of data), then proving these transactions will only require proofs 200 bytes long.

There is little incremental overhead associated with producing blocks with the proper hash-linking to enable these proofs which means there is no reason not to generate blocks this way.

When it comes time to validate proofs on other chains there are a wide variety of time/ space/ bandwidth optimizations that can be made. Tracking all block headers (420 MB/year) will keep proof sizes small. Tracking only recent headers can offer a trade off between minimal long-term storage and proof size. Alternatively, a blockchain can use a lazy evaluation approach where it remembers intermediate hashes of past proofs. New proofs only have to include links to the known sparse tree. The exact approach used will necessarily depend upon the percentage of foreign blocks that include transactions referenced by merkle proof.

After a certain density of interconnectedness it becomes more efficient to simply have one chain contain the entire block history of another chain and eliminate the need for proofs all together. For performance reasons, it is ideal to minimize the frequency of inter-chain proofs.

## 체인 간 통신의 지연 시간 (Latency of Interchain Communication)

When communicating with another outside blockchain, block producers must wait until there is 100% certainty that a transaction has been irreversibly confirmed by the other blockchain before accepting it as a valid input. Using an EOS.IO software-based blockchain and DPOS with 3 second blocks and 21 producers, this takes approximately 45 seconds. If a chain's block producers do not wait for irreversibility it would be like an exchange accepting a deposit that was later reversed and could impact the validity of the blockchain's consensus.

## 완전성 증명 (Proof of Completeness)

When using merkle proofs from outside blockchains, there is a significant difference between knowing that all transactions processed are valid and knowing that no transactions have been skipped or omitted. While it is impossible to prove that all of the most recent transactions are known, it is possible to prove that there have been no gaps in the transaction history. The EOS.IO software facilitates this by assigning a sequence number to every message delivered to every account. A user can use these sequence numbers to prove that all messages intended for a particular account have been processed and that they were processed in order.

# 결론 (Conclusion)

The EOS.IO software is designed from experience with proven concepts and best practices, and represents fundamental advancements in blockchain technology. The software is part of a holistic blueprint for a globally scalable blockchain society in which decentralised applications can be easily deployed and governed.