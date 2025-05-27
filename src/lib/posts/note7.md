---
title: "Distributed System notes 7"
date: "2025-05-22"
updated: "2025-05-22"
categories:
  - "sveltekit"
  - "markdown"
coverImage: "https://www.researchgate.net/profile/Jeff-Squyres/publication/228687501/figure/fig2/AS:669234074918912@1536562747240/Open-MPIs-Layered-Architecture.png"
coverWidth: 16
coverHeight: 9
excerpt: So sánh các giao thức mạng và tìm hiểu OpenMPI.
---

# Tổng hợp kiến thức về giao thức mạng & OpenMPI

## 1. So sánh các giao thức HTTP, TCP/IP, UDP, REST, GraphQL, SOAP, AJAX, RPC, gRPC

### a. Mục đích sử dụng và liên quan

- **TCP/IP**: Bộ giao thức nền tảng của Internet. TCP (Transmission Control Protocol) đảm bảo truyền dữ liệu tin cậy, có kiểm tra lỗi, còn IP (Internet Protocol) lo việc định tuyến gói tin. Hầu hết các giao thức khác đều "chạy trên" TCP/IP.
- **UDP**: Giao thức truyền dữ liệu không đảm bảo, không kiểm tra lỗi, gửi là xong, rất nhanh (dùng cho video call, game online, livestream...).
- **HTTP**: Giao thức truyền tải siêu văn bản, dùng để trình duyệt và server nói chuyện với nhau (web). HTTP chạy trên TCP/IP.
- **REST**: Kiểu thiết kế API dựa trên HTTP, mỗi tài nguyên là một URL, dùng các phương thức GET, POST, PUT, DELETE... REST không phải là giao thức, mà là phong cách thiết kế API.
- **GraphQL**: Một kiểu API mới, cho phép client hỏi đúng dữ liệu mình cần, không thừa không thiếu. Cũng chạy trên HTTP.
- **SOAP**: Giao thức nhắn tin kiểu cũ, dùng XML, thường dùng trong hệ thống lớn, ngân hàng, bảo hiểm. SOAP cũng chạy trên HTTP hoặc các giao thức khác.
- **AJAX**: Kỹ thuật (không phải giao thức) cho phép web gửi/nhận dữ liệu với server mà không cần tải lại trang. AJAX thường dùng HTTP (GET/POST).
- **RPC**: Remote Procedure Call, gọi hàm từ xa, có thể dùng nhiều giao thức truyền tải (TCP, HTTP...).
- **gRPC**: Phiên bản hiện đại của RPC, dùng Protocol Buffers (protobuf), rất nhanh, thường dùng cho microservices, chạy trên HTTP/2.

### b. Tương quan và khác biệt

- **TCP/IP** là nền tảng, HTTP/UDP chạy trên TCP/IP.
- **HTTP** là giao thức ứng dụng, REST/GraphQL/SOAP/AJAX đều dựa trên HTTP.
- **REST, GraphQL, SOAP** là cách thiết kế API, còn HTTP là giao thức truyền tải.
- **AJAX** là kỹ thuật dùng HTTP để lấy dữ liệu động.
- **RPC/gRPC** là kiểu gọi hàm từ xa, gRPC hiện đại, nhanh, dùng protobuf, còn RPC truyền thống có thể dùng nhiều kiểu truyền tải.
- **UDP** khác TCP ở chỗ không đảm bảo dữ liệu, dùng cho ứng dụng cần tốc độ, không cần chính xác tuyệt đối.

**Tóm lại:**
- TCP/IP là nền tảng, HTTP chạy trên TCP, các API (REST, GraphQL, SOAP) chạy trên HTTP. AJAX là kỹ thuật dùng HTTP. RPC/gRPC là kiểu gọi hàm từ xa, gRPC hiện đại hơn, nhanh hơn.

---

## 2. Nghiên cứu về thư viện OpenMPI và giải pháp tính số nguyên tố

