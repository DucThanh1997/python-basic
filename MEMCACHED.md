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

Here again, there is no magical recipe; it depends on the type of application you are building. 
However, there are several outlying cases that should be handled—which we haven’t yet covered in the above example.

A caching server cannot grow infinitely—memory is a finite resource. 
Therefore, keys will be flushed out by the caching server as soon as it needs more space to store other things.

Some keys might also be expired because they reached their expiration time (also sometimes called the “time-to-live” or TTL.) 
In those cases the data is lost, and the canonical data source must be queried again.


Ở đây một lần nữa, không có công thức kỳ diệu nào cả; nó phụ thuộc vào loại ứng dụng bạn đang xây dựng. 
Tuy nhiên, có một số trường hợp ngoại lệ nên được xử lý mà chúng tôi chưa đề cập đến trong ví dụ trên.

Một máy chủ bộ nhớ đệm không thể phát triển vô hạn Bộ nhớ - 1 tài nguyên hữu hạn. 
Do đó, các key sẽ được xóa bởi caching server ngay khi nó cần thêm dung lượng để lưu trữ những thứ khác.

Một số khóa cũng có thể bị hết hạn vì chúng đã hết thời gian lưu trữ 
(đôi khi còn được gọi là thời gian sống hay thời gian sống của người Hồi giáo). 
Trong những trường hợp đó, dữ liệu bị mất và nguồn dữ liệu chính tắc phải được truy vấn lại.



















