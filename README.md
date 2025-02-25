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

## HTTP - Directory indexing

> CTRL+U...

Khi truy cập trang web, chúng ta sẽ chẳng thấy gì. Tuy nhiên, nếu chúng ta xem HTML source của trang sẽ thấy một phần comment thú vị.

![image](images/http-directory-indexing/image-1.png)

Theo đường dẫn tới file `pass.html` cũng không có password cho chúng ta.

![image](images/http-directory-indexing/image-2.png)

Nhưng sẽ ra sao nếu chúng ta chỉ vào `admin`?

Bùm! Chúng ta thấy file và thư mục trong folder `admin`. Đáng chú ý là có một thư mục `backup`.

![image](images/http-directory-indexing/image-3.png)

Và khi truy cập thư mục `backup`, chúng ta thấy có file `admin.txt`.

![image](images/http-directory-indexing/image-4.png)

Cuối cùng, truy cập vào file `admin.txt`, chúng ta sẽ lấy được password.

![image](images/http-directory-indexing/image-5.png)

## HTTP - Headers

> HTTP response give informations
>
> Get an administrator access to the webpage.

Truy cập vào thử thách, chúng ta thấy một dòng chữ bảo rằng nội dung của trang web không phải là phần duy nhất ở trong HTTP response:

![image](images/http-headers/image-1.png)

Nếu chúng ta quan sát response trong Burp Suite sẽ thấy có header `Header-RootMe-Admin`:

![image](images/http-headers/image-2.png)

Vậy, chúng ta sẽ thêm header đó vào trong request để nhận được password:

![image](images/http-headers/image-3.png)

## HTTP - POST

> Do you know HTTP?
>
> Find a way to beat the top score!

Thử thách này yêu cầu chúng ta phải có score lớn hơn `999999` mới lấy được flag.

![image](images/http-post/image-1.png)

Khi nhấn "Give a try!", chúng ta sẽ có một score ngẫu nhiên nhỏ hơn `999999` và chúng ta thua.

![image](images/http-post/image-2.png)

Bên dưới là POST request khi chúng ta nhấn nút đó. Có thể thấy là score đang được truyền vào tham số `score`.

![image](images/http-post/image-3.png)

Vậy chúng ta có thể thay đổi giá trị của tham số `score` thành `1000000` và gửi lại request để lấy được flag.

![image](images/http-post/image-4.png)

## HTTP - Improper redirect

> Don’t trust your browser
>
> Get access to index.

Khi bắt đầu thử thách, chúng ta sẽ thấy giao diện trang web như sau:

![image](images/http-improper-redirect/image-1.png)

Nó bắt chúng ta phải đăng nhập. Tuy nhiên, tên của thử thách liên quan tới redirect, có lẽ chúng ta đang được chuyển tới trang `login.php` từ `index.php`.

Vậy chúng ta có thể thực hiện lệnh `curl` tới `http://challenge01.root-me.org/web-serveur/ch32/` để không bị redirect. Từ đó có được flag:

```text
$ curl http://challenge01.root-me.org/web-serveur/ch32/
<html>
<body><link rel='stylesheet' property='stylesheet' id='s' type='text/css' href='/template/s.css' media='all' /><iframe id='iframe' src='https://www.root-me.org/?page=externe_header'></iframe>
<h1>Welcome !</h1>

<p>Yeah ! The redirection is OK, but without exit() after the header('Location: ...'), PHP just continue the execution and send the page content !...</p>
<p><a href="http://cwe.mitre.org/data/definitions/698.html">CWE-698: Execution After Redirect (EAR)</a></p>
<p>The flag is : ExecutionAfterRedirectIsBad
</p>
</body>
</html>

```

## HTTP - Verb tampering

> HTTP authentication
>
> Bypass the security establishment.

Ở thử thách này, trang web sử dụng Basic Authentication, yêu cầu chúng ta đăng nhập để có quyền truy cập.

![image](images/http-verb-tampering/image-1.png)

Chúng ta có thể thử đăng nhập với `admin:admin` nhưng sẽ không thành công.

![image](images/http-verb-tampering/image-2.png)

Tên thử thách cũng đã gợi ý là chúng ta cần thay đổi request method.

Ở đây, chúng ta sẽ sử dụng method `OPTIONS` để bypass thành công và nhận được password:

![image](images/http-verb-tampering/image-3.png)

## Install files

> You know phpBB ?

Vào thử thách, chúng ta có một trang web trống trơn:

![image](images/install-files/image-1.png)

Xem HTML source code, chúng ta thấy có một đường dẫn `/web-serveur/ch6/phpbb`:

![image](images/install-files/image-2.png)

Truy cập vào `/web-serveur/ch6/phpbb` nhưng lại không có gì:

![image](images/install-files/image-3.png)

