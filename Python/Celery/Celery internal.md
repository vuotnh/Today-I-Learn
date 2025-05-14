```table-of-contents
```

![[image.webp]]
# Celery là gì ?

`Task queues` được sử dụng như một cơ chế để `distribute work` thông qua các threads hoặc các machines.
***Celery*** giao tiếp thông qua các messages, sử dụng một ***broker*** để làm trung gian giữa clients và workers.
Một ***Celery system*** có thể bao gồm nhiều workers và nhiều brokers để duy trì HA và horizontal scaling.

# Tại sao nên sử dụng Celery

 * ***Task queue management***: celery giúp quản lý các background jobs, offload các long running task để tránh delay user response
 * ***Asynchronous Processing***: Celery có thể thực thi các task một các bất đồng bộ, thực thi các task mà không block main application
 * ***Distributed Task Execution***: có thể scale horizontally thông qua nhiều worker ⇒ có thể xử lý được lượng lớn task.
 * ***Scheduled Jobs***: Sử dụng celery để lập lịch cho các task chạy hằng ngày.
 * ***Reliability***: Celery giúp chắc chắn các task sẽ được thực thi, sẽ retry khi có failure và cung cấp monitoring

# Kiến trúc cơ bản của Celery

![[Pasted image 20250505062023.png]]

## 1. Transport

***Celery transport*** chịu trách nhiệm ***lưu trữ các task argument và các task states***. Ngoài ra còn lưu thêm một số các thông tin khác của task ví dụ: name, uuid, runtime, no of retries,...

Một số loại ***celery transport*** phổ biến:
* Redis
* RabbitMQ
* Kafka

## 2. Task Serialization/Deserialization

***Task serialization và deserialization*** chịu trách nhiệm convert các task thành các format chuẩn để có thể gửi qua từng loại broker, và các worker có thể hiểu được yêu cầu của các task.
Hai quy trình trên giúp đảm bảo các task được gửi, lưu trữ và thực thi trong hệ thống phân tán.

Một vài loại ***task serialization format*** thường được sử dụng:
* JSON
* Pickle
* YAML
* Msgpack

## 3. Consumer / Worker

***Consumer / Worker*** sẽ nhận task input từ `transport`, xử lý các task. ***Xử lý tasks*** có nghĩ là thực thi một function với `paramenters` được gửi kèm theo.
Task sẽ kết thúc khi function đó chạy xong hoặc failed.

Khi một task bị failed, `Celery worker` sẽ trả về message cho `transport`.
`transport` sẽ tăng số lần retries của task đó lên, và worker sẽ chạy lại task đó (***cơ chế retries***)

```python
from celery import shared_task

@shared_task
def adder(num1, num2):
	sleep(2)
	return num1 + num2
```

## 4. Producer

***Producer*** có thể là bất kỳ thành phần nào trong hệ thống, gọi một tác vụ Celery. Nó có thể là một hàm trong view, model method, một lịch định kỳ Celery beat, hoặc một script python.

***Producer*** sẽ call các `celery task` kèm theo các `argument` tương ứng của task đó, sử dụng `.delay()`. Khi gọi tới `.delay()`, task sẽ được gửi tới hàng đợi broker

```python
from .tasks import adder

def run():
    res = adder.delay(num1=12, num2=13)
    return res
```
- `adder.delay(...)` sẽ **không chạy ngay lập tức**, mà gửi task tới Celery queue.
- Một **worker Celery đang chạy** sẽ nhận task này và xử lý nó ở nền.
- `res` là một `AsyncResult` object, bạn có thể dùng `.get()` để lấy kết quả nếu cần, ví dụ `res.get()`.

## 5. Một số component khác của celery

### Celery beat

***Celery beat*** là một ***scheduler***, nó sẽ lập lịch cho các task định kỳ trong một khoảng thời gian và được thực thi bởi một worker nào đó đang available.
 Được sử dụng để lập lịch cho các task chạy định kỳ.
