# TECHGEAR STORE - HỆ THỐNG BÁN THIẾT BỊ NGOẠI VI & PHỤ KIỆN CÔNG NGHỆ

Hệ thống thương mại điện tử chuyên cung cấp thiết bị ngoại vi máy tính (bàn phím cơ, chuột gaming, màn hình chuyên dụng, tai nghe) tích hợp nền tảng đám mây Supabase (PostgreSQL, Auth, Storage) và cổng thanh toán trực tuyến VNPAY Sandbox.

---

## 1. Thành viên thực hiện & Thông tin đề tài
* **Sinh viên thực hiện:** Lê Quang Tấn
* **Đơn vị:** Khoa Công nghệ Thông tin - Trường Đại học Phú Xuân (PXU)
* **Tên đề tài:** Xây dựng website bán thiết bị ngoại vi và phụ kiện công nghệ tích hợp cổng thanh toán trực tuyến

---

## 2. Danh mục tài liệu kỹ thuật (Documentation)
Toàn bộ tài liệu phân tích nghiệp vụ, đặc tả giao diện và thiết kế kỹ thuật được lưu trữ trong thư mục `docs/`:

1. [01. Tài liệu Yêu cầu Nghiệp vụ (BRD)](./docs/01_BUSINESS_REQUIREMENTS.md)
   * Giới thiệu bài toán, phân tích các tác nhân (Actors).
   * Danh sách 5 phân hệ nghiệp vụ chính (Auth, Catalog, Cart & Orders, Payment, Admin).
   * Các quy tắc nghiệp vụ (Business Rules).

2. [02. Tài liệu Đặc tả Giao diện & Prompt AI (AI UI/UX Spec)](./docs/02_AI_UI_UX_SPECIFICATION.md)
   * Hệ thống quy chuẩn thiết kế (Design Tokens, màu sắc Cyberpunk/Dark Tech).
   * Đặc tả bố cục màn hình (Trang chủ, Lưới sản phẩm, Trang chi tiết PDP, Trang thanh toán).
   * Bộ System Prompts tiêu chuẩn để điều khiển AI sinh mã giao diện.

3. [03. Tài liệu Kỹ thuật AI phía Frontend (Client Spec)](./docs/03_AI_FRONTEND_TECHNICAL_SPEC.md)
   * Cấu trúc thư mục mã nguồn React 18 + Vite + Tailwind CSS.
   * Xử lý luồng phiên xác thực với Supabase Client SDK.
   * Cấu hình Axios Interceptors gắn tự động Bearer Token.
   * Luồng tích hợp thanh toán và tiếp nhận kết quả chuyển hướng từ VNPAY.

4. [04. Tài liệu Kỹ thuật AI phía Admin & Backend (Backend Spec)](./docs/04_AI_ADMIN_TECHNICAL_SPEC.md)
   * Cơ chế phân quyền và bảo vệ route qua Middleware Admin Guard.
   * Xử lý tải ảnh sản phẩm lên Supabase Storage bucket.
   * Kiến trúc xác thực chữ ký số HMAC-SHA512 qua Webhook IPN của VNPAY.
   * Các endpoint thống kê và phân tích số liệu báo cáo bán hàng.

---

## 3. Công nghệ sử dụng (Tech Stack)

### Frontend (Client)
* **Framework:** React 18 (Vite build tool)
* **UI & Styling:** Tailwind CSS, Lucide Icons
* **State & Data Fetching:** React Context API, Axios

### Backend (Server)
* **Runtime:** Node.js (Express Framework)
* **Security & Utility:** JSON Web Token, Node Crypto (HMAC-SHA512), Dotenv, Multer

### Database & Cloud Platform (Supabase)
* **Database:** PostgreSQL (Hỗ trợ JSONB lưu thông số kỹ thuật động, Database Triggers)
* **Authentication:** Supabase Auth (Quản lý User Session, JWT)
* **Object Storage:** Supabase Storage (Lưu trữ hình ảnh sản phẩm)

### Cổng thanh toán (Payment Gateway)
* **Cổng tích hợp:** VNPAY Sandbox (Thanh toán qua VNPAY-QR, Thẻ ATM/Tài khoản ngân hàng, Thẻ quốc tế)

---

## 4. Cấu trúc thư mục dự án (Project Structure)

tech-gear-store/
├── docs/                                 # Thư mục chứa 4 file tài liệu nộp bài
│   ├── 01_BUSINESS_REQUIREMENTS.md
│   ├── 02_AI_UI_UX_SPECIFICATION.md
│   ├── 03_AI_FRONTEND_TECHNICAL_SPEC.md
│   └── 04_AI_