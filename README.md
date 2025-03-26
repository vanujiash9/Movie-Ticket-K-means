
# Phân tích Dữ liệu Bán vé Phim & Phân cụm Khách hàng

## 1. Tổng quan

Dự án này nhằm phân tích sâu dữ liệu bán vé phim để hiểu rõ hành vi mua vé của khách hàng, nhận diện các xu hướng mua vé theo thời gian và xác định các yếu tố tác động đến quyết định mua hàng. Ngoài ra, dự án còn thực hiện **phân cụm khách hàng** bằng thuật toán K-Means (có kết hợp giảm chiều bằng Autoencoder) nhằm xác định các nhóm khách hàng có đặc điểm mua sắm khác nhau, từ đó đề xuất chiến lược marketing phù hợp.

## 2. Dữ liệu

Dữ liệu sử dụng bao gồm nhiều tệp CSV với các nội dung khác nhau:
- **campaign.csv**: Thông tin về các chiến dịch marketing.
- **customer.csv**: Hồ sơ khách hàng, bao gồm ngày sinh, giới tính, …
- **device_detail.csv**: Thông tin về các thiết bị khách hàng sử dụng (model, hệ điều hành,…).
- **status_detail.csv**: Trạng thái giao dịch (thành công, thất bại, …).
- **ticket_history.csv**: Lịch sử giao dịch bán vé, bao gồm thông tin giao dịch, giá vé, khuyến mãi, thời gian mua vé, …
  
Các tệp này được kết hợp qua các cột khóa chung (như `customer_id`, `campaign_id`, `device_number`, `status_id`) để tạo thành một DataFrame duy nhất (`df_join_all`) chứa đầy đủ thông tin cần thiết cho các phân tích.

## 3. Quy trình Phân tích Dữ liệu

### 3.1. Tải và Làm sạch Dữ liệu
- **Đọc dữ liệu**: Nạp các tệp CSV vào DataFrame sử dụng pandas.
- **Kiểm tra dữ liệu**: Xem thông tin tổng quan (df.info(), df.describe()), kiểm tra giá trị NULL và trùng lặp.
- **Chuyển đổi kiểu dữ liệu**: Chuyển đổi các cột ngày tháng (như `time`, `dob`) sang định dạng datetime.
- **Xử lý giá trị NULL**: Điền giá trị "unknow" hoặc loại bỏ các bản ghi thiếu thông tin quan trọng (ví dụ: `device_number`).
- **Loại bỏ dữ liệu trùng lặp**: Đảm bảo mỗi giao dịch và khách hàng chỉ xuất hiện một lần.

### 3.2. Kết hợp Các Bảng Dữ liệu
- **Kết hợp dữ liệu**: Thực hiện các phép join (theo phương pháp left join) giữa:
  - `ticket_history` và `customer` (theo `customer_id`).
  - Kết quả trên kết hợp với `campaign` (theo `campaign_id`).
  - Sau đó kết hợp với `device_detail` (theo `device_number`).
  - Cuối cùng kết hợp với `status_detail` (theo `status_id`).
- **Xử lý giá trị NULL sau join**: Điền giá trị 'unknow' nếu cần để đảm bảo tính nhất quán.

### 3.3. Phân tích Dữ liệu
- **Chân dung Khách hàng**:  
  - Phân bố độ tuổi, giới tính và phân nhóm theo thế hệ.
  - Trực quan hóa số liệu bằng histogram, boxplot và pie chart.
- **Phân tích chuỗi thời gian**:  
  - Thống kê số lượng vé bán ra theo tháng, theo ngày trong tuần và theo giờ.
  - Sử dụng biểu đồ miền (area chart) và biểu đồ đường để nắm bắt xu hướng mua vé theo thời gian.
- **Yếu tố ảnh hưởng đến quyết định mua hàng**:  
  - Phân tích nền tảng thanh toán, phiên bản hệ điều hành, phương thức thanh toán và khuyến mãi.
  - Trực quan hóa bằng biểu đồ cột, biểu đồ tròn và biểu đồ 100% area chart.
- **Giá trị khách hàng**:  
  - Tính các chỉ số như tần suất mua, tổng giá trị giao dịch, tỷ lệ thành công giao dịch, v.v.
  - Phân phối giá trị khách hàng được trực quan hóa bằng histogram.
- **Hành vi mua hàng**:  
  - Phân loại khách hàng theo số lần mua vé, xác định nhóm có hành vi bất thường.
- **Phân tích Cohort**:  
  - Nhóm khách hàng theo tháng mua hàng đầu tiên và tính tỷ lệ giữ chân khách hàng theo thời gian.
- **Phân tích thành công thanh toán**:  
  - Tính toán tỷ lệ giao dịch thành công, trực quan hóa xu hướng thành công và lỗi giao dịch theo thời gian.

## 4. Phân cụm Khách hàng với K-Means

### 4.1. Xử lý đặc trưng
- **Đặc trưng số**: Chọn các cột số như `original_price`, `discount_value`, `final_price`, `age` và chuẩn hóa bằng StandardScaler.
- **Đặc trưng phân loại**: Mã hóa các biến phân loại như `paying_method`, `theater_name`, `movie_name`, `platform`, `usergender`, `campaign_type` bằng OneHotEncoder.
- **Kết hợp**: Gộp các đặc trưng số và phân loại thành một DataFrame (`df_features`) dùng làm đầu vào cho thuật toán K-Means.

