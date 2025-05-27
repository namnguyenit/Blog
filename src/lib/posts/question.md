---
title: "Distributed System test main questions (Bản mở rộng chi tiết)"
date: "2025-05-28"
updated: "2025-05-28"
categories:
  - "sveltekit"
  - "markdown"
coverImage: "https://tino.org/wp-content/uploads/2021/08/word-image-210.jpeg"
coverWidth: 16
coverHeight: 9
excerpt: Tổng hợp kiến thức hệ phân tán, so sánh giao thức, giải thích sâu, ví dụ thực tế.
---

# Báo cáo tổng hợp ôn tập Hệ Phân Tán (Bản chi tiết)

## 1. So sánh hệ thống tập trung, phân tán, phi tập trung
- **Tập trung:** Mọi thứ xử lý ở một nơi, ví dụ như server trung tâm. Hình dung như Facebook thời đầu, mọi dữ liệu đều nằm ở server của Facebook, hoặc các ngân hàng truyền thống, mọi giao dịch đều phải qua một máy chủ duy nhất. Ưu điểm là dễ quản lý, bảo mật tập trung, nhưng nhược điểm là dễ bị tấn công, nếu server "ngủm" thì cả hệ thống cũng "ngủm" theo. Ngoài ra, khi muốn mở rộng thì khá vất vả, vì phải nâng cấp máy chủ chính.
- **Phân tán:** Nhiều máy cùng xử lý, chia sẻ tài nguyên, không phụ thuộc vào một điểm. Ví dụ như Google Search, các server ở nhiều nơi cùng trả lời truy vấn, hoặc hệ thống đặt vé máy bay, mỗi khu vực có server riêng, khi một server bị lỗi thì các server khác vẫn hoạt động. Tuy nhiên, vẫn có thể có một server chính để điều phối chung. Hệ phân tán giúp tăng khả năng chịu lỗi, mở rộng dễ dàng, nhưng phức tạp hơn về mặt quản lý, phải đồng bộ dữ liệu giữa các server.
- **Phi tập trung:** Không có trung tâm, mọi nút đều ngang hàng. Ví dụ như Blockchain, BitTorrent. Blockchain là ví dụ điển hình, không ai làm chủ, mọi người đều bình đẳng. Ưu điểm là không có điểm chết, khó bị tấn công toàn bộ, nhưng nhược điểm là tốc độ xử lý có thể chậm, khó đồng thuận, và việc cập nhật dữ liệu đồng nhất giữa các node là một thách thức lớn.
- **Cách nhận biết:** Tập trung thì có 1 điểm chết, phân tán thì nhiều điểm nhưng vẫn có thể có nút chủ, phi tập trung thì không có trung tâm nào cả. Trong thực tế, nhiều hệ thống "lai" giữa phân tán và phi tập trung để tận dụng ưu điểm của cả hai.

## 2. Đặc tính của hệ phân tán
- **Tính minh bạch:** Người dùng không biết tài nguyên ở đâu, chỉ cần dùng thôi. Ví dụ như khi dùng Google Drive, bạn không biết file của mình đang nằm ở server nào, chỉ cần truy cập là có dữ liệu. Tính minh bạch còn thể hiện ở việc di chuyển dữ liệu giữa các server mà người dùng không nhận ra.
- **Tính mở:** Dễ mở rộng, thêm bớt máy không ảnh hưởng hệ thống. Khi Facebook mở rộng sang các nước khác, chỉ cần thêm server là xong, không phải xây lại hệ thống. Tính mở còn giúp hệ thống dễ tích hợp với các dịch vụ bên ngoài.
- **Tính mở rộng:** Hệ thống có thể tăng quy mô dễ dàng, ví dụ như thêm server khi số lượng người dùng tăng đột biến.
- **Tính chịu lỗi:** Một số thành phần hỏng, hệ thống vẫn chạy được. Nếu một server Google bị cháy, các server khác vẫn phục vụ bình thường, người dùng không nhận ra. Để làm được điều này, hệ thống phải có cơ chế backup, phát hiện lỗi và chuyển hướng truy cập.
- **Tính đồng bộ:** Các thành phần phối hợp nhịp nhàng. Các server phải cập nhật dữ liệu cho nhau liên tục, tránh trường hợp dữ liệu bị lệch. Ví dụ, khi bạn đổi mật khẩu Facebook ở Việt Nam, server ở Mỹ cũng phải biết ngay.
- **Tính bảo mật:** Dữ liệu an toàn khi truyền và lưu trữ. Dữ liệu truyền qua mạng phải được mã hóa, tránh bị nghe lén. Ngoài ra, phải có cơ chế xác thực, phân quyền truy cập.

