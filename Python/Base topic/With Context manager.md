
`with` là cú pháp được thiết kế để quản lý **tài nguyên** (resource) một cách an toàn (ví dụ: file, kết nối mạng, process…). Nó đòi hỏi đối tượng phải tuân thủ **giao diện context manager**, tức là class phải định nghĩa:

- `__enter__(self)`
    
- `__exit__(self, exc_type, exc_value, traceback)`
    

Khi class có hai hàm này, Python hiểu rằng nó có thể dùng trong `with`.

Khi gọi 
```python
with SomeRunner(...) as runner:
    # làm gì đó với runner
```

Python sẽ thực hiện
```python
runner = SomeRunner(...)           # Gọi __init__
runner.__enter__()                 # Gọi __enter__, giá trị trả về gán vào biến runner
# --- bên trong khối with ---
runner.__exit__(...)               # Khi khối with kết thúc, gọi __exit__
```

Ví dụ
```python
class MyContext:
    def __enter__(self):
        print("👉 Entering context")
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        print("👋 Exiting context")

with MyContext() as ctx:
    print("💡 Doing work")
```

Kết quả
```bash
👉 Entering context
💡 Doing work
👋 Exiting context
```
