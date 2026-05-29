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






