```table-of-contents
```

# Thiết lập kết nối TCP thông thường

- Là giao thức có kết nối (connection-oriented) → thiết lập và duy trì kết nối giữa 2 thiết bị trước khi truyền dữ liệu
- Đảm bảo độ tin cậy
- Trật tự: các gói tin gửi theo đúng thứ tự
- Quy trình hoạt động
    - Thiết lập kết nối, bắt tay 3 bước (Three-way Handshake)
        - **SYN**: Client gửi một gói tin SYN (synchronize) đến Server để yêu cầu kết nối
        - **SYN-ACK**: Server nhận được gói tin SYN, phản hồi lại bằng gói SYN-ACK (synchronize-acknowledge)
        - **ACK**: Client gửi gói tin ACK (acknowledge) để xác nhận kết nối thành công
        
    - Truyền dữ liệu
        - TCP chia dữ liệu thành các gói tin (packets)
        - Mỗi gói tin có một số thứ tự (sequence number) để đảm bảo dữ liệu được nhận đúng thứ tự
        - Khi nhận dữ liệu, bên nhận gửi lại một thông báo xác nhận (ACK). Nếu không nhận được, TCP sẽ gửi lại gói tin đó
        
    - Kết thúc kết nối (Four-way Handshake)
        - **FIN**: Một bên gửi gói FIN để yêu cầu kết thúc kết nối
        - **ACK**: Bên nhận phản hồi bằng một gói ACK
        - **FIN**: Bên nhận cũng gửi một gói FIN để yêu cầu đóng kết nối từ phía mình
        - **ACK**: Bên gửi ban đầu phản hồi lại bằng gói ACK

- Ứng dụng
    - HTTP/HTTPS
    - FTP
    - SMTP/IMAP/POP3 (email)
    - SSH

### Tại sao thiết lập kết nối TCP lại phải là 3 bước mà ko phải là 2 hay 4 bước

## ✅ **Câu trả lời ngắn gọn:**

**Three-way handshake** trong TCP đảm bảo rằng:

1. **Cả client và server đều sẵn sàng giao tiếp**.
2. **Cả hai phía đều biết được rằng phía bên kia đã nhận đúng các gói SYN/SYN-ACK**.
3. **Tránh các lỗi do gói tin cũ (delayed or duplicate packets)**.

Hai bước là **thiếu an toàn**, bốn bước là **không cần thiết**.

---
## 📘 Giải thích chi tiết:

### 📍**3 bước bắt tay (Three-way handshake)** là:

**B1. Client → Server: SYN**
* Client gửi gói SYN để yêu cầu thiết lập kết nối, kèm theo **sequence number** ban đầu.

**B2. Server → Client: SYN-ACK**
* Server nhận SYN, trả lời bằng một gói **SYN-ACK**: xác nhận (ACK) gói SYN của client, và gửi SYN từ phía server (cũng kèm sequence number của server).
    
**B3. Client → Server: ACK**
* Client xác nhận lại SYN của server bằng gói ACK.

➡️ Sau bước 3, cả hai bên đều biết rằng **kết nối đã được thiết lập an toàn**, và **sequence numbers** đã được đồng bộ.

---
### ❌ **Tại sao không phải là 2 bước?**

Ví dụ: chỉ có **SYN → SYN-ACK**.
- Client không gửi ACK lại → server **không thể chắc chắn** client đã nhận được SYN-ACK.
- Có thể **SYN-ACK bị mất** hoặc **client đã chết** ngay sau khi gửi SYN → server nghĩ kết nối đã thiết lập, nhưng thực tế không có client.

➡️ **Nguy hiểm: Kết nối "ma" có thể tồn tại.**

---
### ❌ **Tại sao không cần 4 bước?**

Bạn có thể nghĩ đến việc chia SYN và ACK thành **hai gói riêng biệt** ở bước 2 và 3 (giống như: SYN → ACK → SYN → ACK). Nhưng:
- Việc **gộp SYN và ACK** thành một gói duy nhất trong bước 2 là **tối ưu** và **đủ thông tin**, không cần tách riêng.
- Bốn bước chỉ làm tăng độ trễ và số gói tin, **không tăng thêm độ tin cậy**.

➡️ **Ba bước là tối ưu giữa độ tin cậy và hiệu suất.**

---
# Thiết lập kết nối UDP

- Là giao thức không có kết nối (Connectionless), không cần kết nối trước khi gửi dữ liệu
- Không đảm bảo độ tin cậy, không có cơ chế kiểm tra lỗi hoặc xác nhận gọi tin, có thể mất dữ liệu
- Không đảm bảo thứ tự, gói tin có thể đến không theo thứ tự hoặc bị mất mà không có cơ chế phát hiện

- Quy trình hoạt động
    - UDP chỉ đơn giản lấy dữ liệu từ ứng dụng, đóng gói thành các **datagram**, và gửi đi mà không cần bắt tay (handshake) như TCP
    - Thiết bị nhận dữ liệu sẽ đọc gói tin nhưng không có cơ chế xác nhận hoặc gửi lại nếu mất gói

