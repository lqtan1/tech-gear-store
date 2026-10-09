# TÀI LIỆU YÊU CẦU NGHIỆP VỤ (BUSINESS REQUIREMENTS DOCUMENT - BRD)
**Tên đề tài:** Hệ thống Website Bán Thiết Bị Ngoại Vi & Phụ Kiện Công Nghệ Tích Hợp Cổng Thanh Toán Trực Tuyến  
**Tên thương mại:** TechGear Store  
**Nền tảng triển khai:** Web Application (Responsive)  

---

## 1. Tổng quan dự án
Hệ thống là sàn bán lẻ trực tuyến chuyên biệt về các thiết bị ngoại vi máy tính (bàn phím cơ, chuột gaming, màn hình chuyên dụng, tai nghe cao cấp và phụ kiện setup). Hệ thống giải quyết các bài toán mua sắm đặc thù:
* Cho phép người dùng lọc sản phẩm theo thông số kỹ thuật chuyên sâu (loại switch bàn phím, kết nối không dây/có dây, cảm biến DPI, độ phân giải màn hình).
* Quản lý giỏ hàng và đặt hàng trực tuyến với độ trễ thấp.
* Tích hợp thanh toán trực tuyến bảo mật qua cổng VNPAY Sandbox nhằm tối ưu hóa trải nghiệm không tiền mặt.
* Cung cấp trang quản trị vận hành đơn hàng, quản lý kho biến thể và phân tích doanh thu.

---

## 2. Các nhóm tác nhân (Actors)
1. **Khách vãng lai (Guest):**
   * Người dùng chưa xác thực danh tính.
   * Quyền hạn: Duyệt danh mục sản phẩm, tìm kiếm, xem thông số chi tiết, thêm sản phẩm vào giỏ hàng tạm thời trên trình duyệt.
2. **Khách hàng (Customer):**
   * Người dùng đã đăng ký/đăng nhập qua hệ thống xác thực.
   * Quyền hạn: Đồng bộ giỏ hàng lên máy chủ, quản lý sổ địa chỉ giao hàng, đặt hàng, thanh toán qua cổng VNPAY/COD, tra cứu lịch sử mua hàng và cập nhật hồ sơ cá nhân.
3. **Quản trị viên (Admin):**
   * Nhân sự quản lý hệ thống có quyền truy cập trang quản trị.
   * Quyền hạn: Quản lý danh mục, tạo mới và cập nhật sản phẩm, kiểm soát số lượng hàng tồn, điều phối trạng thái đơn hàng, theo dõi báo cáo doanh thu.

---

## 3. Danh sách ca sử dụng nghiệp vụ chi tiết (Business Use Cases)

### Phân hệ 1: Xác thực & Hồ sơ người dùng (Authentication & Identity)
* **UC-AUTH-01 (Đăng ký tài khoản):** Khách hàng đăng ký bằng Email và Mật khẩu. Hệ thống tự động tạo bản ghi người dùng trên Supabase Auth và kích hoạt database trigger để tạo hồ sơ khách hàng tại bảng `public.profiles`.
* **UC-AUTH-02 (Đăng nhập & Phiên làm việc):** Kiểm tra thông tin đăng nhập, cấp phát mã phiên JWT (JSON Web Token) cho Client để duy trì đăng nhập.
* **UC-AUTH-03 (Quản lý hồ sơ cá nhân):** Khách hàng cập nhật họ tên, số điện thoại, địa chỉ nhận hàng mặc định.

### Phân hệ 2: Danh mục & Sản phẩm (Catalog & Products)
* **UC-CAT-01 (Duyệt & Phân loại):** Khách hàng xem sản phẩm theo từng nhóm danh mục (Keyboards, Mice, Monitors, Headsets).
* **UC-CAT-02 (Bộ lọc thông số kỹ thuật):** Hỗ trợ tìm kiếm theo từ khóa và lọc đa chiều theo mức giá, thương hiệu, và cấu hình phần cứng (Switch, Kết nối, DPI) được lưu trữ theo định dạng JSONB.
* **UC-CAT-03 (Xem chi tiết sản phẩm):** Hiển thị bộ ảnh sản phẩm độ phân giải cao, giá bán, giá niêm yết, tình trạng tồn kho, chính sách bảo hành và bảng thông số kỹ thuật.

### Phân hệ 3: Giỏ hàng & Quy trình Đặt hàng (Cart & Order Processing)
* **UC-ORD-01 (Quản lý giỏ hàng):** Thêm sản phẩm, cập nhật số lượng, xóa từng món hoặc làm trống toàn bộ giỏ. Kiểm tra số lượng tồn kho theo thời gian thực để ngăn chặn đặt vượt số lượng có sẵn.
* **UC-ORD-02 (Tạo đơn hàng):** Khách hàng nhập thông tin nhận hàng, chọn phương thức thanh toán (COD hoặc VNPAY). Hệ thống xác thực tính khả dụng của hàng tồn, tạo đơn hàng ở trạng thái `pending` với mã định danh duy nhất (ví dụ: `ORD1712345678`).
* **UC-ORD-03 (Trừ tồn kho tự động):** Khi đơn hàng được ghi nhận thanh toán thành công, hệ thống tự động trừ số lượng sản phẩm trong kho tương ứng.
* **UC-ORD-04 (Hủy đơn hàng):** Cho phép khách hàng hủy đơn hàng khi trạng thái vẫn đang ở mức `pending` hoặc `processing`, tự động hoàn lại số lượng tồn kho.

### Phân hệ 4: Thanh toán trực tuyến (Payment Integration)
* **UC-PAY-01 (Khởi tạo giao dịch VNPAY):** Tạo liên kết chuyển hướng thanh toán kèm theo mã băm bảo mật checksum HMAC-SHA512.
* **UC-PAY-02 (Xử lý kết quả trả về - Return URL):** Tiếp nhận phản hồi điều hướng của khách hàng từ cổng VNPAY về frontend để hiển thị trạng thái giao dịch (thành công/thất bại).
* **UC-PAY-03 (Đối soát Webhook tự động - IPN):** Server tiếp nhận dữ liệu ngầm từ server VNPAY, kiểm tra tính toàn vẹn chữ ký và tự động chuyển đổi `payment_status` sang `paid`.

### Phân hệ 5: Quản trị hệ thống (Admin Management)
* **UC-ADM-01 (Quản lý sản phẩm):** Thêm, sửa, xóa hoặc ẩn sản phẩm (Soft Delete) không còn kinh doanh.
* **UC-ADM-02 (Quản lý vận đơn):** Chuyển đổi trạng thái đơn hàng theo chu trình: `pending` -> `processing` -> `shipping` -> `completed` / `cancelled`.
* **UC-ADM-03 (Báo cáo kinh doanh):** Thống kê tổng doanh thu theo ngày/tháng, tỷ lệ đơn hàng thành công và danh sách thiết bị bán chạy nhất.

---

## 4. Quy tắc nghiệp vụ (Business Rules)
* **BR-01 (Kiểm soát giá):** Giá bán khuyến mãi (`price`) luôn luôn phải nhỏ hơn hoặc bằng giá gốc niêm yết (`original_price`).
* **BR-02 (Ràng buộc kho):** Khách hàng không thể thêm vào giỏ hàng hoặc đặt hàng số lượng lớn hơn giá trị `stock` hiện có.
* **BR-03 (Toàn vẹn thanh toán):** Trạng thái thanh toán của đơn hàng chỉ chuyển sang `paid` khi chữ ký bảo mật SHA512 gửi qua IPN trùng khớp tuyệt đối với khóa bí mật lưu trên server.