## 3. Mục đích của nút chủ (master) trong hệ phân tán
- Nút chủ quản lý, điều phối, phân công việc, tổng hợp kết quả. Hình dung như lớp trưởng trong lớp, phân công việc cho các bạn, tổng hợp kết quả. Nếu lớp trưởng nghỉ, cả lớp phải bầu lại người mới để mọi việc tiếp tục. Nếu nút chủ hỏng: hệ thống có thể dừng, hoặc phải bầu lại nút chủ mới (dùng thuật toán Bully, Ring...). Một số hệ thống dùng nhiều nút chủ dự phòng (backup master) để tránh bị gián đoạn.
- Trong thực tế, các hệ thống lớn như Google, Facebook thường có nhiều master để dự phòng, tránh "single point of failure".

## 4. Vì sao dùng gossip protocol?
- Gửi thông tin cho tất cả máy sẽ tốn băng thông, dễ nghẽn mạng. Gửi về trung tâm thì nếu trung tâm hỏng là "toang". Gossip: mỗi máy chỉ truyền cho vài máy khác, thông tin lan dần, tiết kiệm tài nguyên, chống nghẽn mạng. Gossip giống như kiểu "truyền miệng" trong lớp, một bạn biết tin gì sẽ kể cho vài bạn khác, rồi các bạn đó lại kể tiếp. Nhờ vậy, thông tin lan rất nhanh mà không bị nghẽn.
- Gossip protocol còn giúp hệ thống phát hiện lỗi nhanh, ví dụ khi một node "im lặng" lâu quá thì các node khác sẽ báo động. Các hệ thống như Cassandra, DynamoDB đều dùng gossip để đồng bộ trạng thái giữa các node.

## 5. Yếu tố cốt lõi của hệ phân tán
- Nhiều nút (máy) cùng làm việc, giao tiếp qua mạng, không phụ thuộc vào một điểm, có thể mở rộng, chịu lỗi. Ngoài ra còn có tính nhất quán (consistency): dữ liệu ở các node phải đồng nhất hoặc có cơ chế xử lý khi không đồng nhất. Hệ phân tán thường phải giải quyết các vấn đề như: làm sao để các node tin tưởng nhau, làm sao để dữ liệu không bị mất khi một node "ra đi". Ví dụ, khi một node lưu trữ dữ liệu bị hỏng, hệ thống phải tự động phục hồi từ các node khác.

## 6. Lý do sử dụng hệ phân tán
- Xử lý dữ liệu lớn, tăng tốc độ, chia sẻ tài nguyên, tăng độ tin cậy, chịu lỗi, dễ mở rộng. Ngoài ra, hệ phân tán còn giúp tăng tính sẵn sàng (availability), ví dụ như các dịch vụ ngân hàng online luôn hoạt động 24/7 nhờ có nhiều server dự phòng. Hệ phân tán còn giúp tiết kiệm chi phí, vì có thể tận dụng nhiều máy tính giá rẻ thay vì đầu tư một siêu máy tính đắt tiền. Các công ty lớn như Google, Amazon đều dùng hệ phân tán để phục vụ hàng tỷ người dùng.

## 7. Định nghĩa hệ phân tán
- Hệ thống gồm nhiều máy tính độc lập, phối hợp với nhau để giải quyết một bài toán chung, nhìn như một hệ thống duy nhất với người dùng. Hệ phân tán có thể là các máy tính ở xa nhau về mặt địa lý, kết nối qua Internet hoặc mạng LAN. Người dùng chỉ thấy một hệ thống duy nhất, không cần quan tâm bên trong có bao nhiêu máy. Ví dụ, khi bạn tìm kiếm trên Google, thực ra có hàng ngàn server cùng xử lý, nhưng bạn chỉ thấy một kết quả duy nhất.

## 8. Hình các thuật toán đồng bộ (Cristian, Berkeley, RBS, Lamport, Bully, Ring)
- Các thuật toán này giúp các node trong hệ phân tán đồng bộ thời gian hoặc bầu chọn leader. Ví dụ: thuật toán Bully sẽ có các node gửi tin nhắn "tôi muốn làm chủ" cho nhau, ai lớn nhất sẽ làm chủ. Thuật toán Ring thì các node xếp thành vòng tròn, truyền thông điệp theo vòng để bầu chủ. Nên tìm hiểu thêm các hình minh họa về cách các thuật toán này hoạt động, có thể vẽ lại bằng draw.io hoặc lấy từ slide môn học.