- Ứng dụng
    - Streaming video/audio (YouTube, Netflix, VoIP, Zoom)
    - Gaming online (PUBG, CS:GO)
    - DNS (tra cứu tên miền)
    - DHCP (gán địa chỉ IP)

## 📡 1. **Full-Duplex là gì?**

**Full-duplex** (song công toàn phần) là cơ chế truyền thông **hai chiều đồng thời**: cả hai bên có thể gửi và nhận dữ liệu **cùng lúc** trên cùng một kênh.

🔹 **Ví dụ đời thực:**
- Cuộc gọi điện thoại: bạn có thể nói và nghe cùng lúc.
- WebSocket: client và server có thể gửi/nhận dữ liệu bất kỳ lúc nào.

---


## 🔁 2. **Half-Duplex là gì?**


**Half-duplex** (song công bán phần) là cơ chế truyền thông **hai chiều nhưng không đồng thời**: chỉ một bên có thể truyền dữ liệu tại một thời điểm. Nếu một bên đang gửi, bên kia phải đợi đến lượt.

🔹 **Ví dụ đời thực:**
- Bộ đàm (walkie-talkie): chỉ nói hoặc nghe, không đồng thời.
- Giao tiếp HTTP truyền thống: client gửi request, server mới gửi response (1 chiều tại 1 thời điểm).

# Các loại kết nối giữa client và server

## Kết nối HTTP thông thường (Half-Duplex)

1. Client gửi request tới server
2. Server xử lý request, sau đó ***liền*** trả về response cho client

➡️ HTTP thông thường mặc định ***ko*** ***keep connection*** ↔ ngay sau khi server trả về response, tcp connection sẽ bị ngắt, khi có request tiếp theo sẽ phải khởi tạo lại tcp connection (bắt tay 3 bước)

❌ Gây lãng phí tài nguyên cho việc thiết lập connection

✅ Tuy nhiên có thể thiết lập `Connection: keep-alive` , khi đó tcp connection sẽ không bị hủy ngay sau khi response, mà sẽ ***được giữ lại trong một khoảng thời gian ngắn***, nếu trong khoảng thời gian đó, client có request mới, tcp connection sẽ được ***tái sử dụng***.

💡 Mặc định thì HTTP 1.1 auto bật keep-alive

Ví dụ cấu hình keep-alive trong http header

```scss

Connection: keep-alive

Keep-Alive: timeout=5, max=100

```

  

## Kết nối HTTP Polling (Half-Duplex)


Hoạt động tương tự như HTTP thông thường, tuy nhiên client sẽ ***gửi request định kỳ lên server*** để hỏi server có dữ liệu mới hay không.

- Nếu có → server trả về dữ liệu.
- Nếu không có → server vẫn trả về ngay (thường là rỗng).
- Client đợi một thời gian rồi gửi lại request khác.

Trong **HTTP Polling thông thường**, mỗi request:
- Mở kết nối TCP mới (nếu không dùng keep-alive),
- Gửi request,
- Nhận response,
- Và **đóng kết nối**.

Tương tự HTTP thông thường, nếu dùng `Connection: keep-alive` thì có thể tái sử dụng connection trong thời gian ngắn

```scss

[Client] --(GET /data)--> [Server]
[Server] <--(200 OK, data = null)-- [Client]
... 5s sau ...
[Client] --(GET /data)--> [Server]
```

⚠️ Nhược điểm:
- Gây lãng phí lượng lớn tài nguyên
- Gánh nặng về băng thông và xử lý request sẽ đồn lên server

## Kết nối HTTP Long Polling (Half-Duplex)

- Client gửi một http request đến server.
- Client **không đóng kết nối** mà **chờ server phản hồi** dữ liệu (đây là điểm khác biệt với polling thông thường).
- Nếu **server chưa có dữ liệu**, **nó giữ kết nối mở (pending)** trong một khoảng thời gian (ví dụ 30s, 60s).
- Nếu dữ liệu mới đến trong thời gian đó, server **ngay lập tức phản hồi**.
- Nếu hết timeout mà không có dữ liệu → server trả về rỗng và client gửi lại request khác.

Trong **HTTP Long Polling**, **client là bên chủ động giữ kết nối** chờ dữ liệu, còn **server chỉ phản hồi và đóng kết nối khi có dữ liệu hoặc timeout**.

```css

[Client] ----GET----> [Server]

                      [Server giữ trong 20s, nếu có data → trả về]

                      [Nếu không → trả về rỗng]

[Client nhận response → gửi GET mới ngay lập tức]

```

✅ ***Ưu điểm***: HTTP Polling sẽ cung cấp một API ổn định, mức độ phản hồi khi có thông tin mới rất nhanh, đáp ứng tiêu chí **realtime**.

❌ ***Nhược điểm***: tương tự như HTTP polling, long polling cũng chiếm băng thông nhiều (do keep connection để chờ phản hồi) ➡️ chu kỳ gửi request không diễn ra quá nhanh để khắc phục nhược điểm này

