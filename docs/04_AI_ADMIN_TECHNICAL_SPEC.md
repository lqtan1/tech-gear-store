# TÀI LIỆU KỸ THUẬT AI PHÍA ADMIN (DASHBOARD ENGINE & BACKEND ARCHITECTURE)
Công nghệ sử dụng: Node.js, Express, Supabase PostgreSQL, Supabase Storage API, Node Crypto (HMAC-SHA512).

---

## 1. Cơ chế Xác thực & Phân quyền Quản trị (Admin Guard Middleware)
Bảo vệ toàn bộ các endpoint bắt đầu bằng /api/admin/* nhằm đảm bảo chỉ có tài khoản mang quyền admin mới được thực thi:

// src/middlewares/adminMiddleware.js
const supabase = require('../config/supabase');

const requireAdmin = async (req, res, next) => {
  try {
    const authHeader = req.headers.authorization;
    if (!authHeader || !authHeader.startsWith('Bearer ')) {
      return res.status(401).json({ success: false, message: 'Yêu cầu token xác thực' });
    }

    const token = authHeader.split(' ')[1];
    const { data: { user }, error: authError } = await supabase.auth.getUser(token);

    if (authError || !user) {
      return res.status(401).json({ success: false, message: 'Phiên đăng nhập không hợp lệ' });
    }

    const { data: profile, error: profileError } = await supabase
      .from('profiles')
      .select('role')
      .eq('id', user.id)
      .single();

    if (profileError || profile?.role !== 'admin') {
      return res.status(403).json({ success: false, message: 'Quyền truy cập bị từ chối: Yêu cầu quyền Quản trị viên' });
    }

    req.user = user;
    next();
  } catch (error) {
    return res.status(500).json({ success: false, message: 'Lỗi kiểm tra quyền hệ thống' });
  }
};

module.exports = requireAdmin;

---

## 2. Quản lý Ảnh sản phẩm với Supabase Storage
Để quản trị viên tải lên ảnh sản phẩm:
- Ảnh được tải trực tiếp lên bucket có tên product-images trên Supabase Storage.
- Controller phía Backend hỗ trợ sinh tên định danh ngẫu nhiên để tránh trùng tên tệp:

// src/controllers/adminMediaController.js
const supabase = require('../config/supabase');
const crypto = require('crypto');

exports.uploadProductImage = async (req, res) => {
  try {
    const file = req.file; // Sử dụng qua middleware multer (bộ nhớ tạm memoryStorage)
    if (!file) {
      return res.status(400).json({ success: false, message: 'Không tìm thấy tệp tải lên' });
    }

    const fileExt = file.originalname.split('.').pop();
    const fileName = `${crypto.randomUUID()}.${fileExt}`;
    const filePath = `products/${fileName}`;

    const { error: uploadError } = await supabase.storage
      .from('product-images')
      .upload(filePath, file.buffer, {
        contentType: file.mimetype,
        upsert: false
      });

    if (uploadError) throw uploadError;

    const { data: { publicUrl } } = supabase.storage
      .from('product-images')
      .getPublicUrl(filePath);

    return res.status(200).json({ success: true, url: publicUrl });
  } catch (error) {
    return res.status(500).json({ success: false, message: error.message });
  }
};

---

## 3. Kiến trúc Xử lý Cổng Thanh toán VNPAY IPN (Instant Payment Notification)
IPN là kênh giao tiếp ngầm giữa server VNPAY và backend server để cập nhật trạng thái đơn hàng một cách bảo mật và tin cậy.

Luồng đối soát mã băm bảo mật (Checksum Validation):
1. Cổng VNPAY gửi GET request tới /api/payments/vnpay-ipn
2. Tách chuỗi tham số và mã băm 'vnp_SecureHash'
3. Sắp xếp toàn bộ tham số còn lại theo bảng chữ cái A-Z
4. Tạo mã băm HMAC-SHA512 với chuỗi tham số đã sắp xếp bằng VNP_HASH_SECRET
5. So sánh: Mã băm vừa tạo === vnp_SecureHash
   - Nếu không khớp: Phản hồi { RspCode: '97', Message: 'Invalid Checksum' }
   - Nếu trùng khớp:
     * Kiểm tra đơn hàng có tồn tại không? (Không: RspCode '01')
     * Kiểm tra số tiền có khớp không? (Không: RspCode '04')
     * Kiểm tra đơn hàng đã xử lý trước đó chưa? (Đã xong: RspCode '02')
     * Hợp lệ toàn bộ: Cập nhật payment_status = 'paid', order_status = 'processing', trừ tồn kho stock và phản hồi { RspCode: '00', Message: 'Confirm Success' }

Mã nguồn Controller xử lý IPN chuẩn VNPAY:

// src/controllers/paymentController.js
const crypto = require('crypto');
const querystring = require('qs');
const supabase = require('../config/supabase');

exports.handleVnpayIpn = async (req, res) => {
  try {
    let vnp_Params = { ...req.query };
    const secureHash = vnp_Params['vnp_SecureHash'];

    delete vnp_Params['vnp_SecureHash'];
    delete vnp_Params['vnp_SecureHashType'];

    vnp_Params = Object.keys(vnp_Params)
      .sort()
      .reduce((result, key) => {
        result[key] = vnp_Params[key];
        return result;
      }, {});

    const secretKey = process.env.VNP_HASH_SECRET;
    const signData = querystring.stringify(vnp_Params, { encode: false });
    const checkHash = crypto.createHmac('sha512', secretKey).update(Buffer.from(signData, 'utf-8')).digest('hex');

    if (secureHash !== checkHash) {
      return res.status(200).json({ RspCode: '97', Message: 'Checksum failed' });
    }

    const orderCode = vnp_Params['vnp_TxnRef'];
    const rspCode = vnp_Params['vnp_ResponseCode'];
    const vnpAmount = Number(vnp_Params['vnp_Amount']) / 100;

    const { data: order, error: orderError } = await supabase
      .from('orders')
      .select('*, order_items(*)')
      .eq('order_code', orderCode)
      .single();

    if (orderError || !order) {
      return res.status(200).json({ RspCode: '01', Message: 'Order not found' });
    }

    if (Number(order.total_amount) !== vnpAmount) {
      return res.status(200).json({ RspCode: '04', Message: 'Amount invalid' });
    }

    if (order.payment_status === 'paid') {
      return res.status(200).json({ RspCode: '02', Message: 'Order already confirmed' });
    }

    if (rspCode === '00') {
      await supabase
        .from('orders')
        .update({
          payment_status: 'paid',
          order_status: 'processing',
          updated_at: new Date().toISOString()
        })
        .eq('order_code', orderCode);

      for (const item of order.order_items) {
        if (item.product_id) {
          const { data: prod } = await supabase
            .from('products')
            .select('stock')
            .eq('id', item.product_id)
            .single();

          if (prod) {
            const nextStock = Math.max(0, prod.stock - item.quantity);
            await supabase
              .from('products')
              .update({ stock: nextStock })
              .eq('id', item.product_id);
          }
        }
      }

      return res.status(200).json({ RspCode: '00', Message: 'Confirm Success' });
    } else {
      await supabase
        .from('orders')
        .update({
          payment_status: 'failed',
          updated_at: new Date().toISOString()
        })
        .eq('order_code', orderCode);

      return res.status(200).json({ RspCode: '00', Message: 'Confirm Success' });
    }
  } catch (err) {
    return res.status(200).json({ RspCode: '99', Message: 'Unknown error' });
  }
};

---

## 4. API Báo cáo & Thống kê dành cho Quản trị viên
- Tổng quan Dashboard (GET /api/admin/analytics/overview):
  * Tổng doanh thu tích lũy từ các đơn paid.
  * Tổng số đơn hàng thành công, số đơn đang chờ xử lý.
  * Tổng số sản phẩm có mức tồn kho báo động (stock <= 5).
- Doanh thu theo chu kỳ (GET /api/admin/analytics/revenue?period=month):
  * Sử dụng truy vấn SQL gộp theo ngày/tháng để vẽ biểu đồ doanh số Line Chart/Bar Chart trên giao diện Admin.