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
excerpt: Giai thich chi tiet cac giao thuc mang (TCP/IP, UDP, HTTP, REST, GraphQL, SOAP, AJAX, RPC, gRPC) va huong dan su dung OpenMPI de tinh toan song song, trinh bay de hieu, van phong sinh vien.
---

# Tổng hợp kiến thức về giao thức mạng & OpenMPI (giải thích kỹ)

## 1. Tìm hiểu kỹ về các giao thức mạng

### a. TCP/IP, UDP
- **TCP/IP** là nền tảng của Internet, gồm nhiều tầng. TCP (Transmission Control Protocol) đảm bảo truyền dữ liệu tin cậy, đúng thứ tự, không mất mát. IP (Internet Protocol) lo việc định tuyến gói tin từ máy này sang máy khác.
- **UDP** (User Datagram Protocol) thì "gửi là xong", không kiểm tra, không đảm bảo đến nơi, nhưng rất nhanh. Dùng cho game online, video call, livestream...

**Ví dụ:**
- Tải file trên Google Drive dùng TCP (chậm nhưng chắc, không bị lỗi file).
- Gọi video Messenger dùng UDP (hình ảnh có thể vỡ, nhưng không bị trễ).

### b. HTTP, REST, GraphQL, SOAP, AJAX
- **HTTP**: Giao thức truyền tải dữ liệu web. Khi bạn vào bất kỳ trang web nào, trình duyệt đều dùng HTTP để lấy dữ liệu từ server về. HTTP chạy trên TCP.
- **REST**: Không phải là giao thức, mà là cách thiết kế API dựa trên HTTP. Mỗi tài nguyên là một URL, dùng các phương thức GET, POST, PUT, DELETE. Dữ liệu trả về thường là JSON.
- **GraphQL**: API kiểu mới, cho phép client hỏi đúng dữ liệu mình cần, không thừa không thiếu. Một endpoint duy nhất, truy vấn linh hoạt. Dùng HTTP làm nền tảng.
- **SOAP**: Giao thức kiểu cũ, dùng XML để truyền dữ liệu. Rất chặt chẽ, nhiều quy tắc, thường dùng trong hệ thống lớn, ngân hàng, bảo hiểm. Chạy trên HTTP hoặc các giao thức khác.
- **AJAX**: Không phải giao thức, mà là kỹ thuật cho phép web gửi/nhận dữ liệu với server mà không cần tải lại trang. AJAX thường dùng HTTP (GET/POST), trả về JSON hoặc XML.

**Ví dụ:**
- Lướt Facebook, kéo xuống là có thêm bài mới mà không cần tải lại trang, đó là AJAX.
- Đăng nhập vào một web hiện đại, thường là REST API phía sau.
- Làm việc với hệ thống ngân hàng, có thể gặp SOAP.

### c. RPC, gRPC
- **RPC**: Gọi hàm từ xa, giống như gọi hàm trong code, nhưng thực ra là gọi qua mạng. Có thể dùng nhiều giao thức truyền tải (TCP, HTTP...).
- **gRPC**: Phiên bản hiện đại của RPC, do Google phát triển. Dùng Protocol Buffers (protobuf) để truyền dữ liệu, rất nhanh, nhẹ. Chạy trên HTTP/2, hỗ trợ streaming, bảo mật tốt. Thường dùng cho microservices, hệ thống lớn cần tốc độ cao.

**Ví dụ:**
- Hệ thống microservices của Google, Netflix, Grab... thường dùng gRPC để các service nói chuyện với nhau.

### d. Tóm tắt mối liên hệ
- **TCP/IP** là nền tảng, HTTP/UDP chạy trên TCP/IP.
- **HTTP** là giao thức ứng dụng, REST/GraphQL/SOAP/AJAX đều dựa trên HTTP.
- **REST, GraphQL, SOAP** là cách thiết kế API, còn HTTP là giao thức truyền tải.
- **AJAX** là kỹ thuật dùng HTTP để lấy dữ liệu động.
- **RPC/gRPC** là kiểu gọi hàm từ xa, gRPC hiện đại hơn, nhanh hơn.
- **UDP** khác TCP ở chỗ không đảm bảo dữ liệu, dùng cho ứng dụng cần tốc độ, không cần chính xác tuyệt đối.