```python
# tasks.py

from celery import Celery
from celery.schedules import crontab

app = Celery('my_app')

@app.task
def periodic_task():
    print("Running periodic task!")

app.conf.beat_schedule = {
    'run-every-hour': {
        'task': 'periodic_task',
        'schedule': crontab(minute=0, hour='*'),  # Executes every hour
    },
}
```

```bash
celery -A your_project beat --loglevel=info
```

### Result backend

***Result backend*** là nơi lưu trữ toàn bộ các ***tasks result*** đã được thực thi. Nó lưu nhiều thông tin khác của task bao gồm:
1. ***Task result*** : lưu kết quả trả về của một task được được thực thi
2. ***Task state***: lưu lại các trạng thái của task như PENDING, STARTED, SUCCESS, FAILURE, RETRY,...
3. ***Task chaining information***: một số thông tin khác ví dụ như là root task của task hiện tại, .....

Một vài celery result backend phổ biến:
1. Memory: lưu các thông tin trong memory của worker
2. Redis
3. RabbitMQ
4. Django database

### Celery monitoring with flower


# Celery concurrency

***Celery concurrency*** cho phép một worker xử lý nhiều task cùng lúc bằng cách sử dụng nhiều tiến trình (multiprocessing) hoặc nhiều luồng (multithreading).
Dưới đây là một số mô hình Concurrency trong celery
## Mô hình Pre-fork (mặc định)

Đây là mô hình concurrency mặc định trong celery, nó tạo ra nhiều tiến trình con, mỗi tiến trình sẽ xử lý một task riêng biệt.

Ưu điểm: ổn định, tránh GIL trong python, phù hợp cho các tác vụ  CPU-bound hoặc hỗn hợp

```bash
celery -A your_app worker --concurrency=4
```
➡️ Worker sẽ dùng 4 process để xử lý các task song song

## Mô hình eventlet và getevent

### 🔷 1. `eventlet` và `gevent` là gì?
- Đây là **các thư viện asynchronous I/O** (xử lý I/O bất đồng bộ).
- Thay vì tạo nhiều tiến trình như pre-fork, chúng dùng cái gọi là **"greenlet"** (luồng nhẹ).
- Phù hợp cho các task **chờ I/O nhiều** như:
    - Gọi API HTTP
    - Truy vấn CSDL
    - Đọc ghi file, socket...

> 🎯 Ưu điểm: Có thể xử lý **hàng trăm hoặc hàng nghìn task song song** mà **không tốn nhiều RAM/CPU** như pre-fork.

### 🔶 2. Hạn chế khi dùng `eventlet` hoặc `gevent`

Khi bạn **chuyển từ pre-fork sang eventlet hoặc gevent**, một số tính năng trong Celery **sẽ không còn hoạt động**, ví dụ:
#### ❌ `soft_timeout` bị vô hiệu hóa

- `soft_timeout`: là timeout mềm của task — task sẽ bị **cảnh báo** (thường bằng `SoftTimeLimitExceeded`) để dừng lại.
- Trong `eventlet` và `gevent`, Celery **không thể can thiệp "mềm"** vào task đang chạy vì không có process thật → nên `soft_timeout` **bị vô hiệu hóa âm thầm**.
#### ❌ `max_tasks_per_child` cũng không hoạt động

- Tính năng này dùng trong pre-fork để tự động **kết thúc worker sau khi chạy X task**, nhằm tránh memory leak.
- Vì `eventlet/gevent` **chỉ có 1 process**, không có child process → tính năng này **không có tác dụng**.
### ⚠️ Cảnh báo quan trọng

> Khi bạn dùng `--pool=eventlet` hoặc `--pool=gevent`, Celery **không báo lỗi rõ ràng** nếu các tính năng như `soft_timeout` hay `max_tasks_per_child` bị vô hiệu hóa → rất dễ khiến bạn **nghĩ rằng chúng vẫn đang hoạt động**, trong khi thực tế thì không.

