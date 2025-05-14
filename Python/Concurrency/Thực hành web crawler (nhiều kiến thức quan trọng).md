```table-of-contents
```
# Code ban đầu
```python
from urllib.request import Request, urlopen
from urllib.parse import urlparse, urljoin
from urllib.error import URLError
import ssl
from bs4 import BeautifulSoup
import threading
import queue

class Crawler:
    base_url = ''
    myssl = ssl.create_default_context()
    myssl.check_hostname=False
    myssl.verify_mode=ssl.CERT_NONE
    errorLinks = set()
    crawled_links = set()

    def __init__(self, base_url):
        Crawler.base_url = base_url

    @staticmethod
    def crawl(thread_name, url, links_to_crawl):
        try:
            link = urljoin(Crawler.base_url, url)
            if (urlparse(link).netloc == 'tutorialedge.net') and (link not in Crawler.crawled_links):
                request = Request(link, headers={'User-Agent': 'Mozilla/5.0'})
                response = urlopen(request, context=Crawler.myssl)
                Crawler.crawled_links.add(link)
                print("Url {} Crawled with Status: {} : {} Crawled In Total".format(response.geturl(), response.getcode(), len(Crawler.crawled_links)))
                soup = BeautifulSoup(response.read(), "html.parser")
                Crawler.enqueueLinks(soup.find_all('a'), links_to_crawl)
        except URLError as e:
            print("URL {} threw this error when trying to parse: {}".format(link, e.reason))
            Crawler.errorLinks.add(link)

    @staticmethod
    def enqueueLinks(links, links_to_crawl):
        for link in links:
            if (urljoin(Crawler.base_url, link.get('href')) not in Crawler.crawled_links):
                if (urljoin(Crawler.base_url, link.get('href')) not in links_to_crawl):
                    links_to_crawl.put(link.get('href'))


class CheckableQueue(queue.Queue):
    def __contains__(self, item):
        with self.mutex:
            return item in self.queue
    def __len__(self):
        return len(self.queue)


THREAD_COUNT = 100
linksToCrawl = CheckableQueue()

def createCrawlers():
    for i in range(THREAD_COUNT):
        t = threading.Thread(target=run)
        t.daemon = True
        t.start()

def run():
    while True:
        url = linksToCrawl.get()
        try:
            if url is None:
                break
            Crawler.crawl(threading.current_thread(), url, linksToCrawl)
        except:
            print("Exception")
        linksToCrawl.task_done()


def main():
    url = input("Website > ")
    linksToCrawl.put(url)
    createCrawlers()
    linksToCrawl.join()
    print("Total Links Crawled: {}".format(len(Crawler.crawled_links)))
    print("Total Errors: {}".format(len(Crawler.errorLinks)))

if __name__ == '__main__':
    main()

```

## Giải thích câu hỏi một chút

### 1. Về câu hỏi **"queue không hỗ trợ `in` mà sao lại `item in self.queue`"**?
- Đúng rồi, bản thân `queue.Queue` **không hỗ trợ trực tiếp** `item in myqueue` trên đối tượng queue bên ngoài.
```python
new_queue = queue.Queue()
# Không hỗ trợ
print("a" in new_queue)
```
- Nhưng **bên trong** `queue.Queue` có một **biến nội bộ** tên là `.queue`:
    - `.queue` thường là một `collections.deque` hoặc `list`.
    - Mà `list` và `deque` thì **hỗ trợ** toán tử `in`.

