## Lộ trình phần mềm EOS. IO

Tài liệu này vạch ra kế hoạch phát triển mức độ cao và sẽ được cập nhật như là tiến trình được thực hiện đối với phiên bản 1.0. Cần lưu ý rằng lộ trình này áp dụng với blockchain và không dành cho các công cụ và tiện ích như ví và xem xét tìm kiếm các chuỗi khối, chúng sẽ có đội phát triển riêng khi giai đoạn 1 đã hoàn tất.

***Tất cả mọi thứ trong tài liệu này ở dạng dự thảo và có thể thay đổi bất cứ lúc nào và chỉ cung cấp cho mục đích thông tin về Eos. block.One không đảm bảo tính chính xác của thông tin trong lộ trình này và các thông tin được cung cấp không có đại diện chiệu trách nhiệm hoặc bảo đảm, rõ ràng.***

# Giai đoạn 1 - môi trường thử nghiệm tối thiểu khả thi - Trong mùa hè năm 2017

Mục tiêu của giai đoạn này là để thiết lập các API dành cho nhà phát triển, họ sẽ yêu cầu để xây dựng và thử nghiệm các ứng dụng trên nền tảng EOS. Để cho các nhà phát triển để bắt đầu thử nghiệm các ứng dụng của họ, sẽ có các yêu cầu sau đây được thực hiện:

### Nút chạy riêng lẻ (Dan &Nathan)

Một nút độc lập hoạt động trên một chuỗi khối(blockchain) thử nghiệm và tạo ra các khối trong khi phơi bày một API. Nút này không cần phải quan tâm, dính dáng đến bất kỳ mã lệnh mạng P2P.

### Hợp đồng bản địa (Nathan)

Nền tảng Phần mềm EOS. IO có một số lượng hợp đồng gốc. Đây là hợp đồng(contract) quản lý các hoạt động cốt lõi của chuối khối(blockchain) và tồn tại bên ngoài giao diện của WebAssembly. Các hợp đồng này bao gồm:

1. @eos - quản lý việc chuyển các tốt Kinh(token) EOS
2. @stake - quản lý khóa EOS, ban hành các cuộc bỏ phiếu, và cuộc bầu cử
3. @system - quản lý phân quyền, cập nhật mã lệnh chovtin nhắn và liên lạc

### Máy ảo API (Dan)

Hợp đồng được biên dịch để WebAssembly (WASM) và WASM phải giao tiếp với chuỗi khối ( blockchain) thông qua một API được xác định. API này là những gì nhà phát triển dựa vào để xây dựng các ứng dụng và tương đối ổn định trước khi các nhà phát triển thực sự có thể bắt đầu xây dựng trên nền tảng EOS.

### Giao tiếp RPC (Arhag, Nathan)

Một giao tiếp JSON RPC đơn giản qua HTTP sẽ cung cấp cho phép nhà phát triển phát đi các giao dịch và truy vấn trạng thái của ứng dụng. Điều này rất quan trọng cho cả việc phát hành và tương tác với các ứng dụng thử nghiệm.

### Công cụ dòng lệnh (Arhag)

Công cụ dòng lệnh tạo thuận lợi cho việc tích hợp giao tiếp RPC với nhà phát triển để xây dựng môi trường ứng dụng EOS.

### Tài liệu phát triển phần mềm cơ bản (Josh)

Tài liệu dạy các nhà phát triển làm thế nào để bắt đầu việc xây dựng phần mềm trên nền tảng chuỗi khối EOS.IO. Điều này bao gồm các tài liệu WASM API, giao tiếp RPC, và các công cụ dòng lệnh.

# Giai đoạn 2 - Mạng lưới thử nghiệm tối thiểu khả thi (Test network) - mùa thu năm 2017

Tất cả mọi thứ trong giai đoạn 1 giả định một môi trường tin cậy chỉ chạy riêng các mã lệnh của nhà phát triển. Trước khi có một mạng thử nghiệm (test network) có thể được triển khai một số tính năng bổ sung cần được thực hiện và kiểm tra.

### Mã lệnh cho mạng P2P(Phil)

Đây là một phần nhúng(plugin) có trách nhiệm đồng bộ hóa hiện trạng của chuổi khối( blockchain) giữa hai nút chạy riêng lẻ( nút độc lập).

### WASM Sanitation & CPU Sandboxing (Brian)

Mã WASM cần phải được kiểm tra cắt tỉa vệ sinh để kiểm tra các hành vi không xác định như điểm hoạt động tràn(floating point operations) và vòng lặp vô hạn.

### Theo dõi Tài nguyên sử dụng & tỷ lệ giới hạn (Arhag)

Để ngăn chặn lạm dụng, việc theo dõi tài nguyên và theo dõi tỷ lệ giới hạn sử dụng của người dùng dựa trên số EOS được đặt cọc.

### Thử nghiệm nhập chuỗi khối gốc-Genesis (DappHub)

Công cụ cần phải được phát triển để xuất dữ liệu từ các phần EOS được phân phối và tạo ra một tập tin cấu hình genesis. Điều này sẽ cho phép bất cứ ai tham gia vào việc phân phối Token để làm một số thử nghiệm ban đầu trên nền tảng EOS (TEOS).

### Giao tiếp xuyên các chuỗi khối -Interblockchain (Nathan)

Tính năng này liên quan đến việc xác minh giao dịch băm kiểu Merkle là thích hợp.

# Giai đoạn 3 - Chạy thử nghiệm & Đánh giá bảo mật - Mùa đông năm 2017, mùa xuân năm 2018

Trong giai đoạn này, nền tảng Eos sẽ trải qua thử nghiệm hạng nặng trung vào việc tìm kiếm các vấn đề sự cố an ninh và lỗi. Vào cuối giai đoạn 3 Phiên bản 1.0 sẽ được đánh dấu.

### Các ứng dụng ví dụ

Ứng dụng ví dụ là rất quan trọng để chứng minh nền tảng cung cấp các tính năng theo yêu cầu của nhà phát triển thực sự.

### Bounties cho tấn công thành công mạng

Tấn công mạng với thư rác, khai thác máy ảo và lỗi treo, và không xác định hành vi sẽ là một quá trình liên quan đến rất nhiều nhưng cần thiết để đảm bảo rằng phiên bản 1.0 là ổn định.

### Ngôn ngữ lập trình được hổ trợ

Hỗ trợ thêm một số ngôn ngữ lập trình bổ sung được biên dịch qua ngôn ngữ WASM: C+ +, Rust, vv.

### Tài liệu   
& hướng dẫn

# Giai đoạn 4 - Tối ưu hóa song song - Mùa hè / mùa thu năm 2018

Sau khi nhận được một phiên bản ổn định 1.0 phát hành, chúng tôi sẽ tiếp tục tối ưu hóa mã lệnh cho việc chạy phần mềm song song.

# Giai đoạn 5 - Áp dụng hoàn thành Cluster trong tương lai