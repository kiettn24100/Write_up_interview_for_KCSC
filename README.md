# Write_up_interview_for_KCSC

vào challenge cho một giao diện như vậy , cho phép chúng ta nhập name , upload file và select theme

<img width="647" height="923" alt="image" src="https://github.com/user-attachments/assets/5e875218-90ff-4f62-83a3-486f6896bf5f" />

thử upload 1 file `shell.php` nhưng nó báo `File .php not allowed!`

<img width="1397" height="766" alt="image" src="https://github.com/user-attachments/assets/0a873bc9-8bab-41ef-9d17-e932e5605e32" />

nhận thấy khi mà Select theme thì ở dòng đầu tiên request ?theme=dark.html hoặc light.html hoặc blue.html , purple.html tức là đang gọi 1 file .html để làm màu nền cho web 

mình thử thay bằng `?theme=flag.txt`

<img width="1416" height="223" alt="image" src="https://github.com/user-attachments/assets/9328b2ce-d34a-40d1-a9de-187df4b3008e" />

nó trả về mình thấy 1 đoạn thế này 
`include(): Failed opening 'themes/flag.txt' for inclusion (include_path='.:/usr/local/lib/php') in <b>/var/www/html/index.php</b>`

tức là ở phía backend đang sử dụng hàm `include()` nhận đường dẫn là `themes/?theme=...` và hàm include() này nhiệm vụ của nó là đọc file và thực thi file 

mình thử check lại coi có thể khai thác LFI ở đây ko nên mình thử bằng 1 file luôn luôn có sẵn trong hệ thống là `/etc/passwd` , nhưng vì đang ở trong /themes cũng ko biết nó đang ở vị trí nào trong hệ thống nên dùng `../` để lùi đường dẫn lại cho đến khi nào mà đường dẫn hợp lệ thì kết quả tự trả về file thông tin cấu hình ( theo lí thuyết là vậy ) 

<img width="1423" height="685" alt="image" src="https://github.com/user-attachments/assets/815b562c-0a15-49fe-8194-8e007192bc67" />

vậy là có thể kết luận được có LFI ở đây , lúc này chỉ cần tìm cấu trúc của các đường dẫn trong hệ thống

ở dưới phần responese , lúc mà mình upload ảnh lên thì có 1 dòng nó hiện lên nơi lưu trữ ảnh của mình upload : `<img src="uploads/19b945bf00a36eee/avatar.jpg" class="avatar">`

như vậy là đã biết được file upload lên được lưu ở đâu , và kết hợp với việc lúc đầu mình đã upload 1 file `.jpg` nhưng nội dung là PHP 
```python
<?php
system('ls -la');
?>
```
rồi mình lại sử dụng lùi đường dẫn như hồi nãy , và kết quả là khi mà ?theme=../uploads/19b945bf00a36eee/avatar.jpg thì nó đã trả về toàn bộ file ở trong hệ thống

<img width="1416" height="698" alt="image" src="https://github.com/user-attachments/assets/2c36c290-fc0c-4251-a053-d2977a084108" />

nhận thấy có file `7h1s_1s_4_fl4g_f1l3.txt` ở ngay root 

và thế là xong `system('cat /7h1s_1s_4_fl4g_f1l3.txt')` 

<img width="1421" height="726" alt="image" src="https://github.com/user-attachments/assets/27b8c284-2bfb-4944-9418-f7d82e82ab56" />

`KCSC{Chuc mung ban da RCE duoc}`

***ở đây vì sao lại có thể dùng .jpg mà lại thực thi được các lệnh linux***

ở phía back-end nó sử dụng hàm `include()` để lấy nội dung file và thực thi , thiết lập sẵn ở trong `include(themes/?theme=)` cho nên khi truyền `?theme=../uploads/19b945bf00a36eee/avatar.jpg` thì sẽ là `include(themes/../uploads/19b945bf00a36eee/avatar.jpg` lúc mà hàm include() tìm đến file avatar.jpg , nó sẽ đọc nội dung và thực thi nội dung ở trong đó , vì vậy mà có thể dùng các lệnh system() để điều khiển và lấy flag