### 4.2. Phân cụm với K-Means
- Sử dụng thuật toán K-Means để phân cụm khách hàng.  
- **Phương pháp Elbow**: Tính Inertia cho nhiều giá trị k (1 đến 9) và chọn số cụm tối ưu (ví dụ: k = 3).
- Gán nhãn phân cụm vào DataFrame ban đầu và tính Silhouette Score (ví dụ: 0.5319).

### 4.3. Giảm chiều và Trực quan hóa
- **Giảm chiều**: Sử dụng Autoencoder để giảm số chiều của dữ liệu và sau đó áp dụng K-Means trên dữ liệu giảm chiều.
- **Trực quan hóa**:  
  - Sử dụng PCA để chuyển đổi dữ liệu giảm chiều thành 2D.
  - Vẽ scatter plot với màu sắc biểu diễn nhãn cụm.
- **Phân tích đặc trưng theo cụm**:  
  - Vẽ các biểu đồ boxplot, countplot và barplot để so sánh các chỉ số (tuổi, final_price, campaign_id, …) giữa các cụm.
  - Gán nhãn phân cụm thành các nhóm khách hàng có ý nghĩa (ví dụ: “Khách hàng trung thành & được ưu tiên marketing”, “Khách hàng trẻ & tiềm năng nhưng ít giao dịch”, “Khách hàng mua sắm tự nhiên, ít tiếp cận marketing”).

## 5. Kết quả & Chiến lược Marketing

### 5.1. Phân tích Đặc điểm Khách hàng
- **Theo độ tuổi**:  
  - Cụm 1 là nhóm khách hàng trẻ nhất, cụm 0 và 2 có khách hàng trung niên.
- **Theo giao dịch**:  
  - Cụm 0 có số lượng giao dịch cao và được tiếp cận nhiều chiến dịch marketing.
  - Cụm 2 có giao dịch cao mặc dù ít quảng cáo, cho thấy tiềm năng mua sắm tự nhiên.
  - Cụm 1 có giao dịch thấp, cần chiến lược kích thích thêm.

### 5.2. Chiến lược Marketing cho từng cụm
- **Cụm 0: Khách hàng trung thành & được ưu tiên marketing**  
  - Chiến lược: Chương trình khách hàng thân thiết, ưu đãi VIP, cá nhân hóa quảng cáo.
- **Cụm 1: Khách hàng trẻ & tiềm năng nhưng ít giao dịch**  
  - Chiến lược: Flash Sale, khuyến mãi lần đầu, gamification, tối ưu quảng cáo trên mạng xã hội.
- **Cụm 2: Khách hàng mua sắm tự nhiên, ít tiếp cận marketing**  
  - Chiến lược: Remarketing, ưu đãi cá nhân hóa, chăm sóc khách hàng bằng dịch vụ hậu mãi.

## 6. Cách chạy Dự án

1. **Clone Repository**:
   ```bash
   git clone https://github.com/your_username/movie-ticket-analysis.git
   cd movie-ticket-analysis
   ```
2. **Kết nối với Google Drive** (nếu sử dụng Google Colab):
   ```python
   from google.colab import drive
   drive.mount('/content/drive')
   ```
3. **Cập nhật Đường dẫn**:  
   Điều chỉnh đường dẫn trong script Python cho phù hợp với vị trí tệp dữ liệu của bạn (ví dụ: `campaign.csv`, `customer.csv`, …).
4. **Chạy toàn bộ Notebook**:  
   Mở file notebook (ví dụ: `movie_ticket_analysis.ipynb`) trong Google Colab hoặc Jupyter Notebook và chạy các cell theo thứ tự từ xử lý dữ liệu, kết hợp các bảng, phân tích đa chiều, đến phân cụm khách hàng và trực quan hóa kết quả.

## 7. Yêu cầu Cài đặt

- Python 3.7+
- Các thư viện: pandas, numpy, matplotlib, seaborn, scikit-learn, tensorflow (nếu dùng Autoencoder), và các thư viện cần thiết khác.

## 8. Tài liệu Tham khảo

- [Pandas Documentation](https://pandas.pydata.org/)
- [Scikit-learn Documentation](https://scikit-learn.org/)
- [TensorFlow Keras Documentation](https://www.tensorflow.org/guide/keras)
- [Các bài viết về phân cụm và phân tích dữ liệu bán vé](https://towardsdatascience.com/)

## 9. Liên hệ

Nếu có bất kỳ thắc mắc hoặc góp ý nào, vui lòng mở issue trên GitHub hoặc liên hệ qua email: thanh.van19062004@gmail.com
```

---

### Hướng dẫn sử dụng README này

1. Tạo file `README.md` trong repository của bạn.
2. Sao chép và dán nội dung trên vào file.
3. Điều chỉnh các thông tin (đường dẫn, email, tên repo, …) cho phù hợp với dự án của bạn.
4. Commit và push file `README.md` lên GitHub.

Mẫu README này cung cấp cái nhìn tổng quan về dự án phân tích dữ liệu bán vé phim, quy trình xử lý và kết hợp dữ liệu, các bước phân tích đa chiều và phân cụm khách hàng bằng K-Means, cũng như chiến lược marketing cho từng nhóm khách hàng.

