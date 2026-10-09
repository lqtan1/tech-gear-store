# TÀI LIỆU KỸ THUẬT AI PHÍA FRONTEND (CLIENT APPLICATION)
Công nghệ sử dụng: React 18, Vite, Tailwind CSS, Lucide React, Axios, Supabase Client SDK, React Router v6.

---

## 1. Cấu trúc thư mục dự án Client

src/
├── api/
│   ├── axiosClient.js          # Instance Axios cấu hình sẵn BaseURL và Interceptors
│   ├── productApi.js           # Các hàm gọi API sản phẩm, danh mục, tìm kiếm
│   ├── orderApi.js             # API tạo đơn hàng, lấy lịch sử đơn mua
│   └── paymentApi.js           # API khởi tạo link VNPAY
├── assets/                     # Tài nguyên ảnh tĩnh, favicon, logo
├── components/
│   ├── common/                 # Header, Footer, LoadingSpinner, Modal
│   ├── product/                # ProductCard, ProductGrid, FilterSidebar, SpecsTable
│   └── cart/                   # CartItemRow, CartSummary
├── context/
│   ├── AuthContext.jsx         # Quản lý trạng thái đăng nhập từ Supabase
│   └── CartContext.jsx         # Quản lý giỏ hàng cục bộ và đồng bộ máy chủ
├── hooks/
│   ├── useDebounce.js          # Hook hỗ trợ tìm kiếm không spam request
│   └── useCart.js              # Hook rút gọn truy xuất CartContext
├── pages/
│   ├── HomePage.jsx            # Trang chủ
│   ├── ProductDetailPage.jsx   # Chi tiết sản phẩm
│   ├── CartPage.jsx            # Xem giỏ hàng
│   ├── CheckoutPage.jsx        # Điền thông tin giao hàng & thanh toán
│   ├── PaymentReturnPage.jsx   # Nhận kết quả trả về từ VNPAY
│   └── ProfilePage.jsx         # Thông tin người dùng & đơn hàng đã đặt
├── routes/
│   └── AppRoutes.jsx           # Cấu hình định tuyến và Protected Routes
├── App.jsx
└── main.jsx

---

## 2. Quản lý trạng thái Xác thực (Authentication Flow)
Hệ thống sử dụng @supabase/supabase-js để duy trì phiên người dùng:

// src/context/AuthContext.jsx
import React, { createContext, useContext, useEffect, useState } from 'react';
import { createClient } from '@supabase/supabase-js';

const supabase = createClient(
  import.meta.env.VITE_SUPABASE_URL,
  import.meta.env.VITE_SUPABASE_ANON_KEY
);

const AuthContext = createContext();

export const AuthProvider = ({ children }) => {
  const [user, setUser] = useState(null);
  const [token, setToken] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    supabase.auth.getSession().then(({ data: { session } }) => {
      setUser(session?.user ?? null);
      setToken(session?.access_token ?? null);
      setLoading(false);
    });

    const { data: { subscription } } = supabase.auth.onAuthStateChange((_event, session) => {
      setUser(session?.user ?? null);
      setToken(session?.access_token ?? null);
      setLoading(false);
    });

    return () => subscription.unsubscribe();
  }, []);

  return (
    <AuthContext.Provider loading, supabase token, user, value="{{" }}>
      {!loading && children}
    </AuthContext.Provider>
  );
};

export const useAuth = () => useContext(AuthContext);

---

## 3. Cấu hình Axios Interceptors
Mọi request gửi tới Backend Express đều được tự động gắn mã xác thực Bearer Token lấy từ phiên đăng nhập:

// src/api/axiosClient.js
import axios from 'axios';
import { createClient } from '@supabase/supabase-js';

const supabase = createClient(
  import.meta.env.VITE_SUPABASE_URL,
  import.meta.env.VITE_SUPABASE_ANON_KEY
);

const axiosClient = axios.create({
  baseURL: import.meta.env.VITE_API_BASE_URL || 'http://localhost:5000/api',
  headers: {
    'Content-Type': 'application/json',
  },
});

axiosClient.interceptors.request.use(async (config) => {
  const { data: { session } } = await supabase.auth.getSession();
  if (session?.access_token) {
    config.headers.Authorization = `Bearer ${session.access_token}`;
  }
  return config;
}, (error) => Promise.reject(error));

axiosClient.interceptors.response.use(
  (response) => response.data,
  (error) => {
    const message = error.response?.data?.message || 'Có lỗi xảy ra, vui lòng thử lại';
    return Promise.reject(new Error(message));
  }
);

export default axiosClient;

---

## 4. Xử lý Quy trình Thanh toán VNPAY phía Client
Khi khách hàng xác nhận đơn hàng trên màn hình CheckoutPage:

1. Client gọi API POST /api/orders để lưu thông tin đơn hàng vào cơ sở dữ liệu.
2. Nếu đơn hàng chọn phương thức VNPAY:
   - Client gọi tiếp API POST /api/payments/create-url với tham số { orderCode, amount }.
   - Server trả về trường paymentUrl.
   - Client thực thi lệnh chuyển hướng trình duyệt: window.location.href = res.paymentUrl;
3. Sau khi hoàn tất giao dịch trên cổng VNPAY, người dùng được điều hướng về route /payment-return của Frontend kèm theo các tham số truy vấn (Query Params: vnp_ResponseCode, vnp_TxnRef, v.v.).
4. Component PaymentReturnPage.jsx đọc các tham số này để thông báo kết quả:
   - Nếu vnp_ResponseCode === '00': Hiển thị thông báo "Thanh toán thành công" và nút chuyển tới trang xem đơn mua.
   - Ngược lại: Hiển thị thông báo "Giao dịch không thành công hoặc đã bị hủy" và nút thử thanh toán lại.