✅ Vì vậy, **`item in self.queue`** thực ra là đang check bên trong cái **list**/**deque** lưu trữ dữ liệu thực sự trong queue.

### 2. Thế còn `with self.mutex:` làm gì?

- `queue.Queue` thiết kế cho **đa luồng** (multi-thread).
- Vì vậy **mọi thao tác truy cập** `.queue` **phải có khóa (`mutex`)** để **tránh race condition** (2 thread cùng đọc/ghi một lúc → hỏng dữ liệu).
- `self.mutex` là một `threading.Lock()`, giúp **khóa** queue tạm thời để thao tác an toàn.

💬 Tức là:
```python
with self.mutex:
	return item in self.queue`
```

Nghĩa là:

> Khóa queue lại, kiểm tra an toàn xem item có trong queue nội bộ không, rồi mở khóa.

Nếu **không `with self.mutex`**, 2 thread cùng lúc đọc queue thì dữ liệu có thể **không chính xác**, thậm chí bị lỗi crash.

---
# Cải tiến lần 1

Các cải tiến:
* Chỉ khởi tạo một thread pool lần đầu, và có nhiều thread trong pool
* Lưu kết quả vào trong CSV

```python
from urllib.request import Request, urlopen
from urllib.parse import urlparse, urljoin
from urllib.error import URLError
from concurrent.futures import ThreadPoolExecutor, as_completed
import ssl
from bs4 import BeautifulSoup
import threading
import csv
import queue

class Crawler:
    base_url = ''
    myssl = ssl.create_default_context()
    myssl.check_hostname=False
    myssl.verify_mode=ssl.CERT_NONE
    errorLinks = set()
    crawled_links = set()

    def __init__(self, base_url):
        Crawler.base_url = base_url

    @staticmethod
    def crawl(thread_name, url, links_to_crawl):
        try:
            print(f"Run with thread {thread_name}")
            link = urljoin(Crawler.base_url, url)
            if (urlparse(link).netloc == 'tutorialedge.net') and (link not in Crawler.crawled_links):
                request = Request(link, headers={'User-Agent': 'Mozilla/5.0'})
                response = urlopen(request, context=Crawler.myssl)
                Crawler.crawled_links.add(link)
                print("Url {} Crawled with Status: {} : {} Crawled In Total".format(response.geturl(), response.getcode(), len(Crawler.crawled_links)))
                soup = BeautifulSoup(response.read(), "html.parser")
                Crawler.enqueueLinks(soup.find_all('a'), links_to_crawl)
        except URLError as e:
            print("URL {} threw this error when trying to parse: {}".format(link, e.reason))
            Crawler.errorLinks.add(link)

    @staticmethod
    def enqueueLinks(links, links_to_crawl):
        for link in links:
            if (urljoin(Crawler.base_url, link.get('href')) not in Crawler.crawled_links):
                if (urljoin(Crawler.base_url, link.get('href')) not in links_to_crawl):
                    links_to_crawl.put(link.get('href'))


class CheckableQueue(queue.Queue):
    def __contains__(self, item):
        with self.mutex:
            return item in self.queue
    def __len__(self):
        return len(self.queue)


THREAD_COUNT = 100
linksToCrawl = CheckableQueue()

def run():
    while True:
        try:
            url = linksToCrawl.get()
            Crawler.crawl(threading.current_thread(), url, linksToCrawl)
        except:
            print("Exception")
        linksToCrawl.task_done()
        return [url]

def append_to_csv(result):
    with open('D:\\omzcloud\\test\\results.csv', 'a') as writer:
        result_writer = csv.writer(writer, delimiter=' ', quotechar='!', quoting=csv.QUOTE_MINIMAL)
        result_writer.writerow(result)


def main():
    url = input("Website > ")
    linksToCrawl.put(url)
    while not linksToCrawl.empty():
        with ThreadPoolExecutor(max_workers=THREAD_COUNT) as executor:
            future = executor.submit(run)
            try:
                if future.result() is not None:
                    append_to_csv(future.result())
            except:
                print(future.exception())

print("Total Links Crawled: {}".format(len(Crawler.crawled_links)))
print("Total Errors: {}".format(len(Crawler.errorLinks)))

if __name__ == '__main__':
    main()
```


---
# Cải tiến lần 2

Trong code trên, do chỉ gọi `.submit()` đúng 1 lần ↔ task đó chỉ đc assign cho 1 thread từ đầu tới cuối
⇒ Không tận dụng được tối đa sức mạnh của các thread khác trong pool

➡️ Cải tiến, do dùng queue đã đảm bảo không bị race-condition, do đó có thể push nhiều task vào trong pool.

```python
THREAD_COUNT = 100
linksToCrawl = CheckableQueue()

def append_to_csv(result):
    with open('D:\\omzcloud\\test\\results.csv', 'a') as writer:
        result_writer = csv.writer(writer, delimiter=' ', quotechar='!', quoting=csv.QUOTE_MINIMAL)
        result_writer.writerow(result)

def run():
    while True:
        try:
            url = linksToCrawl.get(timeout=3)
            if url is None:  # Sentinel để kết thúc
	            linksToCrawl.task_done()
	            break
            Crawler.crawl(threading.current_thread(), url, linksToCrawl)
            append_to_csv([url])
        except queue.Empty:
            break
        except:
            print("Exception")
        finally:
            linksToCrawl.task_done()

        
def main():
    # url = input("Website > ")
    url = "https://tutorialedge.net"
    linksToCrawl.put(url)
    futures = []
    with ThreadPoolExecutor(max_workers=THREAD_COUNT) as executor:
        for i in range(THREAD_COUNT):
            future = executor.submit(run)
            futures.append(future)
        linksToCrawl.join()
	    # Để chắc chắn Gửi tín hiệu kết thúc cho từng thread
        for _ in range(THREAD_COUNT):
            linksToCrawl.put(None)  # Sentinel
```

***Chi tiết các cải tiến***
* **Blocking Queue.get()**: Phương thức `linksToCrawl.get()` mặc định **blocking**, tức là nó sẽ chờ đến khi có phần tử trong hàng đợi. Khi hàng đợi trống, các luồng bị treo tại đây và không bao giờ thoát, dẫn đến chương trình bị đứng
* **Xử lý ngoại lệ queue.Empty không hiệu quả**: Khối `except queue.Empty` không bao giờ được kích hoạt vì `get()` không ném ra ngoại lệ này trừ khi được gọi với `block=False` hoặc có ***timeout***.
### Cách khắc phục triệt để:
1. **Sử dụng get() với timeout**:
```python
try:
    url = linksToCrawl.get(timeout=1)  # Chờ 1 giây trước khi kiểm tra lại
except queue.Empty:
    if linksToCrawl.qsize() == 0:
        break
```
1. **Kết hợp Queue.join() và sentinel values**:
    - Thêm giá trị đặc biệt (ví dụ: `None`) vào hàng đợi để báo hiệu luồng thoát.
    - Gọi `linksToCrawl.join()` trong `main()` để đợi mọi task hoàn thành.