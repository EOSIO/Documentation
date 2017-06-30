## Дорожная карта программного обеспечения EOS.IO

В этом документе обозначен общий план разработки, который обновляться по мере прогресса в работе над версией 1.0. Нужно отметить, что эта дорожная карта применима только к программному обеспечению блокчейна и не относится к другим инструментам и утилитам, например, кошелькам и обозревателям блоков, для разработки которых после завершения Этапа 1 будут созданы отдельные команды и дорожные карты.

***Всё содержимое этого документа находится на стадии черновика и может измениться в любое время. Предоставленная информация служит сугубо информационным целям.  
block.one не гарантирует точности информации в этой дорожной карте, которая предоставляется "как есть" без явных или подразумеваемых гарантий и заявлений.***

# Этап 1 - Минимальная необходимая тестовая среда - Лето 2017

Цель этого этапа - установить API, которые потребуются разработчикам, чтобы начать строить и тестировать приложения на EOS.IO. In order for developers to start testing their applications they will require the following to be implemented:

### Standalone Node (Dan & Nathan)

A standalone node operates a test blockchain and produces blocks while exposing an API. This node does not need to concern itself with any P2P networking code.

### Native Contracts (Nathan)

The EOS.IO software has a number of native contracts. These are contracts that manage the core operations of the blockchain and exist outside the Web Assembly interface. These contracts include:

  1. @eos - manages EOS token transfers
  2. @stake - manages locked EOS, voting, and Producer Election
  3. @system - manages permissions, messages, and contact code updates

### Virtual Machine API (Dan)

Contracts are compiled to WebAssembly (WASM) and WASM must interface with the blockchain via a defined API. This API is what developers depend upon to build applications and be relatively stable before developers can really start to build on EOS.

### RPC Interface (Arhag, Nathan)

A simple JSON RPC over HTTP interface will be provided that enables developers to broadcast transactions and query application state. This is critical for both publishing and interacting with test applications.

### Command line Tools (Arhag)

Command line tools facilitate integrating the RPC interface with developer build environments.

### Basic Developer Documentation (Josh)

Documents that teach developers how to get started with building on EOS.IO blockchains. This includes documentations of the WASM API, RPC Interface, and Command Line Tools.

# Phase 2 - Minimal Viable Test Network - Fall 2017

Everything in Phase 1 assumes a trusted environment that only runs the developer's own code. Before a test network can be deployed several additional features need to be implemented and tested.

### P2P Network Code (Phil)

This is a plugin that is responsible for synchronizing the blockchain state between two standalone nodes.

### WASM Sanitation & CPU Sandboxing (Brian)

The WASM code needs to be sanitized to check for non-deterministic behavior such as floating point operations and infinite loops.

### Resource Usage Tracking & Rate Limiting ( Arhag )

To prevent abuse the resource monitoring and usage tracking rate limits users accoding to staked EOS.

### Genesis Import Testing (DappHub)

Tools need to be developed to export data from the EOS Token Distribution state and create a genesis configuration file. This will enable anyone participating in the Token Distribution to acquire some initial test EOS (TEOS)

### Interblockchain Communication (Nathan)

This feature involves verifying the Merkle hashing of transactions is proper.

# Phase 3 - Testing & Security Audits - Winter 2017, Spring 2018

During this phase the platform will undergo heavy testing with a focus on finding security issues and bug. At the end of Phase 3 version 1.0 will be tagged.

### Develop Example Applications

Example applications are critical to proving the platform provides the features required by real developers.

### Bounties for Succesfully Attacking Network

Attacking the network with spam, virtual machine exploits, and bug crashes, and non-deterministic behavior will be a heavily involved process but necessary to ensure that version 1.0 is stable.

### Language Support

Adding support for additional langauges to be compiled to WASM: C++, Rust, etc.

### Documentation & Tutorials

# Phase 4 - Parallel Optimization Summer / Fall 2018

After getting a stable 1.0 product released, we will move toward optimizing the code for parallel execution.

# Phase 5 - Cluster Implementation The Future