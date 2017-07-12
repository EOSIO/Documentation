## EOS.IO Software Roadmap

이 문서는 EOS. IO 개발 계획에 대한 대략적인 윤곽만을 제공합니다. 이후 버전이 1.0으로 변함에 따라 이 문서의 내용도 업데이트 될 것입니다. 이 문서의 내용은 ‘블록체인 소프트웨어’에만 적용되며, 1단계가 완료될 시 전용 팀과 로드맵이 구비되는 전자지갑이나 블록 익스플로어와 같은 다른 툴이나 유틸리티에는 적용되지 않습니다.

***Everything contained in this document is in draft form and subject to change at any time and provided for information purposes only. Block.one(1)은 로드맵에 포함된 정보의 정확성을 보증할 수 없습니다. 이 문서에 포함된 정보는 함축적이거나 묘사하거나 보증하거나 하는 목적이 전혀 없이 “있는 그대로”를 전달하는 것입니다.***

# 1단계 - 실행 가능한 최소의 테스트 환경 - 2017년 여름

이 단계의 목표는 개발자가 EOS.IO에서 응용프로그램을 빌드하고 테스트하는 데 필요한 API를 설정하는 것입니다. 개발자가 어플리케이션을 테스트하기 위해서는 다음의 것들을 구현해야 합니다.

### 독립실행형 노드 (Dan & Nathan)

독립실행형 노드는 테스트 블록체인을 가동하고 API를 노출하면서 블록을 생성합니다. 이 노드 실행에는 P2P 네트워킹 코드가 필요하지 않습니다.

### 기본 컨트랙 (Nathan)

EOS.IO 소프트웨어에는 여러 가지 기본적 협약들이 있습니다. 이 협약들은 블록체인의 핵심 운영을 관리하는, 웹 어셈블리(Webassembly) 인터페이스 외부에 존재하는 협약입니다. 이 협약에는 다음의 것들이 포함됩니다.

1. @eos - EOS 토큰의 전송을 관리합니다.
2. @stake - 묶인 EOS와 보팅, 생산자 선출을 관리합니다.
3. @system - 사용 권한, 메시지, 연락처 코드 업데이트를 관리합니다.

### 가상 머신 API (Dan)

협약은 웹어셈블리(Web Assembly, 이하 WASM)로 컴파일되고 WASM은 정의 된 API를 통해 블록 체인과 통신해야 합니다. This API is what developers depend upon to build applications and be relatively stable before developers can really start to build on EOS.

### RPC Interface (Arhag, Nathan)

A simple JSON RPC over HTTP interface will be provided that enables developers to broadcast transactions and query application state. This is critical for both publishing and interacting with test applications.

### Command line Tools (Arhag)

Command line tools facilitate integrating the RPC interface with developer build environments.

### Basic Developer Documentation (Josh)

Documents that teach developers how to get started with building on EOS.IO blockchains. This includes documentations of the WASM API, RPC Interface, and Command Line Tools.

# 2단계: 기초적인 테스트 네트워크 - 2017년 가을

Everything in Phase 1 assumes a trusted environment that only runs the developer's own code. Before a test network can be deployed several additional features need to be implemented and tested.

### P2P Network Code (Phil)

This is a plugin that is responsible for synchronizing the blockchain state between two standalone nodes.

### WASM Sanitation & CPU Sandboxing (Brian)

The WASM code needs to be sanitized to check for non-deterministic behavior such as floating point operations and infinite loops.

### Resource Usage Tracking & Rate Limiting (Arhag)

To prevent abuse the resource monitoring and usage tracking rate limits users accoding to staked EOS.

### Genesis Import Testing (DappHub)

Tools need to be developed to export data from the EOS Token Distribution state and create a genesis configuration file. This will enable anyone participating in the Token Distribution to acquire some initial test EOS (TEOS).

### Interblockchain Communication (Nathan)

This feature involves verifying the Merkle hashing of transactions is proper.

# 3단계: 시험운영 및 보안성 검토 - 2017년 겨울 ~ 2018년 봄

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

# 5단계 - 클러스터 구현 - 향후