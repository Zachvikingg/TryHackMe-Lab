# 🔑 Comprehensive Guide: Cryptography & Password Cracking Basics

Tài liệu này tổng hợp toàn bộ kiến thức chuyên sâu và thực hành từ module **Cryptography** thuộc lộ trình [Cyber Security 101](https://tryhackme.com/paths). Tài liệu bao gồm lý thuyết nền tảng, mô hình toán học đơn giản, và hướng dẫn từng bước sử dụng công cụ bẻ khóa mật mã **John the Ripper**.

---

## I. Tổng quan về Mật mã học (Cryptography)

Mật mã học (Cryptography) là sự kết hợp giữa toán học và tin học nhằm bảo vệ thông tin, ngăn chặn các bên thứ ba (Eve/Kẻ tấn công) đọc được dữ liệu riêng tư giữa người gửi (Alice) và người nhận (Bob).

Mật mã học hiện đại hướng tới việc bảo đảm mô hình **CIA Triad**:
* **Confidentiality (Tính bảo mật):** Đảm bảo chỉ những người có thẩm quyền mới đọc được dữ liệu.
* **Integrity (Tính toàn vẹn):** Đảm bảo dữ liệu không bị sửa đổi trái phép trong quá trình truyền tải.
* **Authenticity/Non-repudiation (Tính xác thực/Chống chối bỏ):** Xác minh chính xác danh tính người gửi; người gửi không thể phủ nhận việc mình đã gửi thông điệp.

### 📑 Các thuật ngữ bắt buộc phải nắm rõ
* **Plaintext ($M$ - Message):** Dữ liệu thô ban đầu (văn bản, hình ảnh, file cấu hình...) chưa qua xử lý.
* **Ciphertext ($C$ - Ciphertext):** Dữ liệu đã bị xáo trộn sau khi đi qua thuật toán mã hóa, không thể đọc hiểu trực tiếp.
* **Cipher/Algorithm ($E$ và $D$):** Hàm toán học dùng để mã hóa ($E$) và giải mã ($D$).
* **Key ($K$):** Chuỗi ký tự hoặc bit bí mật được nạp vào thuật toán để tạo ra tính duy nhất cho kết quả mã hóa.

---

## II. Mã hóa đối xứng (Symmetric Encryption)

Mã hóa đối xứng (Symmetric Encryption) là phương thức sử dụng **cùng một khóa bí mật ($K$)** cho cả hai quá trình mã hóa và giải mã.

$$\text{Mã hóa: } C = E(K, M)$$
$$\text{Giải mã: } M = D(K, C)$$



### 1. Đặc điểm kỹ thuật
* **Tốc độ:** Cực kỳ nhanh do cấu trúc thuật toán chủ yếu dựa trên các phép toán nhị phân cơ bản (XOR, Bit-shifting, Substitution, Permutation).
* **Hiệu năng:** Tiêu tốn rất ít tài nguyên CPU/RAM, thích hợp mã hóa dữ liệu khối lượng lớn (Mã hóa toàn bộ ổ đĩa - Full Disk Encryption, mã hóa tệp tin lưu trữ).
* **Hạn chế lớn nhất:** **Bài toán phân phối khóa (Key Distribution Problem).** Alice và Bob phải tìm cách trao đổi khóa $K$ một cách an toàn trước khi truyền tin. Nếu kẻ tấn công chặn được khóa $K$, toàn bộ hệ thống sụp đổ.

### 2. Các thuật toán tiêu biểu
* **DES (Data Encryption Standard):** Kích thước khóa quá ngắn (56-bit), hiện tại có thể bị bẻ khóa bằng phương pháp Brute-force chỉ trong vài giờ. Không còn an toàn.
* **3DES (Triple DES):** Áp dụng thuật toán DES 3 lần với các khóa khác nhau để tăng độ dài khóa lên tối đa 168-bit. Tuy nhiên, tốc độ xử lý chậm và đang dần bị khai tử.
* **AES (Advanced Encryption Standard):** Tiêu chuẩn vàng hiện tại trên toàn thế giới. Hỗ trợ các kích thước khóa **128-bit, 192-bit, và 256-bit**. Cho đến nay, chưa có phương pháp tấn công toán học khả thi nào bẻ gãy được AES nếu cấu hình đúng quy chuẩn.

---

## III. Mã hóa bất đối xứng (Asymmetric / Public-Key Cryptography)

Để giải quyết triệt để điểm yếu lộ khóa của mã hóa đối xứng, mật mã học bất đối xứng sử dụng một **cặp khóa (Key Pair)** có mối quan hệ toán học chặt chẽ nhưng không thể suy ra nhau:
* **Public Key ($K_{pub}$):** Khóa công khai, chia sẻ tự do cho tất cả mọi người. Dùng để **Mã hóa** hoặc **Xác thực chữ ký**.
* **Private Key ($K_{priv}$):** Khóa bí mật, phải được lưu trữ tuyệt đối an toàn và chỉ duy nhất người sở hữu biết. Dùng để **Giải mã** hoặc **Tạo chữ ký**.



### 1. Kịch bản Bảo mật dữ liệu (Confidentiality)
Khi Alice muốn gửi một bức thư tuyệt mật cho Bob:
1. Alice lấy **Public Key của Bob ($K_{pub\_Bob}$)** để tiến hành mã hóa thông điệp.
2. Bản mã $C$ được gửi qua Internet công cộng. Ngay cả khi kẻ tấn công chặn được $C$, họ cũng không thể làm gì.
3. Chỉ duy nhất **Private Key của Bob ($K_{priv\_Bob}$)** mới có khả năng giải mã bản mã $C$ này để lấy lại nội dung gốc.

### 2. Kịch bản Chữ ký số & Xác thực danh tính (Authenticity & Integrity)
Khi Bob muốn gửi một hợp đồng và chứng minh chính mình là người gửi (không ai giả mạo được):
1. Bob băm hợp đồng thành một chuỗi đại diện, sau đó mã hóa chuỗi băm đó bằng **Private Key của mình ($K_{priv\_Bob}$)**. Kết quả thu được gọi là **Chữ ký số (Digital Signature)**.
2. Alice nhận được hợp đồng kèm chữ ký số. Cô sử dụng **Public Key của Bob ($K_{pub\_Bob}$)** để giải mã chữ ký.
3. Nếu giải mã thành công và chuỗi băm khớp hoàn toàn với tệp tin hợp đồng, Alice có thể khẳng định 100%: Hợp đồng này chắc chắn do Bob ký và nội dung không hề bị chỉnh sửa trên đường truyền.

### 3. Ứng dụng thực tế
Do mã hóa bất đối xứng sử dụng các phép toán lũy thừa số nguyên lớn (như thuật toán RSA, ECC) nên tốc độ xử lý rất chậm. Thực tế, hệ thống sẽ kết hợp cả hai loại:
* Sử dụng **Mã hóa bất đối xứng** để bắt tay, xác thực danh tính và trao đổi một "Khóa phiên" (Session Key - Khóa đối xứng).
* Sử dụng **Mã hóa đối xứng** với Khóa phiên vừa trao đổi để mã hóa toàn bộ dữ liệu truyền tải sau đó (Cơ chế cốt lõi của **HTTPS/SSL/TLS**, **SSH**, **VPN**).

---

## IV. Hàm băm (Hashing Basics)

**Lưu ý quan trọng:** Hàm băm (Hashing) **KHÔNG PHẢI** là mã hóa. Mã hóa là quá trình hai chiều (Mã hóa ➡️ Giải mã), còn Hashing là **quá trình toán học một chiều (One-way)**.



### 1. Bốn thuộc tính bắt buộc của một hàm băm an toàn
* **One-way (Tính một chiều):** Cho trước chuỗi băm $H$, không khả thi về mặt thời gian và toán học để tìm lại dữ liệu gốc $M$ sao cho $Hash(M) = H$.
* **Fixed-length output (Đầu ra cố định):** Bất kể dữ liệu đầu vào có kích thước ra sao (1 ký tự, 1 đoạn văn, hay 1 file ISO 4GB), chuỗi băm đầu ra luôn có độ dài cố định về số bit/ký tự.
* **Avalanche Effect (Hiệu ứng thác đổ):** Chỉ cần thay đổi một bit cực nhỏ ở dữ liệu đầu vào (ví dụ biến chữ thường thành chữ hoa, thêm một dấu cách), toàn bộ chuỗi băm đầu ra sẽ thay đổi hoàn toàn và không có bất kỳ mối liên hệ nào với chuỗi băm cũ.
* **Collision Resistance (Kháng va chạm):** Không thể tìm thấy hai chuỗi đầu vào khác nhau $M_1 \neq M_2$ sao cho $Hash(M_1) = Hash(M_2)$.

### 2. Các thuật toán băm phổ biến

| Thuật toán | Độ dài đầu ra | Trạng thái an toàn hiện tại |
| :--- | :--- | :--- |
| **MD5** | 128 bits (32 ký tự Hex) | ❌ **Không an toàn.** Đã bị bẻ gãy hoàn toàn bởi các cuộc tấn công va chạm (Collision). Hiện tại chỉ dùng để kiểm tra tính toàn vẹn file tải về ở mức cơ bản, tuyệt đối không dùng bảo mật mật khẩu. |
| **SHA-1** | 160 bits (40 ký tự Hex) | ❌ **Không an toàn.** Google và các tổ chức lớn đã chứng minh thực tế việc tạo ra va chạm SHA-1 thành công vào năm 2017. |
| **SHA-256** | 256 bits (64 ký tự Hex) |  **An toàn.** Thuộc gia đình SHA-2, hiện tại là tiêu chuẩn bảo mật cho việc chứng thực ứng dụng, chữ ký số và hệ thống Blockchain. |
| **NTLM** | 128 bits (32 ký tự Hex) | ⚠️ **Lạc hậu.** Giao thức băm mật khẩu nội bộ của hệ điều hành Windows. Lưu trữ không có muối (Salt), tốc độ tính toán quá nhanh khiến nó cực kỳ dễ bị bẻ khóa hàng loạt nếu kẻ tấn công lấy được tệp SAM. |

---

## V. Kỹ thuật bẻ khóa mã băm (Hash Cracking)

Do cấu trúc toán học một chiều, kẻ tấn công không thể áp dụng công thức để "giải mã" ngược chuỗi băm. Cách duy nhất để tìm lại mật khẩu văn bản thô (Plaintext) là sử dụng kỹ thuật đoán mò và đối chiếu kết quả băm:
