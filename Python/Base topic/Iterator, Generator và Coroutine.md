```table-of-contents
```

# Iterator

## Iterable

Chúng ta có thể sử dụng `for` để duyệt qua phần tử của một list
```python
for i in [1, 2, 3, 4]:
	print(i)
```
Tương tự với `string`
```python
for c in "python":
	print(c)
```

Tương tự với một dict, chúng ta sẽ duyệt qua các key của dict
```python
for k in {"x": 1,  "y": 2}:
	print(k)
```
Nếu chúng ta sử dụng một file, duyệt qua các dòng trong file đó
```python
for line in open("a.txt"):
	print(line)
```

➡️Những đối tượng (object) có thể được sử dụng với `for...in` được gọi là những ***iterable object***

Thao tác duyệt các ***iterable object*** được gọi là ***iteration***

***Các iterable object có thể được sử dụng để lặp với `for...in` do chúng có implement phương thức `__iter__`***.


## Iterator

<span style="color:rgb(255, 0, 0)">Iterator</span> là một object trong python <span style="color:rgb(255, 0, 0)">tuân theo giao thức lặp</span>
Để được gọi là một ***iterator***, object đó ***phải có hai phương thức***:
* `__iter__()`: trả về chính đối tượng iterator
* `__next__()`: trả về phần tử tiếp theo. Nếu không còn phần tử nào nữa thì `StopIteration` exception sẽ được raise.

Ví dụ tạo một class cho Iterator object
```python
class CountDown:
    def __init__(self, start):
        self.current = start

    def __iter__(self):
        return self  # trả về chính nó là iterator

    def __next__(self):
        if self.current <= 0:
            raise StopIteration
        self.current -= 1
        return self.current + 1
```


⚠️ Các object `list, dict, string,...` trong Python chỉ là các `iterable object` chứ ko phải `iterator` vì chúng chỉ có phương thức `__iter__()` chứ ko có `__next__()`.

💡 Python cung cấp một `built-in function` là `iter()`, hàm này nhận đầu vào là một `iterable object` và trả về một `iterator`.
```python
my_list = [1, 2, 3, 4]
my_list_iter = iter(my_list)

my_list_iter.__next__()
```
💡 Sử dụng để lặp các `iterable object` thay cho `for..in`  → Cơ chế này giúp `for` có thể làm việc mới mọi `iterable`.
```python
for x in my_list:

# tương đương với
iter_my_list = iter(my_list)
while True:
	item = next(iter_my_list)
```

---

# `yield` là gì

**`yield`** là từ khóa dùng trong hàm để biến nó thành generator.

Khi gặp `yield`:
1. Hàm tạm dừng thực thi
2. Giá trị sau `yield` được trả về
3. **Trạng thái hàm được lưu** (biến cục bộ, vị trí thực thi)
4. Khi gọi lại, hàm tiếp tục từ vị trí `yield` cuối cùng

Ví dụ minh họa
```python
def simple_generator():
    print("Bắt đầu")
    yield 1
    print("Sau yield 1")
    yield 2
    print("Kết thúc")

gen = simple_generator()  # Không thực thi ngay, trả về một object generator

# Code chỉ được thực thi khi gọi next() và dừng khi gặp yield.
print(next(gen))  # Output: "Bắt đầu" → trả về 1
print(next(gen))  # Output: "Sau yield 1" → trả về 2
print(next(gen))  # Lỗi StopIteration
```

## Kỹ thuật `yield` có hai chiều

Bình thường `yield` thường được dùng để ***xuất dữ liệu ra ngoài*** (`return` tạm thời từng phần)
```python
def gen():
    yield 1
    yield 2

g = gen()
print(next(g))  # → 1
```

💡 ***Tuy nhiên đôi khi cũng có thể sử dụng yield để nhận dữ liệu***

***Trong coroutine***:

```python
def coroutine():
    while True:
        data = yield
        print("Nhận:", data)

c = coroutine()
next(c)
c.send("Xin chào")
```

