# 🔑 Cryptography Basics & John the Ripper Notes

Tài liệu tổng hợp kiến thức chi tiết và hệ thống về Mật mã học (Cryptography) và kỹ thuật bẻ khóa mật mã bằng công cụ John the Ripper thuộc lộ trình học Cyber Security.

---

## 1. Tổng quan về Mật mã học (Cryptography)

Mật mã học là khoa học về việc bảo vệ thông tin bằng cách biến đổi nó (mã hóa) thành một định dạng mà người không phận sự không thể đọc được, đảm bảo các yếu tố cốt lõi: **Tính bảo mật (Confidentiality)**, **Tính toàn vẹn (Integrity)**, và **Tính xác thực (Authenticity)**.

### 🔄 Các thuật ngữ cơ bản
* **Plaintext (Văn bản thô):** Dữ liệu gốc ban đầu khi chưa được mã hóa mà ai cũng có thể đọc và hiểu được.
* **Ciphertext (Văn bản mã hóa):** Dữ liệu sau khi đã đi qua thuật toán mã hóa, trông như một chuỗi ký tự hỗn độn vô nghĩa.
* **Cipher (Thuật toán mã hóa):** Tập hợp các quy tắc toán học được sử dụng để tiến hành mã hóa hoặc giải mã.
* **Key (Khóa):** Một chuỗi thông tin (số, ký tự) kết hợp với thuật toán để tạo ra tính duy nhất cho văn bản mã hóa.

---

## 2. Mã hóa đối xứng (Symmetric Encryption)

Mã hóa đối xứng là phương thức sử dụng **cùng một khóa duy nhất** cho cả hai quá trình: Mã hóa dữ liệu (Plaintext ➡️ Ciphertext) và Giải mã dữ liệu (Ciphertext ➡️ Plaintext).



### ⚖️ Đặc điểm
* **Ưu điểm:** Tốc độ xử lý cực kỳ nhanh, tiêu tốn ít tài nguyên phần cứng, phù hợp để mã hóa lượng dữ liệu lớn (như ổ cứng, file dung lượng cao).
* **Nhược điểm:** Bài toán phân phối khóa (Key Distribution). Nếu kẻ tấn công chặn được khóa này trên đường truyền, toàn bộ dữ liệu sẽ bị lộ.

### 🛡️ Các thuật toán phổ biến
* **DES / 3DES:** Thuật toán cũ, hiện tại không còn an toàn do độ dài khóa ngắn.
* **AES (Advanced Encryption Standard):** Chuẩn mã hóa tiên tiến và phổ biến nhất hiện nay, cực kỳ an toàn với các kích thước khóa 128-bit, 192-bit và 256-bit.

---

## 3. Mã hóa bất đối xứng (Asymmetric / Public-Key Cryptography)

Mã hóa bất đối xứng giải quyết bài toán chia sẻ khóa của mã hóa đối xứng bằng cách sử dụng **một cặp khóa (Key Pair)** có mối quan hệ toán học chặt chẽ với nhau:

* **Public Key (Khóa công khai):** Được chia sẻ rộng rãi cho bất kỳ ai. Dùng để **Mã hóa** dữ liệu.
* **Private Key (Khóa bí mật):** Phải được giữ bí mật tuyệt đối bởi người sở hữu. Dùng để **Giải mã** dữ liệu.



### ⚙️ Nguyên lý hoạt động (Ví dụ: RSA)
1. Nếu Alice muốn gửi thư bảo mật cho Bob, Alice sẽ lấy **Public Key của Bob** để mã hóa bức thư.
2. Khi Bob nhận được tệp tin mã hóa, Bob là người duy nhất có thể giải mã nó bằng cách sử dụng **Private Key của chính mình**.