## 9. Kỹ thuật phân tán hỗ trợ lập trình gì?
- **Lập trình thủ tục:** RPC, gRPC giúp gọi hàm từ xa như gọi hàm local, ẩn đi chi tiết mạng.
- **Lập trình web:** REST, GraphQL, SOAP, AJAX giúp các hệ thống giao tiếp qua HTTP, dễ tích hợp với nhau.
- **Hướng đối tượng:** RMI (Java), CORBA cho phép gọi phương thức đối tượng từ xa.
- Ngoài ra còn có message queue (RabbitMQ, Kafka) giúp các hệ thống giao tiếp không đồng bộ, tăng hiệu suất. Các framework như Hadoop, Spark cũng là ví dụ về lập trình phân tán xử lý dữ liệu lớn.

## 10. Tiến trình nhẹ, tiến trình, luồng: ưu nhược điểm
- **Tiến trình (process):** Độc lập, an toàn, tốn tài nguyên, tạo chậm. Mỗi process có không gian nhớ riêng, nếu một process "chết" thì không ảnh hưởng process khác.
- **Luồng (thread):** Nhẹ, chia sẻ bộ nhớ, tạo nhanh, dễ lỗi nếu không đồng bộ. Thread nằm trong process, chia sẻ tài nguyên nên dễ bị lỗi race condition nếu không cẩn thận.
- **Tiến trình nhẹ (lightweight process):** Gần giống thread, nhưng quản lý bởi user-level, chuyển đổi nhanh, nhưng nếu 1 thread bị block thì cả tiến trình bị block.
- **Khi gọi hệ thống dừng:**
  - Process: chỉ process đó dừng
  - Thread: chỉ thread đó dừng, process vẫn chạy
  - Lightweight process: có thể ảnh hưởng cả nhóm
- Trong thực tế, các web server hiện đại thường dùng multithread hoặc lightweight process để phục vụ nhiều client cùng lúc, tránh bị "nghẽn cổ chai". Khi lập trình song song, phải chú ý đến deadlock (kẹt tài nguyên), race condition (tranh chấp dữ liệu).

## 11. Mô hình client-server
- **Client:** Gửi yêu cầu, nhận kết quả. **Server:** Xử lý yêu cầu, trả kết quả. VD: Trình duyệt là client, web server là server. Ngoài web, mô hình này còn áp dụng cho game online, ứng dụng chat, email... Một server có thể phục vụ hàng ngàn client cùng lúc, nhờ vào đa luồng hoặc đa tiến trình. Khi thiết kế server, phải chú ý đến bảo mật, phân quyền, chống tấn công DDoS.

## 12. Gọi thủ tục từ xa (RPC)
- Cho phép gọi hàm ở máy khác như gọi hàm local, ẩn đi chi tiết mạng, giúp lập trình dễ hơn. RPC giúp các hệ thống khác nền tảng (Windows, Linux, Mac) giao tiếp với nhau dễ dàng. gRPC là phiên bản hiện đại, hỗ trợ nhiều ngôn ngữ, truyền dữ liệu nhanh nhờ dùng protocol buffer. Khi dùng RPC, phải chú ý đến lỗi mạng, timeout, retry khi gọi hàm thất bại.

## 13. Mục đích mô hình phân tầng giao thức
- Chia nhỏ hệ thống thành các tầng, mỗi tầng làm một việc, dễ bảo trì, nâng cấp, thay thế. Mô hình OSI có 7 tầng: vật lý, liên kết, mạng, vận chuyển, phiên, trình diễn, ứng dụng. Nhờ phân tầng, khi nâng cấp một tầng (ví dụ đổi giao thức mạng) không ảnh hưởng các tầng khác. Hướng thông điệp bền vững: đảm bảo thông điệp không bị mất khi truyền.

