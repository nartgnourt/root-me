<!-- markdownlint-disable MD033 MD041 -->
<p align="center">
<a href="https://www.root-me.org/"><img alt="https://www.root-me.org" src="images/root-me.svg" width="150" height="150"></a>
</p>
<!-- markdownlint-enable MD033 -->

# Root Me

Chào mọi người!

Tại đây, mình sẽ viết lại cách mà mình đã thực hiện để có thể giải được một số thử thách ở trên trang web [Root Me](https://www.root-me.org/).

Cảm ơn vì đã ghé thăm! 🌱

## HTML - Source code

> Don't search too far

Mở đầu với một thử thách đơn giản, chúng ta sẽ nhấn tổ hợp phím `Command + U` trên Mac (hoặc `Ctrl + U` trên Windows) để có thể xem HTML source code của trang web. Từ đó, chúng ta sẽ lấy được password trong phần comment.

![image](images/html-source-code/image-1.png)

## HTTP - IP restriction bypass

> Only local users will be able to access the page
>
> Dear colleagues,
>
> We’re now managing connections to the intranet using private IP addresses, so it’s no longer necessary to login with a username / password when you are already connected to the internal company network.
>
> Regards,
>
> The network admin

Khi bắt đầu thử thách, chúng ta thấy trang web hiện lên thông báo rằng địa chỉ IP của chúng ta không thuộc mạng LAN và yêu cầu xác thực.

![image](images/http-ip-restriction-bypass/image-1.png)

Tuy nhiên, sẽ ra sao nếu chúng ta giả mạo địa chỉ IP nội bộ bằng cách thêm vào một request header như `X-Forwarded-For: 192.168.0.1`.
Với trình duyệt Firefox, chúng ta có thể mở Web Developer Tools và sửa đổi request ở tab Network.

Gửi request, chúng ta thấy password xuất hiện.

![image](images/http-ip-restriction-bypass/image-2.png)

## HTTP - Open redirect

> Internet is so big
>
> Find a way to make a redirection to a domain other than those showed on the web page.

Một trang web giản đơn với 3 nút Facebook, Twitter và Slack.

![image](images/http-open-redirect/image-1.png)

Nếu chúng ta nhấn chọn 1 trong 3 nút này sẽ được điều hướng tới trang web tương ứng. Bên dưới là request khi chúng ta nhấn vào nút Facebook.

![image](images/http-open-redirect/image-2.png)

Có thể thấy phần query string được thêm vào URL, nó có 2 tham số là `url` và `h`. Thực hiện điều hướng sử dụng `document.location`.

Điểm đáng chú ý là tham số `h` chứa giá trị MD5 hash của giá trị trong `url`. Chúng ta dễ dàng kiểm tra được nhờ sử dụng [Hash text - IT tools](https://it-tools.tech/hash-text).

![image](images/http-open-redirect/image-3.png)

Vậy, thử đổi `https://facebook.com` thành một giá trị bất kỳ, ví dụ như số `1`. Ngay lập tức, chúng ta nhận được một thông báo lỗi "Incorrect hash!".

![image](images/http-open-redirect/image-4.png)

Điều đó chứng tỏ rằng ở phía server đang thực hiện tính toán MD5 hash giá trị của tham số `url` và so sánh nó với hash nằm ở tham số `h`. Nếu đúng mới trả về thẻ `<script>`.

Vậy nên chúng ta cần tính giá trị MD5 hash của `1` để truyền vào tham số `h`.

![image](images/http-open-redirect/image-5.png)

Có lẽ server kiểm tra nếu giá trị của `url` không phải là 1 trong 3 URLs tới Facebook, Twitter, Slack và giá trị hash hợp lệ thì trả về flag cho chúng ta.

![image](images/http-open-redirect/image-6.png)

## HTTP - User-agent

> Admin is really dumb...

Trang web thông báo user-agent của chúng ta không phải là admin.

![image](images/http-user-agent/image-1.png)

Do vậy, chúng ta chỉ cần sửa giá trị của header `User-Agent` thành `admin` và gửi request là có được password.

![image](images/http-user-agent/image-2.png)

## Weak password

> Nothing too difficult

Trang web yêu cầu chúng ta đăng nhập.

![image](images/weak-password/image-1.png)

Như tên thử thách, chúng ta có thể thử đăng nhập với tài khoản và mật khẩu dễ đoán. Tại đây, đăng nhập với `admin:admin` chúng ta được phép truy cập trang web.

![image](images/weak-password/image-2.png)

## PHP - Command injection

> Ping service v1
>
> Find a vulnerabilty in this service and exploit it.
>
> You must manage to read index.php

Theo như tên của thử thách cùng với mô tả, chúng ta biết rằng trang web này dính lỗ hổng OS Command Injection và chúng ta cần đọc file `index.php`.

Trang web cho phép chúng ta nhập vào ip sau đó nó sẽ thực hiện câu lệnh `ping` tới ip đó.

![image](images/php-command-injection/image-1.png)

Chúng ta có thể sử dụng [Burp Suite](https://portswigger.net/burp) để chỉnh sửa request cũng như quan sát response dễ dàng hơn.

Chúng ta sẽ dùng dấu `;` để thực thi được liên tiếp các lệnh. Với payload `; ls`, chúng ta xác định được có file `index.php` ở thư mục hiện tại.

![image](images/php-command-injection/image-2.png)

Và để đọc nội dung của file `index.php`, chúng ta có thể dùng payload `; cat index.php`.

![image](images/php-command-injection/image-3.png)

Vậy là flag ở trong file `.passwd`, chúng ta dùng payload `; cat .passwd` để lấy flag.

![image](images/php-command-injection/image-4.png)

## API - Broken Access

> Follow the Swagger!
>
> Your friend has set up a platform where you can register and post a private note. Everything is based on an API. Before setting up the Front-End, he asked you to check that everything was secure.

Truy cập vào trang chủ, chúng ta thấy giao diện API Documentation được tạo ra bằng Swagger. Nó giúp chúng ta dễ dàng biết và sử dụng được các chức năng mà API cung cấp.

![image](images/api-broken-access/image-1.png)

Trước tiên, tạo một tài khoản tại endpoint `/api/signup`.

![image](images/api-broken-access/image-2.png)

Tạo tài khoản thành công, chúng ta thực hiện đăng nhập tại endpoint `/api/login`. Để ý được cấp cookie `session`, chúng ta sẽ dùng cookie này để có quyền truy cập vào 2 endpoints còn lại.

![image](images/api-broken-access/image-3.png)

Tại endpoint `/api/note`, chúng ta có chức năng cập nhật note.

![image](images/api-broken-access/image-4.png)

Và với endpoint cuối cùng `/api/user`, chúng ta xem được 3 thông tin là `note`, `userid` và `username`.

Tuy nhiên, điểm đáng chú ý tại endpoint này là `userid` có giá trị là `2`. Vậy người dùng với `userid` là `1` sẽ là ai?

![image](images/api-broken-access/image-5.png)

Thông thường, chúng ta có thể xem thông tin người dùng theo id nên thử thêm `/1` vào endpoint `/api/user`.

Thành công, chúng ta đã xem được note chứa flag của người dùng `admin`.

![image](images/api-broken-access/image-6.png)

## Backup file

> No clue.

Thử thách này yêu cầu chúng ta đăng nhập nhưng chúng ta đâu có biết thông tin đăng nhập.

![image](images/backup-file/image-1.png)

Với tên thử thách liên quan đến backup file, có thể nghĩ đến việc đọc các file này để tìm được các thông tin hữu ích.

Chúng ta sẽ sử dụng công cụ [dirsearch](https://github.com/maurosoria/dirsearch) với wordlist [backup_files_only.txt](https://github.com/xajkep/wordlists/blob/master/discovery/backup_files_only.txt) để thực hiện fuzz tìm file backup.

```bash
python3 dirsearch.py -w ~/wordlists/backup_files_only.txt -u http://challenge01.root-me.org/web-serveur/ch11/
```

Sau khi thực hiện lệnh trên, chúng ta nhận thấy server có một file `index.php~`.

```text
[11:23:48] 200 -   843B - /web-serveur/ch11/index.php~                      
```

Tiếp theo, chúng ta sẽ lấy nội dung của file đó bằng lệnh `curl`.

```bash
curl http://challenge01.root-me.org/web-serveur/ch11/index.php~
```

```php
<?php

$username="ch11";
$password="OCCY9AcNm1tj";
...
```

Có được username cùng với password, chúng ta có thể đăng nhập thành công.

![image](images/backup-file/image-2.png)