### 🖋️ Ứng dụng trong Chữ ký số (Digital Signature)
* Để chứng minh danh tính, người gửi sẽ mã hóa văn bản bằng **Private Key của họ**.
* Người nhận sử dụng **Public Key của người gửi** để giải mã. Nếu giải mã thành công, điều đó chứng minh bức thư chắc chắn đến từ người gửi đó và nội dung không bị chỉnh sửa.
* **Giao thức thực tế:** Áp dụng cốt lõi trong SSH, HTTPS (SSL/TLS), và chứng chỉ số.

---

## 4. Hàm băm (Hashing Basics)

Băm (Hashing) **không phải là mã hóa**. Khác với mã hóa là quá trình hai chiều (có thể giải mã ngược lại), băm là **quá trình một chiều (One-way function)** toán học biến đổi một dữ liệu có độ dài bất kỳ thành một chuỗi ký tự có độ dài cố định.

### 🎯 Các đặc tính cốt lõi của Hàm băm
1. **Tính một chiều (One-way):** Về mặt toán học, không thể suy ngược từ chuỗi băm (Hash value) ra dữ liệu gốc (Plaintext).
2. **Tính cố định (Fixed-length output):** Dù tệp tin gốc là 1 từ hay 1 bộ phim 10GB, kết quả băm của cùng một thuật toán luôn có độ dài ký tự bằng nhau.
3. **Hiệu ứng thác đổ (Avalanche Effect):** Chỉ cần thay đổi 1 ký tự rất nhỏ ở dữ liệu gốc (ví dụ dấu chấm), chuỗi băm đầu ra sẽ thay đổi hoàn toàn.
4. **Kháng va chạm (Collision Resistance):** Hai dữ liệu gốc khác nhau rất hiếm khi (hoặc không thể) cho ra cùng một chuỗi băm giống nhau.

### 📊 Các thuật toán băm thông dụng

| Thuật toán | Độ dài đầu ra | Trạng thái an toàn |
| :--- | :--- | :--- |
| **MD5** | 128 bits (32 ký tự Hex) | ❌ Không an toàn (Dễ bị tấn công va chạm - Collision) |
| **SHA-1** | 160 bits (40 ký tự Hex) | ❌ Không an toàn (Đã bị bẻ gãy thực tế) |
| **SHA-256** | 256 bits (64 ký tự Hex) |  An toàn (Tiêu chuẩn hiện tại cho hệ thống bảo mật) |
| **NTLM** | 128 bits (32 ký tự Hex) | ⚠️ Sử dụng trong Windows, dễ bị tấn công Brute-force/Rainbow Table |

---

## 5. Tấn công bẻ khóa mã băm (Hash Cracking)

Vì không thể "giải mã" hàm băm bằng toán học, kẻ tấn công phải sử dụng phương pháp thử và sai (Guessing): Băm một từ khóa ngẫu nhiên rồi đối chiếu kết quả xem có trùng với chuỗi băm mục tiêu hay không.

### 💥 Các hình thức tấn công
* **Brute-Force Attack:** Thử mọi sự kết hợp có thể có của các ký tự (ví dụ: `aaaa`, `aaab`, `aaac`...). Tốn rất nhiều thời gian và tài nguyên phần cứng.
* **Dictionary Attack (Tấn công từ điển):** Sử dụng một danh sách các từ có sẵn (Wordlist - ví dụ file `rockyou.txt` chứa hàng triệu mật khẩu phổ biến) để tiến hành băm thử nghiệm. Nhanh và hiệu quả hơn rất nhiều so với Brute-force.

---

## 6. Công cụ bẻ khóa: John the Ripper (JTR)

John the Ripper là một trong những công cụ bẻ khóa mật khẩu và mã băm mạnh mẽ, phổ biến nhất thế giới được tích hợp sẵn trong Kali Linux.

### 💻 Các câu lệnh JTR cơ bản

* **Bẻ khóa tự động (JTR tự nhận diện định dạng hash):**
  ```bash
  john [file_chứa_chuỗi_băm]
