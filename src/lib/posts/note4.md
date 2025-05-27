---
title: "Distributed System notes 4"
date: "2025-05-16"
updated: "2025-05-16"
categories:
  - "sveltekit"
  - "markdown"
coverImage: "/images/image.png"
coverWidth: 16
coverHeight: 9
excerpt: Giai thich ve ISP, chan web va doi DNS o Viet Nam, van phong sinh vien.
---

# Một số câu hỏi về mạng ở Việt Nam

## 1. Các nhà mạng (ISP) lớn ở Việt Nam là ai?

Nếu bạn đang dùng Internet ở nhà hay ký túc xá, chắc chắn bạn sẽ gặp mấy cái tên quen thuộc này:
- **VNPT**: Nhà mạng quốc doanh, phủ sóng rộng, nhiều trường học, cơ quan nhà nước dùng.
- **Viettel**: Của quân đội, tốc độ ổn, giá cũng cạnh tranh, nhiều bạn sinh viên dùng.
- **FPT Telecom**: Dịch vụ chăm sóc khách hàng tốt, nhiều combo cho sinh viên, lắp đặt nhanh.
- **CMC Telecom, SCTV**: Ít phổ biến hơn, nhưng ở một số khu vực vẫn có.
- Ngoài ra còn có Mobifone, NetNam, SPT... nhưng thực tế sinh viên hay gặp nhất vẫn là 3 ông lớn trên.

## 2. Vì sao đôi lúc bị chặn truy cập một số web?

Chắc hẳn ai cũng từng bị: đang lướt Facebook, TikTok, hay tìm phim, tự nhiên web không vào được, báo lỗi "không thể truy cập" hoặc quay vòng mãi không load. Trong khi đó, bạn bè ở nước ngoài vẫn vào bình thường, thậm chí dùng VPN là vào được ngay.

Lý do là các nhà mạng ở Việt Nam sẽ chặn một số website theo yêu cầu của nhà nước (ví dụ: Bộ Thông tin & Truyền thông), hoặc do nội dung web đó vi phạm pháp luật Việt Nam (phim lậu, cá cược, tin giả, v.v.).

Cách chặn phổ biến nhất là **chặn DNS**. Khi bạn gõ tên web, trình duyệt sẽ hỏi máy chủ DNS của nhà mạng "web này IP là gì?". Nếu web bị chặn, DNS sẽ trả về IP sai, hoặc không trả về gì, nên bạn không vào được. Đôi khi còn bị chuyển hướng sang trang thông báo "website bị chặn theo quy định của pháp luật".

## 3. Tại sao đổi DNS sang 8.8.8.8 hay 1.1.1.1 lại vào được web bị chặn?

8.8.8.8 là DNS của Google, 1.1.1.1 là của Cloudflare. Khi bạn đổi DNS trên máy tính hoặc điện thoại sang mấy địa chỉ này, trình duyệt sẽ hỏi "web này IP là gì?" với Google hoặc Cloudflare thay vì hỏi nhà mạng Việt Nam. Các DNS quốc tế này thường không chặn web theo yêu cầu của Việt Nam, nên sẽ trả về IP thật của web, và bạn vào được bình thường.

Ví dụ: Bạn không vào được Facebook bằng mạng nhà, nhưng sau khi đổi DNS sang 8.8.8.8, tự nhiên vào được luôn. Đó là vì Google DNS không chặn Facebook như DNS của nhà mạng.

Tóm lại, đổi DNS là một cách "lách luật" để vượt qua việc chặn DNS của nhà mạng. Tuy nhiên, nếu nhà mạng chặn kiểu khác (ví dụ chặn IP, chặn ở tầng cao hơn, hoặc chặn bằng tường lửa), thì đổi DNS cũng không chắc vào được đâu nhé! Có lúc phải dùng thêm VPN hoặc proxy mới "qua mặt" được.

## 4. Lưu ý thực tế cho sinh viên

- Đổi DNS rất dễ, chỉ cần vào phần cài đặt mạng trên máy tính/điện thoại, nhập 8.8.8.8 hoặc 1.1.1.1 là xong.
- Đổi DNS giúp vào được nhiều web bị chặn, nhưng không phải lúc nào cũng thành công.
- Nếu mạng trường, ký túc xá chặn mạnh, có thể phải dùng VPN (nhưng nhớ chọn VPN uy tín, tránh lộ thông tin cá nhân).
- Một số web bị chặn vì lý do an toàn, nên cân nhắc trước khi cố truy cập nhé!

---

**Nguồn tham khảo:**
- Các bài báo về mạng ở Việt Nam, tài liệu của Google và Cloudflare về DNS.


