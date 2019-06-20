# Python + Memcached: Efficient Caching in Distributed Applications

Khi viết 1 ứng dụng Python, caching rất quan trọng. Vì khi sử dụng caching bạn sẽ tránh được việc phải tính toán lại những dữ liệu đã
tính toán rồi và giảm tại lượng truy cập không cần thiết đến server

Python đã tích hợp sẵn khả năng cho việc caching, đơn giản nhất là dictionary đến phức tạp nhất là ` functools.lru_cache`. Ngoài ra,
các request ở lần sau còn có thể truy cập các dữ liệu đã được lưu trữ ở request trước và cho phép chúng ta giới hạn dung lượng lưu trữ nhờ
thuật toán  `Least-Recently Used algorithm`

Tuy nhiên, các cấu trúc dữ liệu đó được định nghĩa theo cục bộ cho quy trình Python của bạn. Khi một vài bản sao ứng dụng của bạn chạy 
trên một nền tảng lớn, sử dụng cấu trúc dữ liệu trong bộ nhớ sẽ không cho phép chia sẻ nội dung được lưu trong bộ nhớ cache. 
Đây có thể là một vấn đề cho các ứng dụng phân tán và quy mô lớn.

![image](https://user-images.githubusercontent.com/45547213/59737272-232e7c00-9287-11e9-88d9-eb4609060ead.png)

Khi hệ thống phân tán qua 1 mạng, nó cũng cần 1 cache để phân tán dữ liệu với nó. Như bạn sẽ thấy trong hướng dẫn này, 
memcached là một tùy chọn tuyệt vời khác cho bộ nhớ đệm phân tán. Sau phần giới thiệu nhanh về cách sử dụng memcached cơ bản, 
bạn sẽ tìm hiểu về các mẫu nâng cao như bộ nhớ cache và cài đặt bộ nhớ cache và sử dụng bộ đệm dự phòng để tránh các vấn đề về 
hiệu năng bộ đệm lạnh.

## Cài đặt
link đây tự đọc rồi cài
`https://commaster.net/content/installing-memcached-windows`

## Lưu trữ và truy xuất các giá trị cache bằng python
Nếu bạn chưa bao giờ sử dụng memcached, nó khá dễ hiểu. Về cơ bản nó cung cấp một từ điển mạng có sẵn khổng lồ.
Từ điển này có một vài thuộc tính khác với từ điển Python, chủ yếu là:
+ Key và values phải là byte
+ Key và values tự động xóa sau 1 khoảng thời gian

Do đó, hai thao tác cơ bản để tương tác với memcached được `set` và `get`. Như bạn có thể đoán,
2 thao tác này đã sử dụng để gán giá trị cho khóa hoặc để lấy giá trị từ khóa tương ứng.

Chúng ta sẽ dùng pymemcache để thực hiện 2 thao tác này, nhớ phải chạy memcache đã nhóe

```
>>> from pymemcache.client import base  # cái này để import vào này

# Don't forget to run `memcached' before running this next line:
>>> client = base.Client(('localhost', 11211))  # cái này để khởi tạo này

# Sau khi khởi tạo xong chúng ta có thể access cache:
>>> client.set('some_key', 'some value')  # set key và value cho cache, key trước value sau

# Bây giờ truy xuất value qua key bằng phương thức get
>>> client.get('some_key')
'some value'
```

memcached network protocol thực sự rất đơn giản và việc triển khai cực kỳ nhanh, điều này giúp lưu trữ dữ liệu có tốc độ truy xuất chậm
từ nguồn dữ liệu chính tắc hoặc dữ liệu tính toán lại:

Mặc dù đủ đơn giản, ví dụ trên cho phép lưu trữ keys / value trên mạng và truy cập chúng thông qua nhiều bản sao, phân phối, 
đang chạy của ứng dụng của bạn. Điều này đơn giản, nhưng hiệu quả. Và nó là bước tuyệt vời đầu tiên để tối ưu hóa ứng dụng của bạn.

## Automatically Expiring Cached Data

Khi lưu trữ dữ liệu vào memcached, bạn có thể đặt thời gian hết hạn là một khoảng thời gian tính số giây tối đa cho memcached để giữ keys 
và values . Sau khoảng thời gian đó, memcached sẽ tự động xóa key khỏi bộ đệm.

Bạn nên đặt thời gian bộ đệm này là gì? Không có con số chuẩn mực nào cho khoảng thời gian này và 
nó sẽ hoàn toàn phụ thuộc vào loại dữ liệu và ứng dụng mà bạn đang làm việc. Nó có thể là một vài giây, hoặc nó có thể là một vài giờ.

Vô hiệu hóa bộ đệm, bạn cần xóa cache vì nó không đồng bộ với dữ liệu hiện tại, 
cũng là điều mà ứng dụng của bạn sẽ phải xử lý. Đặc biệt là nếu trình bày dữ liệu đã quá cũ


Ở đây một lần nữa, không có công thức kỳ diệu nào cả; nó phụ thuộc vào loại ứng dụng bạn đang xây dựng. 
Tuy nhiên, có một số trường hợp ngoại lệ nên được xử lý mà chúng tôi chưa đề cập đến trong ví dụ trên.

Một máy chủ bộ nhớ đệm không thể phát triển vô hạn Bộ nhớ - 1 tài nguyên hữu hạn. 
Do đó, các key sẽ được xóa bởi caching server ngay khi nó cần thêm dung lượng để lưu trữ những thứ khác.

Một số khóa cũng có thể bị hết hạn vì chúng đã hết thời gian lưu trữ  (đôi khi còn được gọi là thời gian sống). 
Trong những trường hợp đó, dữ liệu bị mất và nguồn dữ liệu chính tắc phải được truy vấn lại.

Nghe hơi phức tạp nên làm 1 ví dụ nhá Thành =))))
```
from pymemcache.client import base


def do_some_query():
    # Replace with actual querying code to a database,
    # a remote REST API, etc.
    return 42


# Don't forget to run `memcached' before running this code
client = base.Client(('localhost', 11211))
result = client.get('some_key')

if result is None:
    # The cache is empty, need to get the value
    # from the canonical source:
    result = do_some_query()

    # Cache the result for next time:
    client.set('some_key', result)

# Whether we needed to update the cache or not,
# at this point you can work with the data
# stored in the `result` variable:
print(result)
```

## Warming Up a Cold Cache

Có vài trường hợp vì lí do nào đó bạn biết là memcache sẽ crash vì lí do nào đó bạn không thể biết nhưng cũng có trường hợp bạn biết
là nó sẽ crash chính vì vậy bạn cần thực hiện 1 cái tôi gọi là di tản và pymemcache có hỗ trợ cái này
```
from pymemcache.client import base
from pymemcache import fallback


def do_some_query():
    code chuẩn
    return 42


# nhớ set `ignore_exc=True` để có thể tắt old cache trước khi xóa đi dung lượng của chương trình 
old_cache = base.Client(('localhost', 11211), ignore_exc=True)
new_cache = base.Client(('localhost', 11212))

client = fallback.FallbackClient((new_cache, old_cache))

result = client.get('some_key')
```

The FallbackClient queries the old cache passed to its constructor, respecting the order. In this case, the new cache server will always be queried first, and in case of a cache miss, the old one will be queried—avoiding a possible return-trip to the primary source of data.

If any key is set, it will only be set to the new cache. After some time, the old cache can be decommissioned and the FallbackClient can be replaced directed with the new_cache client.

FallbackClient truy vấn old cache rồi chuyển đến hàm tạo của nó, tuân theo thứ tự. Trong trường hợp này, cache server mới sẽ luôn được truy vấn trước và trong trường hợp bộ nhớ cache bị lỗi, máy chủ cũ sẽ được truy vấn lại tránh việc quay trở lại nguồn dữ liệu chính.

Nếu bất kỳ khóa nào được đặt, nó sẽ chỉ được đặt thành bộ đệm mới. Sau một thời gian, bộ đệm cũ có thể ngừng hoạt động và FallbackClient có thể được thay thế trực tiếp bằng new_cache client.

## Check and set
Khi giao tiếp với bộ đệm từ xa, vấn đề tương tranh thông thường sẽ quay trở lại: có thể có một số khách hàng cố gắng truy cập cùng một khóa cùng một lúc. memcached cung cấp một thao tác kiểm tra và thiết lập, rút ngắn thành CAS, giúp giải quyết vấn đề này.

Ví dụ đơn giản nhất là một ứng dụng muốn đếm số lượng người dùng mà nó có. Mỗi khi khách truy cập kết nối, một bộ đếm được tăng thêm 1. Sử dụng memcached, một cách thực hiện đơn giản sẽ là:

```
def on_visit(client):
    while True:
        result, cas = client.gets('visitors')
        if result is None:
            result = 1
        else:
            result += 1
        if client.cas('visitors', result, cas):
            break
 ```

Phương thức `gets` trả về 1 giá trị, giống như phương thức get, nhưng nó cũng trả về giá trị CAS. Những gì trong giá trị này không liên quan, nhưng nó được sử dụng cho phương thức cas tiếp theo. Phương pháp này tương đương với phương thức set, ngoại trừ việc nó không thành công nếu giá trị gets đã thay đổi kể từ khi từ khi cas được đặt. Trong trường hợp thành công, vòng lặp break. Nếu không, hoạt động được khởi động lại từ đầu.

Trong trường hợp, hai nơi cố gắng cập nhật bộ đếm cùng một lúc, chỉ có một trường hợp thành công để chuyển bộ đếm từ 42 sang 43. Trường hợp thứ hai nhận được giá trị Sai được trả về bởi lệnh gọi client.cas và phải thử lại vòng lặp. Nó sẽ lấy 43 như giá trị lần này, sẽ tăng nó lên 44 và phương thức cas của nó sẽ thành công



# REDIS
## Định nghĩa
Redis là một phần mềm mã nguồn mở được dùng để lưu trữ một cách tạm thời trên bộ nhớ (hay còn gọi là cache data) và giúp truy xuất dữ liệu một cách nhanh chóng. Do tốc độ truy xuất dữ liệu vượt trội so với các cơ sở dữ liệu thông thường như MySQL nên redis được sử dụng rất nhiều trong kỹ thuật Caching.

## Cài đặt
`https://dzone.com/articles/running-redis-on-windows-81-and-prior` 
Đây nha

## Dùng
```
import redis

# bước 1: import redis vào
# bước 2: thiết lập các thông số để connect đến server
redis_host = "localhost"
redis_port = 6379
redis_password = ""


def hello_redis():
    """Example Hello Redis Program"""

    # step 3: tạo Redis Connection object
    r = redis.Redis(host='localhost', port=6379, db=0)

    r.set('count', 1)
    # r.lpush('hispanic', 'thành')
    print(r.llen('hispanic'))
    print(r.get('hispanic'))

if __name__ == '__main__':
    hello_redis()
```

Các kiểu dữ liệu mà redis support
- Binary-safe strings.
- Lists
- Sets
- Sorted sets
- Hashes
- Bit arrays (or simply bitmaps)
- HyperLogLogs

