## 14. Sharding là gì?
- Chia nhỏ dữ liệu thành nhiều phần (shard), lưu ở nhiều máy khác nhau. Giúp tăng tốc độ truy cập, mở rộng dễ dàng. Sharding thường dùng trong các hệ thống lớn như Facebook, Google, giúp chia nhỏ database thành nhiều phần, mỗi phần lưu ở một server khác nhau. Khi tìm kiếm, hệ thống sẽ xác định dữ liệu nằm ở shard nào để truy vấn nhanh hơn. Tuy nhiên, phải có cơ chế cân bằng tải, backup giữa các shard.

## 15. Gói luồng (thread package) làm gì?
- Tạo, quản lý, đồng bộ luồng, chia sẻ tài nguyên giữa các luồng, hỗ trợ lập trình song song. Gói luồng cung cấp API cho lập trình viên tạo, quản lý, đồng bộ luồng dễ dàng hơn. Ví dụ: thư viện pthreads trong C, threading trong Python. Khi dùng thread package, phải chú ý đến deadlock, race condition.

## 16. Luồng người dùng vs luồng nhân
- **Luồng người dùng:** Quản lý bởi user, chuyển đổi nhanh, nhưng nếu 1 thread bị block thì cả process bị block. Thường dùng trong các ứng dụng cần chuyển đổi luồng nhanh, như web server, game server.
- **Luồng nhân:** Quản lý bởi OS, chuyển đổi chậm hơn, nhưng thread block không ảnh hưởng thread khác. Phù hợp cho các tác vụ cần bảo mật, ổn định cao, như hệ điều hành, database. Một số hệ điều hành hỗ trợ hybrid thread (kết hợp user và kernel thread) để tận dụng ưu điểm của cả hai.

## 17. Các hàm chính trong RPC
- `rpc_register`: Đăng ký hàm
- `rpc_call`: Gọi hàm từ xa
- `rpc_send`, `rpc_receive`: Gửi/nhận dữ liệu
- `rpc_init`, `rpc_finalize`: Khởi tạo, kết thúc môi trường RPC
- Ngoài các hàm trên, còn có các hàm xử lý lỗi, kiểm tra trạng thái kết nối, log giao dịch để debug khi có sự cố. Khi lập trình RPC, phải kiểm tra kỹ lỗi mạng, timeout, retry.

## 18. Định nghĩa tiến trình, thread, multithread client/server
- **Tiến trình:** Chương trình đang chạy, có không gian nhớ riêng. Nếu một tiến trình bị lỗi thì không ảnh hưởng tiến trình khác.
- **Thread:** Đơn vị nhỏ nhất của CPU, chia sẻ bộ nhớ với process. Thread giúp tận dụng tối đa CPU đa nhân.
- **Multithread client/server:** Client/server có nhiều thread cùng xử lý song song. Multithread client/server giúp tăng hiệu suất xử lý, giảm thời gian chờ của client. Khi thiết kế multithread, phải chú ý đến việc chia sẻ tài nguyên, tránh deadlock.

## 19. Map, Reduce trong hệ phân tán
- **Map:** Chia nhỏ dữ liệu, xử lý song song trên nhiều node. **Reduce:** Tổng hợp kết quả từ các node. Mục đích: Xử lý dữ liệu lớn nhanh hơn. MapReduce là nền tảng của các hệ thống xử lý dữ liệu lớn như Hadoop, Google BigQuery. Map chia nhỏ công việc, Reduce tổng hợp kết quả, giúp xử lý hàng TB dữ liệu trong thời gian ngắn. Khi lập trình MapReduce, phải chú ý đến việc chia nhỏ dữ liệu hợp lý, tránh bottleneck ở bước Reduce.

## 20. Ảo hóa (Virtualization)
- Tạo nhiều máy ảo trên 1 máy thật, tận dụng tài nguyên, cô lập môi trường, dễ quản lý. Ảo hóa còn giúp chạy nhiều hệ điều hành trên cùng một máy, ví dụ vừa chạy Windows vừa chạy Linux. Docker là ví dụ về ảo hóa nhẹ (container), giúp triển khai ứng dụng nhanh, dễ di chuyển. Khi dùng ảo hóa, phải chú ý đến bảo mật giữa các máy ảo, phân bổ tài nguyên hợp lý.

## 21. Kiến trúc server đa luồng
- Server có nhiều thread, mỗi thread xử lý 1 client, tăng hiệu suất, phục vụ nhiều client cùng lúc. Server đa luồng giúp giảm thời gian chờ của client, tăng trải nghiệm người dùng. Tuy nhiên, phải quản lý đồng bộ dữ liệu giữa các thread, tránh lỗi race condition. Ngoài ra, phải có cơ chế giám sát, restart thread khi bị lỗi.