Ở đây `yield` làm nhiệm vụ nhận dữ liệu
`c.send("xin chào")` sẽ:
- **Đưa chuỗi `"Xin chào"` vào** → nó sẽ được gán cho `data = yield`
- Generator chạy tiếp dòng sau `yield` → `print("Nhận:", data)`  → in `"Nhận: Xin chào"`

➡️ `send(value)` là cách để “gửi dữ liệu vào coroutine”, và `yield` sẽ nhận dữ liệu đó.

---
# Generator

`Generator` là một hàm chứa `yield`.
`Generator` là một cách đơn giản để tạo ra `iterator`
- **Generator** là một loại iterator đặc biệt trong Python, cho phép tạo giá trị **"lazy evaluation"** (tính toán khi cần).
- Khác với hàm thông thường (trả về kết quả và kết thúc), generator **tạm dừng (pause)** và **tiếp tục (resume)** thực thi khi cần.
## Tại sao nên sử dụng `generator`

### 1. Tiết kiệm bộ nhớ (Memory Efficiency)

***Vấn đề***: cần đọc dữ liệu trong 1 file lớn (ví dụ 1 file 1TB, đọc 10 triệu bản ghi) 
❌ Nếu sử dụng list thông thường, cần đọc và load hết dữ liệu vào trong RAM để xử lý → gây tràn ram

✅ Sử dụng generator, đọc, load từng dòng vào RAM và xử lý từng dòng

```python
def read_large_file(filename):
    with open(filename, 'r') as f:
        for line in f:  # Đọc từng dòng
            yield line  # Trả về từng dòng

# Sử dụng chỉ tốn RAM cho 1 dòng
for line in read_large_file("big_data.txt"):
    process(line)  # Xử lý từng dòng
```

Với đoạn code trên, đọc tới đâu, hàm `for..in` sẽ gọi `__next__()` tới đó để load dữ liệu tiếp theo ra để xử lý, không cần load toàn bộ file vào RAM

### 2. Xây dựng pipeline xử lý dữ liệu

Kết hợp nhiều generator thành một pipeline để xử lý dữ liệu tuần tự

```python
def filter_even(numbers):
    for n in numbers:
        if n % 2 == 0:
            yield n

def square(numbers):
    for n in numbers:
        yield n ** 2

# Pipeline: Số nguyên → Lọc chẵn → Bình phương
numbers = range(10)
pipeline = square(filter_even(numbers))

for result in pipeline:
    print(result)  # 0, 4, 16, 36, 64
```

### 3. 💡💡Coroutines và Đa nhiệm

`Generator` <span style="font-style:italic; font-weight:bold; color:rgb(0, 176, 80)">hỗ trợ coroutine (coroutine là các hàm có thể tạm dừng và tiếp tục)</span>, tiền thân tạo nên `async/await` trong python

```python
def coroutine():
    while True:
        data = yield  # Nhận dữ liệu từ bên ngoài
        print("Nhận:", data)

c = coroutine()
# Gọi hàm generator, nhưng chưa chạy bất kỳ dòng nào bên trong nó. Lúc này `c` là một **coroutine object** (hay generator object).

next(c)  # Khởi tạo
c.send("A")  # In "Nhận: A"
c.send(123)  # In "Nhận: 123"

'''
1. c = coroutine()         → tạo generator object, chưa thực thi
2. next(c)                 → chạy tới yield, "pause" để chờ dữ liệu
3. c.send("A")             → gửi "A", yield trả về "A" → in "Nhận: A"
4. c.send(123)             → gửi 123, yield trả về 123 → in "Nhận: 123"
'''
```

## Nhược điểm khi sử dụng generator

- Không thể “đi lùi” hoặc lặp lại
- Khó debug vì dữ liệu biến mất sau mỗi `next()`

---

# Coroutine

`Coroutine` trong Python là một khái niệm mở rộng từ generator

