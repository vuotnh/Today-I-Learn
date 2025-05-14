```table-of-contents
```

# *Khái niệm cốt lõi*

## *Module*

### 1. Modules – Cấu trúc cơ bản trong Node.js

* Trong Nodejs, ***module*** là đơn vị cơ bản để tổ chức mã nguồn
* Một chương trình được chia nhỏ thành nhiều module, mỗi module đảm nhiệm một chức năng riêng biệt
➡️ Ứng dụng dễ phát triển, mở rộng và tái sử dụng

### 2. Triết lý: Modules nhỏ, phạm vi hẹp

* Module nên ***nhỏ*** không chỉ vì số dòng code, mà ***quan trọng hơn là về phạm vi chức năng.***
* Một module lý tưởng chỉ ***nên làm một việc***.

***npm*** và ***yarn*** là các công cụ quản lý module.
- Trước đây, việc nhiều package phụ thuộc vào các phiên bản khác nhau của cùng một thư viện thường gây xung đột (gọi là **dependency hell**).
- Node.js giải quyết vấn đề này bằng cách cho phép mỗi package có thể cài riêng phiên bản mà nó cần ⇒ Dùng hàng chục thư viện nhỏ mà ko lo xung đột phiên bản

---

# *Nodejs internal*

## I/O is slow

Việc truy xuất dữ liệu trong disk hoặc network tốn nhiều thời gian hơn so với truy xuất dữ liệu trong RAM. 
Bản chất việc xử lý I/O không tốn nhiều CPU, nhưng do nhiều process cùng chia sẻ (đọc, ghi) khiến I/O cần có thời gian chờ ⇒ gây ra độ trễ đáng kể

Trong thời gian chờ I/O, chương trình sẽ bị chặn nếu không có cơ chế bất đồng bộ ***non-blocking***

## Blocking I/O

Trong lập trình truyền thống, khi một function gửi một I/O request, nó sẽ ***block luồng thực thi cho tới khi việc xử lý I/O hoàn thành***.

➡️ Tốn nhiều thời gian, chương trình không thể handle nhiều tác vụ trong một thread


