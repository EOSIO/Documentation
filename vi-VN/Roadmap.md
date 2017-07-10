## Lộ trình phần mềm EOS. IO

Tài liệu này vạch ra kế hoạch phát triển mức độ cao và sẽ được cập nhật như là tiến trình được thực hiện đối với phiên bản 1.0. Cần lưu ý rằng lộ trình này áp dụng với blockchain và không dành cho các công cụ và tiện ích như ví và xem xét tìm kiếm các chuỗi khối, chúng sẽ có đội phát triển riêng khi giai đoạn 1 đã hoàn tất.

***Tất cả mọi thứ trong tài liệu này ở dạng dự thảo và có thể thay đổi bất cứ lúc nào và chỉ cung cấp cho mục đích thông tin về Eos. block.One không đảm bảo tính chính xác của thông tin trong lộ trình này và các thông tin được cung cấp không có đại diện chiệu trách nhiệm hoặc bảo đảm, rõ ràng.***

# Giai đoạn 1 - môi trường thử nghiệm tối thiểu khả thi - Trong mùa hè năm 2017

Mục tiêu của giai đoạn này là để thiết lập các API dành cho nhà phát triển, họ sẽ yêu cầu để xây dựng và thử nghiệm các ứng dụng trên nền tảng EOS. Để cho các nhà phát triển để bắt đầu thử nghiệm các ứng dụng của họ, sẽ có các yêu cầu sau đây được thực hiện:

### Nút Độc lập (Dan &Nathan)

Một nút độc lập hoạt động trên một chuỗi khối(blockchain) thử nghiệm và tạo ra các khối trong khi phơi bày một API. Nút này không cần phải quan tâm, dính dáng đến bất kỳ mã lệnh mạng P2P.

### Hợp đồng bản địa (Nathan)

Nền tảng Phần mềm EOS. IO có một số lượng hợp đồng gốc. Đây là hợp đồng(contract) quản lý các hoạt động cốt lõi của chuối khối(blockchain) và tồn tại bên ngoài giao diện của WebAssembly. Các hợp đồng này bao gồm:

1. @eos - quản lý việc chuyển các tốt Kinh(token) EOS
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

### Resource Usage Tracking & Rate Limiting (Arhag)

To prevent abuse the resource monitoring and usage tracking rate limits users accoding to staked EOS.

### Genesis Import Testing (DappHub)

Tools need to be developed to export data from the EOS Token Distribution state and create a genesis configuration file. This will enable anyone participating in the Token Distribution to acquire some initial test EOS (TEOS).

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