**Long polling là kỹ thuật trong đó client gửi yêu cầu HTTP đến server, nhưng server giữ kết nối mở cho đến khi có dữ liệu mới hoặc hết timeout. Sau khi phản hồi, client sẽ lập tức gửi yêu cầu mới.**

Kỹ thuật này giúp **giảm tải** so với HTTP polling thông thường bằng cách **giảm số request vô ích khi không có dữ liệu**, nhưng vẫn dựa trên mô hình **request-response truyền thống**.

Nhược điểm: server cần **duy trì nhiều kết nối mở đồng thời** và client cần xử lý timeout hoặc retry.

  

## Kết nối **Server-Sent Events** (Half-Duplex)
### Nguyên lý hoạt động

SSE hoạt động dựa trên kết nối HTTP giữ mở từ client đến server, qua đó server có thể gửi các bản tin mới (events) cho client mà không cần phải client phải gửi yêu cầu mới.
### **1. Khởi tạo kết nối**:

- Client sẽ gửi một yêu cầu HTTP tới server để mở kết nối SSE. Để chỉ định rằng đây là kết nối SSE, client sẽ gửi yêu cầu HTTP với header `Accept: text/event-stream`.
- Server sẽ đáp lại với header `Content-Type: text/event-stream`, báo hiệu cho trình duyệt biết rằng nó sẽ nhận dữ liệu theo định dạng SSE.
### 2. **Server giữ kết nối mở:**

- Server nhận request từ client và phản hồi với `Content-Type: text/event-stream`.
- Sau khi gửi header ban đầu, **server giữ kết nối mở** và bắt đầu gửi các sự kiện đến client dưới dạng các dòng dữ liệu.
    - **Server không đóng kết nối ngay lập tức** mà sẽ tiếp tục truyền dữ liệu khi có sự kiện mới (hoặc theo thời gian định kỳ).

### 3. **Dữ liệu được gửi từ server tới client:**

- Dữ liệu được gửi dưới dạng **sự kiện (events)**, với mỗi sự kiện được phân tách bởi dòng "data" hoặc "event":

    ```

    kotlin

    CopyEdit

    data: Một sự kiện mới đến từ server

    ```


### 4. **Kết nối không bị đóng:**

- **Server giữ kết nối mở** lâu dài để có thể tiếp tục gửi các sự kiện.
- Nếu kết nối bị mất (client hoặc server ngắt kết nối), ***client có thể tự động thử lại kết nối***.

## Websocket (Full-Duplex)

![image.png](image.png)

## 🧩 **1. WebSocket là gì?**


**WebSocket** là một giao thức mạng chuẩn (RFC 6455) cho phép **kết nối hai chiều (full-duplex)** giữa client và server qua một kết nối TCP duy nhất.

Không giống như HTTP request/response truyền thống (half-duplex), WebSocket cho phép cả hai bên gửi dữ liệu bất kỳ lúc nào.
## 🔄 **2. Quá trình bắt tay (Handshake)**
### 🔹 2.1. Khởi tạo từ client

WebSocket bắt đầu bằng một HTTP request đặc biệt gọi là "Upgrade request" (lên cấp từ HTTP sang WebSocket):

```
GET /chat HTTP/1.1
Host: example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Version: 13
```

- `Upgrade: websocket`: yêu cầu chuyển sang giao thức WebSocket.
- `Sec-WebSocket-Key`: chuỗi Base64, server dùng để tạo response hash xác thực.
- `Sec-WebSocket-Version`: version đang dùng (thường là 13).
### 🔹 2.2. Server phản hồi

Server xác nhận nâng cấp bằng:
```yaml
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
```

- `Sec-WebSocket-Accept`: giá trị được mã hóa từ `Sec-WebSocket-Key` + GUID chuẩn.

Nếu server không hỗ trợ, nó trả HTTP 400/426.

---

## 📡 **3. Truyền dữ liệu sau khi kết nối**

Sau khi bắt tay thành công:

- Kết nối TCP vẫn mở. ***Cả hai bên đều keep connection***
- Cả server và client có thể gửi **WebSocket frames** qua lại bất kỳ lúc nào.
### 🔹 Frame bao gồm:

| Thành phần     | Ý nghĩa                                 |
| -------------- | --------------------------------------- |
| FIN bit        | Đánh dấu frame cuối                     |
| Opcode         | Kiểu dữ liệu (text, binary, ping, pong) |
| Mask           | Client luôn mask dữ liệu gửi lên        |
| Payload Length | Độ dài dữ liệu                          |
| Payload Data   | Dữ liệu thực tế                         |
  
## 🔐 **5. Bảo mật trong WebSocket**

- **`ws://`** → không mã hóa
- **`wss://`** → dùng TLS như HTTPS
## 🛠️ **Khi nào nên (và không nên) dùng WebSocket?**
  
### ✅ Nên dùng:

- Chat real-time
- Game online
- Live trading dashboard
- IoT device bi-directional communication
- Collaborative tools (whiteboard, Figma…)

### ❌ Không nên dùng:

- Nội dung tĩnh, ít cập nhật (blog, news)
- Trường hợp mà long polling hoặc SSE đủ dùng và đơn giản hơn