### a. OpenMPI là gì?

- **OpenMPI** là một thư viện mã nguồn mở giúp lập trình song song phân tán (distributed & parallel computing). Nó cho phép nhiều máy tính (hoặc nhiều core trên 1 máy) cùng làm việc để giải quyết một bài toán lớn.
- OpenMPI dùng chuẩn **MPI (Message Passing Interface)**, tức là các tiến trình gửi/nhận dữ liệu cho nhau qua các hàm gửi/nhận (send/recv).

### b. Một số hàm cơ bản trong OpenMPI

- `MPI_Init`: Khởi tạo môi trường MPI.
- `MPI_Comm_size`: Lấy tổng số tiến trình (process) đang chạy.
- `MPI_Comm_rank`: Lấy số thứ tự (ID) của tiến trình hiện tại.
- `MPI_Send`: Gửi dữ liệu sang tiến trình khác.
- `MPI_Recv`: Nhận dữ liệu từ tiến trình khác.
- `MPI_Finalize`: Kết thúc chương trình MPI.
- Ngoài ra còn nhiều hàm khác như broadcast, scatter, gather...

### c. Giải pháp tính 10 triệu số nguyên tố với 32 core

**Bài toán:** Tính 10,000,000 số nguyên tố đầu tiên, dùng 16 máy, mỗi máy 2 core (tổng 32 core).

**Ý tưởng giải:**
- Chia đều công việc cho 32 core. Mỗi core sẽ tính một đoạn số, ví dụ core 0 tính từ 2 đến N1, core 1 từ N1+1 đến N2, v.v.
- Dùng OpenMPI để mỗi core tự tính phần của mình, sau đó gửi kết quả về 1 core tổng hợp (gọi là master).
- Để linh hoạt số core (8, 10, 12, 32...), chỉ cần chia đoạn số cần kiểm tra cho số core hiện có, mỗi core tự biết mình phải làm gì dựa vào ID (rank).
- Sau khi các core tính xong, master sẽ ghép kết quả lại, sắp xếp đúng thứ tự, lấy ra 10 triệu số nguyên tố đầu tiên.

**Cách làm chi tiết:**
1. Dùng `MPI_Comm_size` để biết tổng số core (P), `MPI_Comm_rank` để biết core hiện tại là số mấy (r).
2. Chia đoạn số cần kiểm tra thành P phần, mỗi core tính phần của mình.
3. Mỗi core tự tìm các số nguyên tố trong đoạn của mình (dùng thuật toán sàng Eratosthenes hoặc kiểm tra chia hết).
4. Gửi kết quả về core 0 (master) bằng `MPI_Send`/`MPI_Recv` hoặc `MPI_Gather`.
5. Core 0 tổng hợp, sắp xếp, lấy ra 10 triệu số nguyên tố đầu tiên.

**Lưu ý:**
- Nếu số core thay đổi (8, 10, 12...), chỉ cần chia lại đoạn số, code không cần sửa nhiều.
- Nên chọn thuật toán tìm nguyên tố tối ưu (sàng, chia hết tới căn bậc hai).
- Có thể tối ưu thêm bằng cách mỗi core chỉ kiểm tra số lẻ, hoặc dùng các kỹ thuật chia nhỏ hơn nữa.

**Tóm lại:**
- OpenMPI giúp chia nhỏ bài toán cho nhiều máy/core cùng làm, tăng tốc rất nhiều so với chạy 1 máy.
- Chỉ cần chia đều công việc, dùng các hàm MPI để gửi/nhận dữ liệu, là có thể giải quyết bài toán lớn một cách hiệu quả.

---

**Nguồn tham khảo:**
- Tài liệu chính thức OpenMPI: https://www.open-mpi.org/doc/
- Kinh nghiệm thực tế lập trình song song, các bài toán tính nguyên tố trên nhiều core.



