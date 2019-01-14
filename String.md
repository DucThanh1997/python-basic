# Tạo 1 String
Có 4 cách
```
str1 = " Thành "

str2 = ' Thành '

str3 = """
    Thành đẹp trai
    Thành học giỏi
    """
a = 1
str(a)
nhưng nếu bạn làm thế này sẽ không được int("AVC") bạn chỉ có thể đưa nó về nhị phân bằng hàm bin()
```
Cái 3 dấu ngoặc kép cho phép viết nhiều dòng

Không như list, String không thay đổi các phần tử trong nó được nhưng có thể in ra các phần tử trong nó
`print(a[0])`

# Ghép String

Cách ghép string
```
a = "1"
b = "2"
print("a + b")
>>> 12
```
string chỉ support + còn không support trừ nhân chia
**Mỗi 1 biến từ string,int,hay kể cả list đều sẽ có id**

Xem id bằng cách
```
>>> my_string = "abc"
>>> id(my_string)
19397208
>>> my_string = "def"
>>> id(my_string)
25558288
>>> my_string = my_string + "ghi"
>>> id(my_string)
31345312
```
Mỗi lần cta thay đổi cái gì trong biến thì id nó sẽ thay đổi

# Phương thức của String
- `str.upper`: để in hoa
- `str.lower`: để bỏ in hoa
- `str.capitalize`: để in hoa chữ đầu tiên

Thực ra nó có nhiều lắm để xem hết thì dùng lệnh `print(dir(my_str))` để xem 

Còn muốn xem cách dùng thì dùng lệnh `help(my_string.capitalize)`

# String slicing
```
>>>my_string = "I like Python"
>>>len(my_string)
13
>>> my_string[:1]
'I'
>>> my_string[0:12]
'I like Pytho'
>>> my_string[0:13]
'I like Python'
>>> my_string[0:14]
'I like Python!'
>>> my_string[0:-5]
'I like Py'
>>> my_string[:]
'I like Python!'
>>> my_string[2:]
'like Python!'
```

# String formating

Kiểu như là bạn sẽ phải đưa các biến vào string thì sau đây là cách làm
```
Đưa vào biến string

my_string = "Tôi tên là %s, tôi %i tuổi, tôi cao %f cm" % ("Thành", 22, 173.3)

%s: đưa vào biến string
%i: đưa vào biến interger
%f: đưa vào biến flow

Đây là những kiểu biến đơn giản còn list hay dictionary thì sao
```
cách in dictionary:
print("%(value)s %(so1)i %(so2)f !" % {"value":"SPAM", "so1":3, "so2":3.0})

cách in list 1 tập các biến giống list và có thể arrange được:
```
>>> "Python is as simple as {0}, {1}, {2}".format("a", 1, "c")
'Python is as simple as a, 1, c'
>>> "Python is as simple as {1}, {0}, {2}".format("a", "b", "c")
'Python is as simple as b, a, c'
```






























