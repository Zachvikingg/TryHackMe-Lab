Đầu tiên ta sẽ dò quét cơ bản bằng `nmap`
```
nmap -sC -sV 10.49.133.230
```
Từ đó ta được 
```
Starting Nmap 7.95 ( https://nmap.org ) at 2026-05-29 02:55 EDT
Nmap scan report for 10.49.133.230
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
Vì đề bài là `SQHell` nên ta ưu tiên khai thác cổng 80 trước 
Truy cập vào địa chỉ `10.49.133.230` bằng trình duyệt ( ở đây tôi dùng trình duyệt trên Burp luôn ) 
<img width="1913" height="859" alt="image" src="https://github.com/user-attachments/assets/4878ff0e-7ec2-401d-a712-9cf01746de06" />
Thử đọc page source nhưng có vẻ chưa có gì bất thường ( vì tên bài ta có thể đoán là lỗi khả năng ở sql) 
Nhìn qua ta có thể thấy có các mục sau là nơi thêm dữ liệu người dùng nhập là ô `login` `register`  ngoài ra ta còn biết 1 user là admin
Ta sẽ thử vào mục `login` trước vì khả năng nó là `In-Band`   
Tôi đã thử nhập vào mục username `admin`,`admin'`,`admin"` nhưng kết quả đều là không hợp lệ ,flag trả về khi nhập đúng là `admin ' or 1=1 -- `
`THM{FLAG1:E786483E5A53075750F1FA792E823BD2}`
Tiếp theo tôi chọn mục `admin` :
<img width="1919" height="845" alt="image" src="https://github.com/user-attachments/assets/377f9e62-2076-4965-8d4b-a09306984dbd" />
Nhận thấy id trên url có thể khai thác ta ném nó qua burp, tôi thử thay giá trị `1` thành `2` và kết quả trả về là `Cannot find user` tôi thử nhập `2 or 1=1 ` thì kết quả lại trả ra kết quả hợp lệ chứng tỏ nó có lỗi sqli .Tiếp theo ta sẽ dùng lệnh để thử xác định số cột 
`1 order by 3 -- ` nhập lệnh trên kết quả trả về ra màn hình bình thường còn  `1 order by 4 -- ` thì trả về lỗi chứng tỏ có 3 cột 
<img width="1601" height="851" alt="image" src="https://github.com/user-attachments/assets/223a184f-f9b7-4354-9881-0e120b50beb7" />
Nhập `2 union select 'asd','asdf','asdf'--` thì kết quả là : 
<img width="1606" height="852" alt="image" src="https://github.com/user-attachments/assets/b06f0e2d-1bfb-433c-86ca-3f2ed5bba567" />
Như vậy dữ liệu được union vào được in thẳng ra màn hình từ đây ta có thể dễ dàng tìm ra thông tin chi tiết của các bảng các cột 
Thay dữ liệu cột 2 từ `'asdf'` thành `database()` ta được database() là `sqhell_4`
Tiếp tục tìm các bảng trong databse() trên qua câu lệnh `2 union select 'asd',group_concat(table_name),3 from information_schema.tables where table_schema = 'sqhell_4'  --` kết quả ta thu được table `users` ,tiếp tục tìm các cột trong bảng qua lệnh `` 1















Tiếp theo tôi sẽ khai thác sang mục `register` 
<img width="692" height="476" alt="image" src="https://github.com/user-attachments/assets/3f953292-612d-4a77-aac1-e489e843f6f0" />
Kết quả khi nhập `admin` vào đã cho thấy có username tên admin tồn tại trên hệ thống 
Khi nhập username khác thì trang web hiện kết quả như ảnh : 
<img width="727" height="240" alt="image" src="https://github.com/user-attachments/assets/a1fe8968-8cc6-492e-ab7f-85bc5931a255" />
Tôi đã thử login bằng tài khoản vừa đăng kí nhưng không được có vẻ chức năng này chỉ có tác dụng cho ta biết username nào hợp lệ thôi 
Giờ ta sẽ xoáy sâu vào phần username vì kết quá nó trả ra là đúng hoặc sai tức là nó thuộc dạng `Boolean-Based` 
<img width="1603" height="848" alt="image" src="https://github.com/user-attachments/assets/87b725a3-b83b-4880-a8b7-f36b20511af5" />
Dùng burp để chỉnh sửa cho nhanh : 
<img width="1595" height="855" alt="image" src="https://github.com/user-attachments/assets/3c97779f-5931-4e02-8dc2-edc9426d9636" />
Ta có thể thấy khi ta nhập `admin` vào username thì web trả ra giá trị `"available":false`
<img width="1606" height="856" alt="image" src="https://github.com/user-attachments/assets/a47268ca-e25b-4d75-8deb-8e3747d969e5" />
Khi tôi nhập username hợp lệ thì nó trả ra giá trị `true` , tôi đã thử nhập `admin'` nhưng hệ thống vẫn báo hợp lệ