## 22. Hướng tiếp cận cài đặt luồng
- **User-level thread:** Quản lý bởi user, nhanh, nhưng block dễ chết cả process. **Kernel-level thread:** Quản lý bởi OS, an toàn hơn, nhưng chậm hơn. Một số hệ điều hành hỗ trợ hybrid thread (kết hợp user và kernel thread) để tận dụng ưu điểm của cả hai. Khi chọn mô hình thread, phải cân nhắc giữa hiệu suất và độ an toàn.

## 23. Bảng băm phân tán, Consistent Hashing, Finger table
- **Bảng băm phân tán:** Lưu trữ dữ liệu trên nhiều node, mỗi node chịu trách nhiệm 1 phần. **Consistent Hashing:** Chia đều dữ liệu, khi thêm/bớt node không ảnh hưởng nhiều. **Finger table:** Bảng chỉ dẫn giúp tìm kiếm nhanh hơn trong DHT (ví dụ: Chord). Consistent Hashing giúp hệ thống mở rộng hoặc thu hẹp số lượng node mà không phải phân phối lại toàn bộ dữ liệu. Finger table giúp tìm kiếm dữ liệu trong mạng ngang hàng (P2P) nhanh hơn, giảm số bước tìm kiếm. Khi thiết kế DHT, phải chú ý đến việc cập nhật finger table khi thêm/bớt node.

## 24. Không gian phẳng, định danh
- **Không gian phẳng:** Mỗi node có 1 ID duy nhất, không phân cấp. Đặc điểm: Đơn giản, dễ mở rộng, nhưng tìm kiếm chậm nếu không có chỉ mục. Không gian phẳng giúp hệ thống dễ mở rộng, nhưng khi số lượng node quá lớn thì phải có cơ chế tìm kiếm hiệu quả (như DHT, finger table).

## 25. Đồng bộ hóa đồng hồ logic, vật lý
- **Đồng hồ vật lý:** Không đảm bảo chính xác tuyệt đối do trôi giờ, thường dùng trong các hệ thống cần thời gian thực (real-time), như giao dịch tài chính. **Đồng hồ logic:** Đảm bảo thứ tự sự kiện, dùng cho hệ phân tán. Mục đích: Đảm bảo các sự kiện xảy ra đúng thứ tự, tránh xung đột. Thuật toán: Lamport, Vector clock, Cristian, Berkeley... Đồng hồ logic dùng trong hệ phân tán để xác định thứ tự sự kiện, tránh xung đột khi nhiều node cùng cập nhật dữ liệu.

## 26. Đồng hồ Lamport
- **Giải quyết:** Thứ tự sự kiện trong hệ phân tán. **Rules:**
  1. Mỗi sự kiện local tăng đồng hồ lên 1
  2. Khi gửi message, gửi kèm giá trị đồng hồ
  3. Khi nhận message, đồng hồ = max(local, nhận được) + 1
- Lamport clock không đảm bảo đồng bộ tuyệt đối, nhưng đảm bảo thứ tự sự kiện, rất quan trọng trong hệ phân tán. Vector clock là phiên bản nâng cấp, giúp xác định quan hệ nhân quả giữa các sự kiện. Khi làm bài tập, nên vẽ bảng thời gian cho từng tiến trình, áp dụng đúng 3 rule của Lamport để tính giá trị đồng hồ.

## 27. Giao thức đồng bộ NTP, PTP
- **NTP (Network Time Protocol):** Đồng bộ giờ qua Internet, sai số vài ms. **PTP (Precision Time Protocol):** Đồng bộ chính xác hơn, dùng trong công nghiệp, sai số nhỏ hơn 1ms. **Cách tính:** Gửi/nhận message, đo thời gian đi/về, tính toán độ trễ, điều chỉnh đồng hồ local. NTP phổ biến trên Internet, giúp các server đồng bộ giờ với nhau, tránh sai lệch khi xử lý giao dịch. PTP dùng trong các hệ thống công nghiệp, yêu cầu độ chính xác cao như nhà máy, trạm phát sóng. Khi triển khai NTP/PTP, phải chú ý đến độ trễ mạng, đồng bộ nhiều tầng server.

---

**Nguồn:** Slides môn Hệ Phân Tán, tài liệu tham khảo, kinh nghiệm học tập, tổng hợp từ thực tế và các hệ thống lớn.