## Mô hình Solo

Các task sẽ được xử lý tuần tự trong main thread của task. Mô hình này phù hợp để chạy các task đơn giản, lightweight task. Ngoài ra có thể sử dụng mô hình này để debug logic task. Tuy nhiên mô hình này không thể scale với số lượng lớn.

## Mô hình sử dụng multithreads

Celery có thể sử dụng các `threads` để thực thi các task song song thay vì sử dụng `processes` như mô hình pre-fork.

Mô hình thread-base này sử dụng `ThreadPoolExecutor` từ `concurrent.futures`, không cần cài thêm thư viện ngoài.
Worker ***chỉ chạy một processs*** và ***các task sẽ được thực thi trong các thread của process đó***. Các thread cùng chia sẻ bộ nhớ.

Threads rất phù hợp cho các **task I/O-bound**, tức là các task dành nhiều thời gian **chờ đợi** thay vì xử lý tính toán:
- Gọi HTTP API
- Đọc/ghi file
- Giao tiếp mạng

→ Trong lúc một thread đang chờ I/O, các thread khác vẫn có thể làm việc.

### ⚠️ Nhược điểm và cảnh báo:

- Vì các thread dùng **chung vùng nhớ**, nên:
    - Dễ xảy ra **race condition** nếu nhiều thread cùng sửa đổi một biến.
    - Nếu không kiểm soát kỹ, có thể gây **deadlock**.

→ Cần cẩn trọng nếu task có cập nhật dữ liệu dùng chung (biến toàn cục, cache, đối tượng singleton...).

 🔸 **"GIL can create further problems"**
- Python (CPython) có **Global Interpreter Lock (GIL)**, chỉ cho phép **một thread thực thi Python bytecode tại một thời điểm**.
- Với **CPU-bound task**, GIL làm threads **không thực sự chạy song song**, gây nghẽn hiệu năng.
→ Vì thế, thread-based **không phù hợp** cho các tác vụ nặng về tính toán.

---
# Cấu hình worker pool thông qua ENV

## Cấu hình Celery limits 

### 🕒 1. **Soft Time Limit** — "giới hạn mềm"

- là thời gian tối đa (tính bằng giây) mà task **nên được hoàn thành**.  
    Khi hết thời gian này, Celery **gửi một cảnh báo** dưới dạng **exception**: `SoftTimeLimitExceeded`.
    
- Task có thể **bắt exception này** để:
    - Thực hiện dọn dẹp (cleanup)
    - Lưu trạng thái tạm thời
    - Ghi log
    - Thoát một cách "êm đẹp"

```python
from celery.exceptions import SoftTimeLimitExceeded

@shared_task(soft_time_limit=5)
def long_task():
    try:
        do_something()
    except SoftTimeLimitExceeded:
        cleanup()
        return "Stopped early"

```


### 💣 2. **Hard Time Limit** — "giới hạn cứng"

- Thời gian tối đa tuyệt đối mà task **phải kết thúc**. Nếu task vượt quá giới hạn này, nó sẽ bị **kill ngay lập tức**.
    
- Nếu vượt quá giới hạn thời gian này:
    - Celery gửi tín hiệu `SIGTERM` hoặc `SIGKILL` đến process đang chạy task.
    - Không có cơ hội dọn dẹp, xử lý lỗi hay lưu trạng thái gì cả.
    - Thường được dùng để đảm bảo task **không bao giờ chạy vượt quá một ngưỡng nguy hiểm**.

### ⏱️ **Quan hệ giữa soft và hard time limit**

> **Soft limit luôn được kiểm tra trước hard limit.**

- Bạn có thể thiết lập cả hai cùng lúc, ví dụ:
```python
@shared_task(soft_time_limit=10, time_limit=15)
def some_task():
    ...
```

