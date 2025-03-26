# Phân tích hành vi đặt vé xem phim của khách hàng
# Phân cụm Khách hàng Sử dụng K-Means

## 1. Giới thiệu
Dự án này nhằm mục tiêu **phân cụm khách hàng** dựa trên dữ liệu giao dịch và thông tin khách hàng từ hệ thống bán vé xem phim. Qua đó, chúng ta có thể nhận diện các nhóm khách hàng khác nhau nhằm đề xuất các chiến lược marketing phù hợp. 

Các bước chính trong dự án:
- Xử lý dữ liệu: Chuẩn hóa các đặc trưng số và mã hóa các biến phân loại.
- Áp dụng thuật toán K-Means để phân cụm khách hàng.
- Sử dụng Autoencoder để giảm chiều dữ liệu và cải thiện chất lượng phân cụm.
- Đánh giá phân cụm qua Silhouette Score và trực quan hóa bằng PCA.
- Phân tích đặc trưng của từng cụm và gợi ý chiến lược marketing cho từng nhóm khách hàng.

## 2. Dữ liệu
- **Nguồn dữ liệu**: Dữ liệu giao dịch từ hệ thống bán vé (file `df_join_all`).
- **Các biến chính**:
  - Thông tin giao dịch: `original_price`, `discount_value`, `final_price`
  - Thông tin khách hàng: `age`, `dob`, `customer_id`
  - Thông tin giao dịch khác: `paying_method`, `theater_name`, `movie_name`, `platform`, `time`, `status_id`, `campaign_id`, v.v.
  
Ngoài ra, dữ liệu còn chứa các biến liên quan đến thời gian như `month`, `year_month`, `name_day`, `hour` và thông tin hệ thống như `os_version`, `type`.

## 3. Quy trình thực hiện

### 3.1. Xử lý dữ liệu
- **Tiền xử lý số**:  
  - Lấy các đặc trưng số như `original_price`, `discount_value`, `final_price`, `age` và chuẩn hóa bằng StandardScaler.
  
- **Xử lý dữ liệu phân loại**:  
  - Mã hóa các biến phân loại như `paying_method`, `theater_name`, `movie_name`, `platform`, `usergender`, `campaign_type` sử dụng OneHotEncoder.
  
- **Kết hợp đặc trưng**:  
  - Gộp các đặc trưng đã xử lý (số và phân loại) lại thành một DataFrame để làm đầu vào cho thuật toán phân cụm.

### 3.2. Phân cụm với K-Means
- Áp dụng thuật toán K-Means với số cụm được chọn (ví dụ: k = 3).
- Tính toán Inertia theo nhiều giá trị k và sử dụng phương pháp Elbow để lựa chọn số cụm tối ưu.
- Gán nhãn phân cụm cho từng mẫu trong dữ liệu gốc.

### 3.3. Giảm chiều dữ liệu bằng Autoencoder
- Xây dựng và huấn luyện Autoencoder (với 10 epoch) nhằm giảm chiều dữ liệu ban đầu.
- Sử dụng phần mã hóa (encoder) để trích xuất đặc trưng mới, sau đó áp dụng K-Means trên dữ liệu giảm chiều.
- Tính Silhouette Score để đánh giá chất lượng phân cụm.

### 3.4. Trực quan hóa phân cụm
- Áp dụng PCA để chuyển đổi dữ liệu giảm chiều thành 2 chiều và vẽ biểu đồ scatter với màu sắc thể hiện nhãn cụm.
- Vẽ các biểu đồ boxplot, countplot và barplot để phân tích đặc điểm của từng cụm theo các đặc trưng như `age`, `final_price`, `status_id`, `campaign_id`.

### 3.5. Phân tích và Chiến lược Marketing
- **Cụm 0: Khách hàng trung thành & được ưu tiên marketing**
  - Đặc điểm: Giao dịch cao, được tiếp cận nhiều chiến dịch marketing, độ tuổi trung niên.
  - Chiến lược: Triển khai chương trình khách hàng thân thiết, ưu đãi VIP, cá nhân hóa quảng cáo.
  