**Coroutine** là một loại **generator nâng cao**, cho phép:
- Không chỉ **trả về giá trị** (như generator),
- Mà còn **nhận dữ liệu từ bên ngoài** thông qua `.send()`.

Khác với hàm thông thường (chạy từ đầu đến cuối rồi trả về), `coroutine` là một hàm có thể tạm dừng và tiếp tục thực thi, cho phép trao đổi dữ liệu hai chiều (nhận và trả dữ liệu).  `coroutine` có thể:
- **Tạm dừng (pause)** thực thi tại các điểm xác định
- **Duy trì trạng thái** (giá trị biến, vị trí thực thi)
- **Tiếp tục (resume)** từ điểm dừng
- **Nhận dữ liệu đầu vào** trong khi đang thực thi

Trong Python, coroutine được xây dựng dựa trên generator bằng cách sử dụng `yield` để nhận giá trị.

***Cách hoạt động:***

- Khi gọi một coroutine, nó không thực thi ngay mà trả về một generator object.

- Bạn phải "khởi tạo" coroutine bằng cách gọi `next()` hoặc `send(None)` lần đầu để nó chạy đến điểm `yield` đầu tiên.

- Sau đó, sử dụng phương thức `send(value)` để truyền dữ liệu vào coroutine. Giá trị này sẽ được gán cho biểu thức `yield` và coroutine tiếp tục chạy từ điểm đó cho đến khi gặp `yield` tiếp theo (hoặc kết thúc).

- Khi coroutine kết thúc, nó sẽ ném ra ngoại lệ `StopIteration`.

Ví dụ
```python
def simple_coroutine():
    print("Coroutine bắt đầu")
    # yield vừa trả giá trị vừa nhận đầu vào
    data = yield "Sẵn sàng"
    print(f"Nhận dữ liệu: {data}")
    yield "Hoàn thành"

# Khởi tạo coroutine
coro = simple_coroutine()

# Khởi động coroutine (chạy đến yield đầu tiên)
status = next(coro)
print(status)  # Output: "Sẵn sàng"

# Truyền dữ liệu vào coroutine
result = coro.send("Hello!")
print(result)  # Output: "Hoàn thành"
```


## Đặc điểm quan trọng của coroutine

- **Khởi tạo bằng `next()` hoặc `send(None)`** trước khi truyền dữ liệu
    
- **Duy trì trạng thái** giữa các lần gọi
    
- **Không đồng bộ**: Kiểm soát thủ công khi nào tạm dừng/tiếp tục
    
- **Hợp tác**: Coroutine tự nguyện nhường quyền kiểm soát ➡️ một coroutine chỉ **tạm dừng và cho phép coroutine khác chạy khi chính nó gọi `yield` hoặc `await`.**
	- Không giống như **thread**, nơi hệ điều hành tự ý “giật” quyền CPU giữa các thread, coroutine hoạt động theo **nguyên tắc tự giác**.

## Coroutine hiện đại với `async/await`

Từ Python 3.5+, coroutine được chuẩn hóa với cú pháp mới:
```python
import asyncio

async def main():
    print("Bắt đầu")
    await asyncio.sleep(1)
    print("Sau 1 giây")

# Chạy coroutine
asyncio.run(main())
```

**Lợi ích**:
- Cú pháp rõ ràng hơn `yield`
- Tích hợp với thư viện asyncio
- Hỗ trợ `async for`, `async with`

## Tại sao chọn Coroutine thay vì đa luồng?

 
| Tiêu chí       | Coroutine               | Đa luồng (Threading)  |
| -------------- | ----------------------- | --------------------- |
| Chi phí        | Rất thấp (KB)           | Cao (MB/thread)       |
| Chuyển context | không cần OS            | cần OS scheduler      |
| Đồng bộ hóa    | không cần lock          | cần lock/mutex        |
| Share data     | An toàn (không chia sẻ) | Rủi ro race condition |
| Độ phức tạp    | quản lý thủ công        | OS tự quản lý         |
