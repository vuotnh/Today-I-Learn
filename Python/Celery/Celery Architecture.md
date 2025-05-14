```table-of-contents
```

# Kiến trúc mức high-level

Celery hoạt động theo mô hình **Producer-Consumer** với các thành phần chính:
- **Client (Producer)**: Gửi tác vụ đến hàng đợi.
- **Broker (Message Queue)**: Trung gian lưu trữ và định tuyến tin nhắn.
- **Worker (Consumer)**: Xử lý tác vụ từ hàng đợi.
- **Result Backend**: Lưu trữ kết quả tác vụ (tùy chọn).
- **Celery Beat**: Lập lịch tác vụ định kỳ.

**a. Client (Task Producer)**
- Định nghĩa và gọi tác vụ thông qua `delay()` hoặc `apply_async()`.
```python
@app.task
def add(x, y):
    return x + y

add.delay(4, 5)  # Gửi tác vụ đến broker
```
**b. Message Broker**
- Đóng vai trò trung gian, lưu trữ tin nhắn tác vụ.
- Hỗ trợ các hệ thống:
    - **RabbitMQ** (khuyến nghị, hỗ trợ AMQP).
    - **Redis**, **Amazon SQS**, hoặc các hàng đợi khác.

**c. Worker** 
- Tiến trình xử lý tác vụ, gồm các thành phần
    - **Task Pool**: Quản lý concurrency (multiprocessing, eventlet, gevent).
    - **Mediator**: Điều phối việc lấy tác vụ từ broker.
    - **Queues**: Các hàng đợi mà worker đăng ký (default, high_priority, ...).
- Khởi chạy bằng lệnh: `celery -A app worker --loglevel=info`.
    

**d. Result Backend**
- Lưu trữ trạng thái và kết quả tác vụ.
- Hỗ trợ: Redis, RabbitMQ, Django ORM, SQLAlchemy, hoặc các cơ sở dữ liệu.
- Truy vấn kết quả qua `AsyncResult`:    
```python
result = add.delay(4, 5)
print(result.get(timeout=10))  # Chờ kết quả
```

**e. Celery Beat**
- Lập lịch tác vụ định kỳ (ví dụ: gửi báo cáo hàng ngày).
- Cấu hình trong `app.conf.beat_schedule`.

# Workflow mức low-level

- **Producer → Broker → Worker → Result Backend**
    - **Producer**: Serialize task → Gửi message (dạng JSON/msgpack) đến broker.
    - **Broker**: Lưu message trong queue (ví dụ: `celery` queue mặc định).
    - **Worker**:
        - **Mediator**: Dùng `celery.worker.consumer.Consumer` để lấy message từ broker.
        - **Task Execution**: Deserialize message → Gọi hàm task → Xử lý exception.
        - **Acknowledgment (ACK)**: Gửi ACK đến broker khi task hoàn thành (tránh reprocessing).
    - **Result Backend**: Lưu kết quả dưới dạng key-value (task ID làm key).

| Component      | Mô tả                                               |
| -------------- | --------------------------------------------------- |
| **Task Pool**  | Quản lý concurrency (prefork, eventlet, gevent).    |
| **Event Loop** | Xử lý I/O events (dùng cho eventlet/gevent).        |
| **Timer**      | Điều khiển retries và scheduled tasks.              |
| **State DB**   | Lưu trạng thái worker (dùng SQLite hoặc in-memory). |

# Tham khảo

https://nirjalpaudel.com.np/posts/@nirjalpaudel54312/from-yin-to-yang-understanding-celery/#understanding-the-basic-architecture-of-celery