→ Task này:
- Có **5 giây** để xử lý sau khi bị cảnh báo soft timeout.
- Nếu sau 15 giây vẫn chưa xong, sẽ bị **kill ngay lập tức**.

| Thuộc tính             | Soft Time Limit                         | Hard Time Limit                         |
| ---------------------- | --------------------------------------- | --------------------------------------- |
| Mục đích               | Cảnh báo để task tự thoát               | Cưỡng bức dừng task                     |
| Cách xử lý             | Raise exception (SoftTimeLimitExceeded) | Gửi `SIGTERM` / `SIGKILL`               |
| Có cleanup được không? | ✅ Có                                    | ❌ Không                                 |
| Thường dùng khi nào?   | Cho phép task dọn dẹp an toàn           | Ngăn task chạy quá lâu/lỗi nghiêm trọng |

## Celery task configuration ([tham khảo](https://github.dev/celery/celery/blob/main/celery/app/task.py))

|Task Configuration|Description|Default Value|Notes|
|---|---|---|---|
|max_retries|Maximum number of retries before giving up|3|If set to None, it will never stop retrying|
|default_retry_delay|Default time in seconds before a retry|180 (3 minutes)|Determines wait time between retries|
|rate_limit|Rate limit for this task type|None (no rate limit)|Examples: ‘100/s’, ‘100/m’, ‘100/h’|
|ignore_result|If enabled, worker won’t store task state and return values|None|Defaults to task_ignore_result setting|
|time_limit|Hard time limit for task execution|None|Defaults to task_time_limit setting|
|serializer|Serializer that will be used|json|json, pickle|
|soft_time_limit|Refer to time limits|:setting:`task_soft_time_limit`||

Example 1 (không nên sử dụng)
```python
from celery import shared_task
from celery.exceptions import SoftTimeLimitExceeded


@shared_task(
    max_retries=3,                     # Maximum retries
    default_retry_delay=180,            # Default retry delay (in seconds
    rate_limit=None,                    # No rate limit
    ignore_result=False,                # Store task state and result
    time_limit=None,                    # No hard time limit
    soft_time_limit=None,               # No soft time limit
    serializer='json'                   # Use JSON for serialization
)
def example_shared_task(*args, **kwargs):
    try:
        # Task logic here
        print(f"Processing task with args: {args} and kwargs: {kwargs}")
        # Simulate long processing task
    except SoftTimeLimitExceeded:
        print("Task exceeded the soft time limit")
        # Handle cleanup if necessary
```

DRY code (DONT REPEAT YOUR SELF)

```python
class BaseTaskWithRetry(Task):
    autoretry_for = (Exception,)
    retry_kwargs = {"max_retries": 5}
    retry_backoff = True
    retry_backoff_max = 60 * 5  # 5 minutes
    retuy_jitter = True

    def on_failure(self, exc, task_id, args, kwargs, einfo):
        super().on_failure(exc, task_id, args, kwargs, einfo)
        self.retry(exc=exc, countdown=60 * 5)



class BaseFor10HoursTask(Task):
    autoretry_for = (Exception,)
    retry_kwargs = {"max_retries": 10}
    retry_backoff = True
    retry_backoff_max = 60 * 60 * 10  # 10 hours
    retuy_jitter = True


    def on_failure(self, exc, task_id, args, kwargs, einfo):
        super().on_failure(exc, task_id, args, kwargs, einfo)
        self.retry(exc=exc, countdown=60 * 60 * 10)



class BaseFor24HoursTask(Task):
    autoretry_for = (Exception,)
    retry_kwargs = {"max_retries": 60}
    retry_backoff = True
    retry_backoff_max = 60 * 60 * 24  # 24 hours
    retuy_jitter = True


    def on_failure(self, exc, task_id, args, kwargs, einfo):
        super().on_failure(exc, task_id, args, kwargs, einfo)
        self.retry(exc=exc, countdown=60 * 60 * 24)

```