Sử dụng [dirsearch](https://github.com/maurosoria/dirsearch), chúng ta có thể tìm ra đường dẫn `/web-serveur/ch6/phpbb/install/`:

```text
$ python3 dirsearch.py -x 403 -u http://challenge01.root-me.org/web-serveur/ch6/phpbb

  _|. _ _  _  _  _ _|_    v0.4.3
 (_||| _) (/_(_|| (_| )

Extensions: php, asp, aspx, jsp, html, htm | HTTP method: GET | Threads: 25 | Wordlist size: 12266

Target: http://challenge01.root-me.org/

[10:38:09] Scanning: web-serveur/ch6/phpbb/
[10:40:25] 200 -   295B - /web-serveur/ch6/phpbb/index.html
[10:40:27] 301 -   162B - /web-serveur/ch6/phpbb/install  ->  http://challenge01.root-me.org/web-serveur/ch6/phpbb/install/
[10:40:27] 200 -   12KB - /web-serveur/ch6/phpbb/install/
CTRL+C detected: Pausing threads, please wait...

Task Completed
```

Truy cập vào `/web-serveur/ch6/phpbb/install/`, chúng ta có một file `install.php`:

![image](images/install-files/image-4.png)

Vào file `install.php`, chúng ta lụm được mật khẩu:

![image](images/install-files/image-5.png)

## Nginx - Alias Misconfiguration

> Off By Slash
>
> Our company’s web developer has finished developing the new intranet.\
> Mission: You must assess the security of this site before it goes live.

![image](images/nginx-alias-misconfiguration/image-1.png)

Thử thách này liên quan đến việc khai thác việc cấu hình sai Nginx alias dẫn đến lỗ hổng Path Traversal. Chúng ta có thể tìm kiếm được một số bài viết liên quan:

- <https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-web/nginx.html#alias-lfi-misconfiguration>
- <https://www.acunetix.com/vulnerabilities/web/path-traversal-via-misconfigured-nginx-alias/>

Xem HTML source code, chúng ta thấy có phần comment đề cập tới `/assets/`:

![image](images/nginx-alias-misconfiguration/image-2.png)

Truy cập vào `/assets../`, chúng ta có thể xem được các files và thư mục:

![image](images/nginx-alias-misconfiguration/image-3.png)

Nhấn vào "flag.txt", chúng ta đọc được flag:

![image](images/nginx-alias-misconfiguration/image-4.png)

## API - Mass Assignment

> Anyway, I had no further use for it.
>
> Your friend thanks you for your previous vulnerability report, and assures you that this time he has removed the possibility of accessing notes, and has even created an administration role!

Vào URL của thử thách, chúng ta có giao diện API Documentation như sau:

![image](images/api-mass-assignment/image-1.png)

API có 5 endpoints, chúng ta sẽ lần lượt xem qua chức năng ở từng endpoint.

Tại endpoint `/api/signup`, cho phép chúng ta tạo một người dùng với `username` và `password`:

![image](images/api-mass-assignment/image-2.png)

Tại endpoint `/api/login`, thực hiện đăng nhập với thông tin tài khoản:

![image](images/api-mass-assignment/image-3.png)

Tại endpoint `/api/user`, chúng ta có thể xem thông tin về người dùng:

![image](images/api-mass-assignment/image-4.png)

Tại endpoint `/api/note`, cho phép cập nhật ghi chú của người dùng với nội dung ở tham số `note`:

![image](images/api-mass-assignment/image-5.png)

Và ở endpoint `/api/flag`, cho phép lấy flag nếu chúng ta là admin:

![image](images/api-mass-assignment/image-6.png)

Tiến hành khai thác, chúng ta sẽ đăng ký một người dùng có `username` là `foo` và `password` là `bar`:

![image](images/api-mass-assignment/image-7.png)

Khi đăng nhập thành công, chúng ta được cấp một cookie `session` để có thể truy cập vào các endpoints còn lại:

![image](images/api-mass-assignment/image-8.png)

Khi gửi GET request tới `/api/user`, nhận về kết quả chứa một số thông tin của người dùng, chúng ta sẽ chú ý tới `"status":"guest"`:

![image](images/api-mass-assignment/image-9.png)

Thử chức năng cập nhật ghi chú, không có gì đặc biệt:

![image](images/api-mass-assignment/image-10.png)

Truy cập vào `/api/flag`, chúng ta xác định cần phải là admin mới có thể lụm flag:

![image](images/api-mass-assignment/image-11.png)

Quay trở lại endpoint `/api/user`, nếu chúng ta gửi request với method `OPTIONS` sẽ thấy server còn hỗ trợ thêm method `PUT`:

![image](images/api-mass-assignment/image-12.png)

Chúng ta sẽ khai thác lỗ hổng Mass Assignment tại endpoint `/api/user` bằng cách thêm `{"status":"admin"}` vào request body, chú ý thêm header `Content-Type: application/json`:

![image](images/api-mass-assignment/image-13.png)

Giờ, chúng ta sẽ lụm flag thành công nếu truy cập vào endpoint `/api/flag`:

![image](images/api-mass-assignment/image-14.png)

## CRLF

> Inject false data in the journalisation log.

Vào thử thách, chúng ta có một trang web yêu cầu đăng nhập để xác thực và hiển thị nội dung log:

![image](images/crlf/image-1.png)

Đăng nhập thử `foo:bar`, chúng ta thấy thông báo người dùng đăng nhập được ghi vào log:

![image](images/crlf/image-2.png)

Như tên thử thách sẽ liên quan tới [CRLF Injection](https://book.hacktricks.wiki/en/pentesting-web/crlf-0d-0a.html#crlf-0d0a-injection), khai thác bằng cách chèn ký tự `%0D%0A` vào input để làm thay đổi log.

Cộng thêm việc trong log chúng ta thấy chuỗi "admin authenticated." nên hiểu rằng cần tạo một log như vậy để xác thực thành công. Chúng ta sẽ tiến hành khai thác bằng cách thêm `admin+authenticated.%0D%0Ahello` vào tham số `username` bởi giá trị của tham số này đang được ghi vào log:

![image](images/crlf/image-3.png)