---

## 2. OpenMPI và bài toán tính số nguyên tố (giải thích kỹ)

### a. OpenMPI là gì?
- Là thư viện giúp nhiều máy tính (hoặc nhiều core trên 1 máy) cùng làm việc, chia sẻ dữ liệu qua mạng.
- Dùng chuẩn MPI (Message Passing Interface), các tiến trình gửi/nhận dữ liệu qua các hàm như `MPI_Send`, `MPI_Recv`.
- Rất mạnh khi cần xử lý dữ liệu lớn, tính toán nặng (ví dụ: mô phỏng vật lý, AI, xử lý ảnh, tính toán khoa học).

### b. Một số hàm OpenMPI thường dùng
- `MPI_Init`: Khởi tạo môi trường MPI, phải gọi đầu tiên.
- `MPI_Comm_size`: Lấy tổng số tiến trình (process) đang chạy (tức là tổng số core bạn dùng).
- `MPI_Comm_rank`: Lấy số thứ tự (ID) của tiến trình hiện tại (từ 0 đến N-1).
- `MPI_Send`, `MPI_Recv`: Gửi/nhận dữ liệu giữa các tiến trình.
- `MPI_Bcast`: Gửi dữ liệu từ 1 tiến trình đến tất cả tiến trình khác.
- `MPI_Gather`, `MPI_Scatter`: Gom dữ liệu từ nhiều tiến trình về 1, hoặc chia dữ liệu từ 1 ra nhiều tiến trình.
- `MPI_Finalize`: Kết thúc chương trình MPI.

### c. Giải pháp chi tiết cho bài toán 10 triệu số nguyên tố

**Bước 1:**
- Dùng `MPI_Comm_size` để biết tổng số core (P), `MPI_Comm_rank` để biết core hiện tại là số mấy (r).

**Bước 2:**
- Chia đoạn số cần kiểm tra thành P phần, mỗi core tính phần của mình.
  - Ví dụ: Tìm 10 triệu số nguyên tố đầu tiên, ước lượng số cuối cùng là N (có thể dùng công thức xấp xỉ hoặc thử dần).
  - Mỗi core kiểm tra một đoạn:
    - Core 0: từ 2 đến N1
    - Core 1: từ N1+1 đến N2
    - ...
    - Core P-1: từ N(P-1)+1 đến N

**Bước 3:**
- Mỗi core tự tìm các số nguyên tố trong đoạn của mình (dùng sàng Eratosthenes hoặc kiểm tra chia hết).

**Bước 4:**
- Gửi kết quả về core 0 (master) bằng `MPI_Send`/`MPI_Recv` hoặc `MPI_Gather`.

**Bước 5:**
- Core 0 tổng hợp, sắp xếp, lấy ra 10 triệu số nguyên tố đầu tiên.

**Lưu ý thực tế:**
- Nếu số core thay đổi (8, 10, 12...), chỉ cần chia lại đoạn số, code không cần sửa nhiều.
- Nên chọn thuật toán tìm nguyên tố tối ưu (sàng, chia hết tới căn bậc hai).
- Có thể tối ưu thêm bằng cách mỗi core chỉ kiểm tra số lẻ, hoặc dùng các kỹ thuật chia nhỏ hơn nữa.
- Nếu đoạn số quá lớn, có thể chia nhỏ hơn nữa, mỗi core xử lý nhiều lần.

**Ví dụ code chia đoạn (giả lập):**
```python
P = 32  # số core
r = ... # rank của core hiện tại
N = ... # số cuối cùng cần kiểm tra
start = r * (N // P) + 2
end = (r+1) * (N // P) + 1 if r != P-1 else N
# mỗi core kiểm tra từ start đến end
```

---

**Nguồn tham khảo:**  
- Tài liệu chính thức OpenMPI: https://www.open-mpi.org/doc/   
- Các bài blog, video hướng dẫn về MPI, tính toán phân tán.



