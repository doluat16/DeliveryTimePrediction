# BÀI TOÁN DỰ ĐOÁN THỜI GIAN GIAO ĐỒ ĂN NHANH
Bài toán dự đoán thời gian giao hàng dựa trên các dữ liệu đầu vào như: Tuổi và độ đánh giá của nhân viên giao hàng, tọa độ nhà hàng, thời gian đặt hàng, thời gian nhân viên lấy hàng, điều kiện thời tiết, giao thông, phương tiện di chuyển. 

![data đầu vào](data_goc.JPG)

<h3> Tổng thời gian giao hàng = Thời gian quán chuẩn bị món + Thời gian vận chuyển. </h3>

Ta đã biết Tổng thời gian giao hàng và Thời gian chuẩn bị món. Vì vậy, ta sẽ xây dựng một mô hình để dự đoán Thời gian vận chuyển. Sau khi dự đoán được Thời gian vận chuyển, chúng ta sẽ cộng nó với Thời gian chuẩn bị món để tính ra Tổng thời gian giao hàng


## Tiền xử lý dữ liệu
#### Loại bỏ những dữ liệu dư thừa
- Xóa những cột không dùng đến 
- Chuyển đổi dữ liệu ở các cột về đúng định dạng
#### Trích xuất dữ liệu
-  Tính khoảng cách đường bộ từ các cặp tọa độ bằng OSRM (sử dụng dữ liệu từ OpenStreetMap). Sau đó thêm khoảng cách vừa tính được (distanc_km) vào data.
- Trích xuất Date sang ngày, tháng, năm 
(VD: 20/10/2023 -> day: 20, month:10, year:2023)
#### Xử lý dữ liệu thiếu, giá trị ngoại lai
![Dữ liệu thiếu](DL_thieu.png)

- Xử lý thứ tự dữ liệu thiếu, giá trị ngoại lai theo tầm quan trọng của các đặc trưng.
- Trực quan hóa dữ liệu ở từng cột, sau đó thay thế các giá trị thiếu ở các cột bằng các phương pháp khác nhau: thay thế bằng giá trị trung bình, xuất hiện nhiều nhất, trung vị... 
- Sử dụng biểu đồ Histogram, Box-Plot, phân tán để trực quan hóa dữ liệu và phát hiện các giá trị ngoại lai. Biểu đồ này hiển thị phân phối của dữ liệu dựa trên các giá trị phân vị (quartile) và xác định các giá trị ngoại lai (outliers) là những giá trị nằm ngoài phạm vi của 1.5 lần IQR từ các phân vị Q1 (25%) và Q3 (75%).
Công thức tính IQR và phát hiện ngoại lai:

    **IQR = Q3 - Q1**
    *Lower Bound = Q1 - 1.5 * IQR*
    *Upper Bound = Q3 + 1.5 * IQR*
    Các giá trị nằm ngoài phạm vi [Lower Bound, Upper Bound] được coi là giá trị ngoại lai.
![alt text](rating_trckhiXL.png)
![alt text](rating_saukhixl.png)

![alt text](phantan_trc.png)
- Ta thấy có một lượng thời gian vận chuyển <0, nguyên nhân có thể do khi hệ thống đã điền thiếu Thời gian vận chuyển hoặc Thời gian chuẩn bị món, dẫn đến thời gian vận chuyển bị âm. Do đó, ta sẽ thay thế các giá trị ở những cột âm bằng các cách như: giá trị trung bình, trung vị, xuất hiện nhiều nhất hay giá trị gần 0. Sau khi thử các cách, ta sẽ được biểu đồ phân tán mới như sau.
![alt text](phantansau.png)

- Việc xử lý trên cho thấy hiệu quả khi nó tăng độ đo đánh giá R2 (0.8 -> 0.82).
![alt text](image.png)
- Biểu đồ cho thấy những người trẻ thường sẽ có thời gian giao hàng nhanh hơn so với người lớn tuổi.
![alt text](time.png)
- Số lượng đơn hàng nhiều nhất vào khoảng 20h-21h khi giao thông tắc nghẽn. Điều này có thể là do nhu cầu đặt hàng vào buổi tối sau giờ làm việc.
Buổi sáng và buổi tối muộn là khoảng thời gian giao thông thấp nhưng vẫn có một lượng đơn hàng nhất định.
## Mã hóa dữ liệu
- Sử dụng mã hóa Ordinal Encoding và One hot Encoding với những đặc trưng khác nhau. 
![alt text](mahoa.JPG)

## Xây dựng và huấn luyện mô hình
#### Chia dữ liệu
Chia dữ liệu thành tập huấn luyện và tập kiểm tra theo tỷ lệ 80:20.
#### Lựa chọn và huấn luyện mô hình
- Sử dụng các thuật toán học máy: Linear Regression, Random Forest,  Neural Networks, Gradient Boosting, XGBoost để huấn luyện mô hình dự đoán thời gian vận chuyển. 
- Tối ưu hóa và điều chỉnh siêu tham số của mô hình để đạt được độ chính xác cao nhất.
- Đánh giá mô hình
Sử dụng các chỉ số đánh giá như MSE, RMSE, MAE, R2 để đánh giá hiệu suất của mô hình.
![độ đo đánh giá](kq.JPG)
- So sánh giá trị dự đoán và giá trị thực tế để kiểm tra độ chính xác của mô hình.
![so sánh ](ss1.JPG)


![so sánh dự đoán- thực tế](ss.png)
- Lưu và thử nghiệm mô hình với các bộ dữ liệu khác nhau
## Xây dựng giao diện
- Khởi tạo mô hình đã được lưu ở phần xây dựng mô hình
- Tính khoảng cách: Sử dụng API OSRM để tính khoảng cách giữa 2 tọa độ
- Xử lý và mã hóa dữ liệu đầu vào
- Sử dụng thư viện folium để tạo và hiển thị bản đồ
- Xây dựng giao diện bằng tkinter với các ô nhập liệu và nút chức năng.

![so sánh ](gd.JPG)
- Nút “Chọn vị trí trên bản đồ” sẽ hiển thị ra bản đồ để người dùng nhập tọa độ cửa hàng và nhà hàng.
 Khi người dùng click chuột vào vị trí xác định, nó sẽ hiển thị kinh độ và vĩ độ của điểm đang chọn. Sau đó ta sẽ điền vào các ô tương ứng

 ![bd](bd2.JPG)

 **Dự đoán thời gian từ SVĐ  Nehru -> Hồ Kumaraswamy**
- Với điều kiện giao hàng tốt: thời tiết tốt, không tắc đường, thời gian hết 25.52’
 ![dd1](dd1.JPG)



 - Với điều kiện xấu: mưa bão, tắc đường, thời gian sẽ lâu hơn: 33.34’
  ![dd2](dd2.JPG)


## Kết Luận
- Kết quả cho thấy rằng mô hình XGBoost đã đạt được hiệu suất tốt trong việc dự đoán thời gian giao hàng. Mặc dù mỗi mô hình đều có ưu điểm và hạn chế riêng

- Thông qua quá trình này, ta đã xác định được các yếu tố quan trọng ảnh hưởng đến thời gian giao hàng và áp dụng các phương pháp phân tích dữ liệu phù hợp để cải thiện hiệu suất dự đoán. Tuy nhiên, vẫn còn nhiều cơ hội để cải thiện mô hình, bao gồm việc tối ưu hóa các tham số, mở rộng phạm vi dữ liệu và thử nghiệm các phương pháp tiên tiến hơn.


