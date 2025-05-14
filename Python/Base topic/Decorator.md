***Decorator*** là một trong những design pattern phổ biến trong lập trình.
Decorator là một pattern trong đó ***một function (hoặc một class) sẽ nhận input là một hàm, và trả ra output là một hàm***. 
- Hàm được trả ra sẽ có hành vi khác so với hàm input, có thể tính năng của hàm được extends,....

***Ví dụ***:
Ta có 2 hàm sau:
```python
def ham_cong(*args): # Hàm dùng tính tổng
	return sum(args)

def luy_thua(*args): # Hàm tính lũy thừa
	return pow(args)
```
Nếu muốn thêm log thời gian thực thi , ta cần cập nhật code cho cả 2 function như sau:

```python
def add(*args):
	start_at = time.time() 
	s = sum(*args) 
	print("Take", time.time()-s, "second") 
	return s

def luy_thua(*args):
	start_at = time.time() 
	s = pow(*args) 
	print("Take", time.time()-s, "second") 
	return s
```

Tuy nhiên khi có ***rất nhiều function*** khác muốn sử dụng cùng tính năng như vậy => cần code lại hết các func à 😒😒😒 ➡️ cần sử dụng ***decorator***

**Định nghĩa một decorator function để thực hiện tính năng log**

```python
def log(func): # Định nghĩa hàm decorator
	def func_with_log(*args, **kwargs):
		start_at = time.time()
		ret = func(*args, **kwargs)
		print("Take", time.time()-start_at, "second")
		return ret
	return func_with_log

def ham_cong(*agrs): # Hàm dùng tính tổng
	return sum(args)

def luy_thua(*args): # Hàm tính lũy thừa
	return pow(args)

# Sử dụng hàm decorator
add_with_log = log(ham_cong)
print(add_with_log(5, 10))
```

***Cách sử dụng ngắn gọn hơn của decorator***

```python
def log(func): # Định nghĩa hàm decorator
	def func_with_log(*args, **kwargs):
		start_at = time.time()
		ret = func(*args, **kwargs)
		print("Take", time.time()-start_at, "second")
		return ret
	return func_with_log

@log
def ham_cong(*agrs): # Hàm dùng tính tổng
	return sum(args)

# Sử dụng hàm decorator
print(ham_cong(5, 10))

'''
Nếu dùng decorator như dưới đây thì sẽ bị báo lỗi
do khi gọi @log() thì đã gọi log trước khi dùng nó làm decorator
=> my_log sẽ không nhận func ham_cong làm đối số truyền vào nữa
'''
@log()
def ham_cong(*args): # Hàm dùng tính tổng
	return sum(args)
```

## Decorator stack
Khi một function có nhiều decorator lồng nhau, thì các decorator sẽ được thực thi theo thứ tự ***stack***

```python
def decorator1(func):
	def new_func(*args, **kwargs):
		print('Decorator 1')
		return func(*args, **kwargs)
	return new_func

def decorator2(func):
	def new_func(*args, **kwargs):
		print('Decorator 2')
		return func(*args, **kwargs)
	return new_func

@decorator1
@decorator2
def func(msg):
	print('Original func:', msg)

func("Hello world")
```
Kết quả in ra sẽ là:
```bash
Decorator 1
Decorator 2
Original func: Hello world
```

## Decorator có tham số
Khi muốn truyền tham số cho decorator, ta sử dụng ***một decorator cha bọc bên ngoài*** decorator cần truyền tham số

```python
def log(show_input=False):
	def config(func):
		def func_with_log(*args, **kwargs):
			start_at = time.time()
			ret = func(*args, **kwargs)
			print(func.__name__, "takes", time.time()-start_at, "second")
			if show_input:
				print("Input:", args, kwargs)
			return ret
		return func_with_log
	return config

# Sử dụng
@log(show_input=True)
def need_log_input():
	pass

@log(show_input=False)
def dont_need_log_input():
	pass
```

***Giải thích***: hàm `log` đầu tiên sẽ làm hàm nhận vào parameter của decorator (decorator cha), sau đó nó trả về hàm `config` (hàm decorator con), hàm `config` này nhận vào parameter là function.

```python
@log
def ham_cong(*args):
	return sum(args)
```

Nếu định nghĩa decorator có đối số truyền vào, mà khi gọi ko truyền tham số => thì hàm `ham_cong` khi được gọi ra sẽ là instance của `func_with_log` chứ ko phải là instance của `function` ta truyền vào.


## Decorator class
Trong python, class cũng có thể được sử dụng để build decorator
***Ví dụ*** 

```python
class my_log(object): # Định nghĩa decorator class
	def __init__(self, function):
		self.function = function

	def __call__(self, *args, **kwargs):
		log_string = self.function.__name__ + "was called"
		print(log_string)
		return self.function(*args, **kwargs)

@my_log
def ham_cong(*args): # Hàm dùng tính tổng
	return sum(args)


'''
Nếu dùng decorator như dưới đây thì sẽ bị báo lỗi
do khi gọi @my_log() thì đã gọi my_log trước khi dùng nó làm decorator
=> my_log sẽ không nhận func ham_cong làm đối số truyền vào nữa
'''
@my_log()
def ham_cong(*args): # Hàm dùng tính tổng
	return sum(args)
```