- **Cụm 1: Khách hàng trẻ & tiềm năng nhưng ít giao dịch**
  - Đặc điểm: Độ tuổi trẻ nhất, nhận nhiều chiến dịch quảng cáo nhưng giao dịch thấp.
  - Chiến lược: Tổ chức Flash Sale, khuyến mãi lần đầu, áp dụng gamification và tối ưu quảng cáo trên mạng xã hội.
  
- **Cụm 2: Khách hàng mua sắm tự nhiên, ít tiếp cận marketing**
  - Đặc điểm: Giao dịch cao, ít được quảng cáo, nhưng có hành vi mua sắm ổn định.
  - Chiến lược: Tăng cường chiến lược remarketing, ưu đãi dựa trên sở thích, và chăm sóc khách hàng bằng dịch vụ hậu mãi.

## 4. Kết quả
- **Số lượng mẫu phân cụm**:  
  - Cụm 0: ~83,577 mẫu  
  - Cụm 1: ~42,688 mẫu  
  - Cụm 2: ~28,460 mẫu  
- **Silhouette Score sau khi giảm chiều**: 0.5319 (cho thấy phân cụm đạt mức ổn định)
- **Trực quan hóa PCA**: Biểu đồ scatter thể hiện các cụm được phân chia rõ ràng.
- Các biểu đồ thống kê cho thấy sự khác biệt về tuổi, giá trị giao dịch và chiến dịch marketing giữa các cụm.

## 5. Hướng phát triển thêm
- Tinh chỉnh số cụm bằng cách thử nghiệm với các giá trị k khác nhau.
- Áp dụng các thuật toán phân cụm khác (như Hierarchical Clustering) để so sánh kết quả.
- Mở rộng phân tích với các đặc trưng khác hoặc kết hợp thêm dữ liệu mới để làm giàu thông tin.
- Triển khai dashboard trực quan để hỗ trợ chiến lược marketing theo thời gian.

## 6. Cách chạy Dự án
1. **Clone Repository**:
   ```bash
   git clone https://github.com/your_username/customer-clustering.git
   cd customer-clustering
   ```
2. **Mở Notebook**:  
   Chạy file notebook (ví dụ: `customer_clustering.ipynb`) trong Google Colab hoặc Jupyter Notebook.
3. **Cập nhật Đường dẫn**:  
   Đảm bảo đường dẫn đến dữ liệu (`df_join_all`) là chính xác.
4. **Chạy toàn bộ các cell** theo thứ tự: Tiền xử lý dữ liệu → Phân cụm (K-Means) → Giảm chiều (Autoencoder) → Trực quan hóa → Phân tích và gán nhãn chiến lược.

## 7. Yêu cầu Cài đặt
- Python 3.7+
- Các thư viện: pandas, numpy, matplotlib, seaborn, scikit-learn, tensorflow (cho Autoencoder), và các thư viện khác theo yêu cầu.

## 8. Tài liệu Tham khảo
- [Scikit-learn Documentation](https://scikit-learn.org/)
- [TensorFlow Keras Documentation](https://www.tensorflow.org/guide/keras)
- [Pandas Documentation](https://pandas.pydata.org/)
- [PCA and K-Means clustering](https://towardsdatascience.com/)

## 9. Liên hệ
Nếu có thắc mắc hoặc góp ý, vui lòng mở issue trên GitHub hoặc liên hệ qua email: thanh.van19062004@gmail.com
```

---

### Hướng dẫn sử dụng README này
1. Tạo file `README.md` trong repository của bạn.
2. Sao chép và dán nội dung trên vào file.
3. Điều chỉnh các thông tin (đường dẫn, email, tên repo, …) cho phù hợp với dự án của bạn.
4. Commit và push file `README.md` lên GitHub.

