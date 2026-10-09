# TÀI LIỆU ĐẶC TẢ GIAO DIỆN & PROMPT THỰC THI CHO AI (AI UI/UX SPECIFICATION)
**Mục tiêu:** Cung cấp thông số thiết kế, cấu trúc giao diện và các bộ System Prompt chuẩn hóa để các công cụ AI (GitHub Copilot, Cursor, v.v.) tự động sinh mã nguồn giao diện chính xác.

---

## 1. Quy chuẩn Hệ thống Thiết kế (Design Tokens)
* **Phong cách thẩm mỹ:** Dark Tech / Cyberpunk Minimalist kết hợp Glassmorphism nhẹ.
* **Bảng màu chủ đạo:**
  * Background: Slate Dark (`#0b0f19`) cho toàn bộ nền, Slate Sub-surface (`#111827`) cho thẻ card/sidebar.
  * Primary Accent: Cyber Cyan (`#06b6d4`) - nút bấm chính, điểm nhấn hover, trạng thái active.
  * Secondary Accent: Electric Purple (`#8b5cf6`) - badge sản phẩm nổi bật, banner khuyến mãi.
  * Typography Color: Text Primary (`#f8fafc`), Text Muted (`#94a3b8`).
  * Trạng thái: Success (`#10b981`), Warning (`#f59e0b`), Error (`#ef4444`).
* **Typography:** Font chữ sans-serif không chân hiện đại (Inter, Roboto), font-weight từ 400 đến 700.

---

## 2. Đặc tả bố cục các màn hình chính (Wireframes & Screen Layouts)

### 2.1. Trang chủ & Danh mục sản phẩm (Home & Catalog View)
* **Header Bar (Sticky):**
  * Bên trái: Logo "TechGear", liên kết danh mục nhanh.
  * Ở giữa: Thanh tìm kiếm với hiệu ứng loading icon, debounce 300ms.
  * Bên phải: Nút chuyển đổi Dark/Light mode, Icon giỏ hàng kèm số đếm đỏ, Menu avatar tài khoản.
* **Khu vực lọc đa chiều (Filter Sidebar - 25% chiều rộng):**
  * Accordion 1: Danh mục thiết bị (Checkboxes: Bàn phím cơ, Chuột, Màn hình, Tai nghe).
  * Accordion 2: Khoảng giá (Dual Range Slider từ 0 VNĐ đến 20.000.000 VNĐ).
  * Accordion 3: Thông số kỹ thuật nhanh (Checkboxes: Không dây 2.4GHz, Bluetooth, Switch Hotswap, RGB).
* **Lưới sản phẩm (Product Grid - 75% chiều rộng):**
  * Bố cục 3-4 cột trên màn hình Desktop, 2 cột trên Tablet/Mobile.
  * Thẻ sản phẩm (Product Card):
    * Ảnh đại diện vuông tỉ lệ 1:1, hiệu ứng phóng to nhẹ khi hover (hover-zoom).
    * Tag hiển thị thương hiệu và giảm giá (nếu có).
    * Tên sản phẩm hiển thị tối đa 2 dòng, tự động cắt ngắn bằng dấu ba chấm (`line-clamp-2`).
    * Giá hiển thị màu cyan nổi bật; giá gốc gạch ngang màu xám mờ.
    * Nút "Thêm vào giỏ" nhanh với icon giỏ hàng và hiệu ứng chuyển màu.

### 2.2. Trang Chi tiết sản phẩm (Product Detail Page - PDP)
* **Bố cục chia 2 cột đối xứng (50% - 50%):**
  * **Cột trái:** Khung ảnh lớn với khả năng phóng to khi di chuột, kèm carousel thumbnail 4-5 góc chụp phía dưới.
  * **Cột phải:**
    * Tên sản phẩm, mã SKU, trạng thái còn hàng (badge xanh lá) / hết hàng (badge đỏ).
    * Bảng thông số kỹ thuật dạng thẻ table tinh gọn (Hãng, Switch, Kết nối, Trọng lượng, Bảo hành).
    * Bộ đếm số lượng với 2 nút tăng/giảm và ô nhập trực tiếp.
    * Cặp nút hành động: "Thêm vào giỏ hàng" (Outline Cyan) và "Mua ngay" (Solid Cyan).
* **Phần mô tả chi tiết phía dưới:** Tab chuyển đổi giữa "Mô tả sản phẩm" và "Chính sách đổi trả - bảo hành".

### 2.3. Trang Thanh toán (Checkout Flow)
* **Khối thông tin giao hàng (60%):** Form thu thập Họ tên, Số điện thoại, Tỉnh/Thành phố, Quận/Huyện, Địa chỉ nhà, Ghi chú giao hàng.
* **Khối phương thức & Tóm tắt (40%):**
  * Tùy chọn thanh toán dạng Radio Box:
    * Option 1: Thanh toán khi nhận hàng (COD).
    * Option 2: Cổng thanh toán trực tuyến VNPAY (Hiển thị logo VNPAY-QR, Thẻ ATM nội địa, Thẻ quốc tế).
  * Hộp tính tiền: Tạm tính, Phí vận chuyển, Giảm giá voucher, Tổng thanh toán cuối cùng.
  * Nút "Xác nhận & Tiến hành thanh toán" kích thước lớn, loading spinner khi đang kết nối API.

---

## 3. Bộ System Prompt chuẩn để AI sinh giao diện Frontend

### Prompt 1: Tạo Component Card Sản phẩm
```text
Bạn là một Senior Frontend Engineer chuyên về React và Tailwind CSS.
Hãy tạo một React Component có tên là `ProductCard.jsx` với các tiêu chí kỹ thuật sau:
1. Props nhận vào: product (gồm id, name, price, original_price, images, brand, stock, rating).
2. Thiết kế theo phong cách Dark Mode hiện đại: background slate-900, border slate-800, text slate-100.
3. Ảnh tỉ lệ 1:1, bo góc rounded-xl, có hiệu ứng zoom nhẹ (transition-transform group-hover:scale-105).
4. Hiển thị % giảm giá nếu original_price > price.
5. Định dạng tiền tệ theo định dạng chuẩn Việt Nam Đồng (VND).
6. Có nút "Thêm vào giỏ" hỗ trợ icon Lucide-react (ShoppingCart). Nếu stock = 0, vô hiệu hóa nút và hiển thị badge "Hết hàng".
7. Đảm bảo mã nguồn React sạch, hỗ trợ xử lý sự kiện click chuyển trang qua React Router Link.
```

### Prompt 2: Tạo Trang Giỏ hàng & Thanh toán
```text
Hãy viết trọn vẹn file React component `CheckoutPage.jsx` sử dụng Tailwind CSS và Axios:
1. Chia layout làm 2 cột: Cột trái là Form thông tin giao hàng có validation cơ bản; Cột phải là danh sách tóm tắt đơn hàng và lựa chọn phương thức thanh toán.
2. Cung cấp 2 lựa chọn radio: COD và VNPAY.
3. Khi người dùng bấm "Đặt hàng":
   - Nếu chọn COD: gọi API tạo đơn và điều hướng tới /order-success/:orderCode.
   - Nếu chọn VNPAY: gọi API lấy liên kết thanh toán và thực hiện chuyển hướng trình duyệt bằng window.location.href.
4. Xử lý toàn bộ các trạng thái: submitting (hiển thị spinner), disable nút để chống bấm đúp, và hiển thị thông báo lỗi nếu có lỗi mạng.
```