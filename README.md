# 🏛️ ONLINE AUCTION SYSTEM - HỆ THỐNG ĐẤU GIÁ TRỰC TUYẾN

> **Dự án**: Phát triển ứng dụng Web - Final Project  
> **Môn học**: PTUDW (Web Application Development)  
> **Năm học**: 2024-2025  
> **Nhóm phát triển**: TayDuKy Team

---

## 📋 MỤC LỤC

- [Giới thiệu](#-giới-thiệu)
- [Tech Stack](#-tech-stack)
- [Kiến trúc hệ thống](#-kiến-trúc-hệ-thống)
- [Các chức năng đã triển khai](#-các-chức-năng-đã-triển-khai)
- [Database Schema](#-database-schema)
- [Cài đặt và chạy dự án](#-cài-đặt-và-chạy-dự-án)
- [Test Cases - Bộ test chi tiết](#-test-cases---bộ-test-chi-tiết)
- [Team Workflow](#-team-workflow)
- [API Documentation](#-api-documentation)

---

## 🎯 GIỚI THIỆU

Hệ thống đấu giá trực tuyến cho phép người dùng:
- 👥 **Guest**: Xem sản phẩm, tìm kiếm, xem chi tiết
- 🎯 **Bidder**: Đấu giá sản phẩm, theo dõi watchlist, lịch sử đấu giá
- 🏪 **Seller**: Đăng bán sản phẩm, quản lý sản phẩm, xem thống kê
- 👨‍💼 **Admin**: Quản lý users, duyệt sản phẩm, quản lý categories, xử lý spam

---

## 🛠️ TECH STACK

### **Backend**
- **Runtime**: Node.js v22.20.0
- **Framework**: Express.js 4.18.2
- **Language**: JavaScript (ES Modules)
- **Database**: Supabase (PostgreSQL)
- **Authentication**: 
  - JWT (Access Token + Refresh Token)
  - Passport.js (Google OAuth)
  - OTP Email Verification (Nodemailer)
- **File Upload**: Multer + Supabase Storage
- **Validation**: Express Validator
- **Security**: bcrypt, cookie-parser, CORS

### **Frontend**
- **Framework**: React 19.1.1
- **Build Tool**: Vite 7.1.7
- **Styling**: Tailwind CSS 3.4.17
- **Routing**: React Router DOM 7.9.5
- **HTTP Client**: Axios 1.13.1
- **Rich Text Editor**: Quill 1.3.7
- **reCAPTCHA**: react-google-recaptcha 3.1.0

### **Database & Storage**
- **Database**: Supabase PostgreSQL
- **Storage**: Supabase Storage (product images, avatars, payment proofs)
- **Email Service**: Nodemailer (Gmail SMTP)

---

## 🏗️ KIẾN TRÚC HỆ THỐNG

```
auction-system/
├── Backend/
│   ├── config/           # Passport, Supabase config
│   ├── controllers/      # Business logic (5 controllers)
│   ├── middleware/       # Auth middleware
│   ├── routes/           # API routes (5 modules)
│   ├── utils/            # OTP, upload, system settings
│   ├── migrations/       # Database migrations
│   └── server.js         # Entry point
│
├── Frontend/
│   ├── src/
│   │   ├── components/   # React components
│   │   ├── pages/        # Page components
│   │   ├── services/     # API services
│   │   ├── context/      # React Context
│   │   └── lib/          # Supabase client
│   └── vite.config.js
│
└── Database/
    ├── DATABASE-SCHEMA.sql       # Full schema
    ├── DATABASE-OTP-TABLE.sql    # OTP table
    └── UPDATE-RPC-FUNCTIONS.sql  # PostgreSQL functions
```

---

## ✅ CÁC CHỨC NĂNG ĐÃ TRIỂN KHAI

### 🔐 **1. AUTHENTICATION & AUTHORIZATION**
**Người phụ trách**: Chung (Team)

#### Chức năng:
- ✅ Đăng ký tài khoản (email + password)
- ✅ Xác thực email bằng OTP (6 số, hết hạn sau 10 phút)
- ✅ Gửi lại OTP
- ✅ Đăng nhập (email + password)
- ✅ Đăng nhập bằng Google OAuth
- ✅ JWT Authentication (Access Token 15 phút + Refresh Token 7 ngày)
- ✅ Refresh Token (HTTP-only cookie)
- ✅ Đăng xuất
- ✅ Lấy thông tin profile
- ✅ Middleware xác thực role (guest/bidder/seller/admin)

#### API Endpoints:
```
POST   /api/auth/register          # Đăng ký
POST   /api/auth/verify-otp        # Verify OTP
POST   /api/auth/resend-otp        # Gửi lại OTP
POST   /api/auth/login             # Đăng nhập
GET    /api/auth/google            # Google OAuth
GET    /api/auth/google/callback   # Google callback
POST   /api/auth/refresh           # Refresh token
POST   /api/auth/logout            # Đăng xuất
GET    /api/auth/profile           # Lấy profile (protected)
```

---

### 👥 **2. GUEST FEATURES** (Người chưa đăng nhập)
**Người phụ trách**: KHẢI

#### Chức năng:
- ✅ Xem danh sách sản phẩm đấu giá (pagination, filter theo category)
- ✅ Xem chi tiết sản phẩm (thông tin seller, lịch sử bid, Q&A)
- ✅ Tìm kiếm sản phẩm (full-text search)
- ✅ Xem danh sách categories (2 cấp)
- ✅ Xem sản phẩm nổi bật:
  - Sắp kết thúc (ending_soon)
  - Nhiều bid nhất (most_bids)
  - Giá cao nhất (highest_price)
- ✅ Xem profile người bán (public info)

#### API Endpoints:
```
GET    /api/guest/products              # Danh sách sản phẩm
GET    /api/guest/products/:id          # Chi tiết sản phẩm
GET    /api/guest/search?q=keyword      # Tìm kiếm
GET    /api/guest/categories            # Danh sách categories
GET    /api/guest/featured?type=...     # Sản phẩm nổi bật
GET    /api/guest/sellers/:id           # Profile seller
```

---

### 🎯 **3. BIDDER FEATURES** (Người đấu giá)
**Người phụ trách**: KHOA

#### Chức năng:
- ✅ Xem danh sách sản phẩm đấu giá (filter, sort)
- ✅ Đặt giá đấu (place bid):
  - Kiểm tra giá hợp lệ (>= current_price + step_price)
  - Kiểm tra bidder có bị từ chối không
  - Kiểm tra rating >= 80% (nếu seller yêu cầu)
  - Tự động gia hạn nếu bid trước 5 phút kết thúc
- ✅ Mua ngay (Buy Now) nếu sản phẩm có giá Buy Now
- ✅ Xem lịch sử đấu giá của tôi
- ✅ Thêm/xóa sản phẩm vào Watchlist
- ✅ Xem danh sách Watchlist
- ✅ Xem lịch sử giá đấu của sản phẩm
- ✅ Gửi câu hỏi cho người bán (Q&A)
- ✅ **Checkout & Payment**:
  - Xem thông tin đơn hàng
  - Cập nhật địa chỉ giao hàng
  - Upload ảnh hoá đơn thanh toán (payment proof)

#### API Endpoints:
```
GET    /api/bidder/products              # Danh sách sản phẩm
POST   /api/bidder/bids                  # Đặt giá đấu
GET    /api/bidder/bids/my               # Lịch sử đấu giá
POST   /api/bidder/watchlist             # Thêm watchlist
DELETE /api/bidder/watchlist/:productId  # Xóa watchlist
GET    /api/bidder/watchlist             # Danh sách watchlist
GET    /api/bidder/products/:id/bids     # Lịch sử bid của sản phẩm
POST   /api/bidder/products/:id/questions # Gửi câu hỏi
GET    /api/bidder/orders/:productId     # Xem đơn hàng
POST   /api/bidder/orders                # Cập nhật đơn hàng
POST   /api/bidder/uploads/payment-proof # Upload ảnh thanh toán
```

---

### 🏪 **4. SELLER FEATURES** (Người bán)
**Người phụ trách**: CƯỜNG

#### Chức năng:
- ✅ **Profile Management**:
  - Xem profile seller
  - Cập nhật profile (full_name, address, phone, date_of_birth)
  - Upload avatar
- ✅ **Product Management**:
  - Đăng sản phẩm mới (pending → admin duyệt)
  - Upload ảnh sản phẩm (Supabase Storage)
  - Xem danh sách sản phẩm của tôi (filter theo status)
  - Cập nhật sản phẩm (chỉ khi chưa có bid)
  - Xóa sản phẩm (chỉ khi chưa có bid)
  - Xem danh sách giá đấu của sản phẩm
- ✅ **Sales Analytics**:
  - Thống kê doanh thu (tổng sản phẩm, đã bán, pending, rejected)

#### API Endpoints:
```
GET    /api/seller/profile                 # Xem profile
PUT    /api/seller/profile                 # Cập nhật profile
POST   /api/seller/profile/avatar          # Upload avatar
POST   /api/seller/uploads/images          # Upload ảnh sản phẩm
POST   /api/seller/products                # Đăng sản phẩm
GET    /api/seller/products                # Danh sách sản phẩm
PUT    /api/seller/products/:id            # Cập nhật sản phẩm
DELETE /api/seller/products/:id            # Xóa sản phẩm
GET    /api/seller/products/:id/bids       # Lịch sử bid
GET    /api/seller/stats                   # Thống kê doanh thu
```

---

### 👨‍💼 **5. ADMIN FEATURES** (Quản trị viên)
**Người phụ trách**: THẮNG

#### Chức năng:
- ✅ **User Management**:
  - Xem danh sách users (filter theo role, pagination)
  - Xem chi tiết user
  - Thay đổi role (bidder ↔ seller ↔ admin)
  - Cấm user (ban/unban)
  - Xóa user
- ✅ **Product Management**:
  - Xem tất cả sản phẩm (filter theo status)
  - Duyệt sản phẩm (pending → active)
  - Từ chối sản phẩm (pending → rejected với lý do)
  - Xóa sản phẩm vi phạm
- ✅ **Upgrade Requests**:
  - Xem yêu cầu nâng cấp bidder → seller
  - Duyệt/từ chối yêu cầu
- ✅ **Category Management**:
  - Xem danh sách categories
  - Tạo category mới (2 cấp)
  - Cập nhật category
  - Xóa category (không được xóa nếu có sản phẩm)
- ✅ **Bid Management**:
  - Xem lịch sử đấu giá
  - Hủy bid (xử lý gian lận)
  - Giải quyết tranh chấp
- ✅ **Spam Management**:
  - Xem báo cáo spam (user/product/bid)
  - Xử lý spam (warn, ban, delete)
  - Bỏ qua báo cáo spam
  - Thống kê spam
- ✅ **System Settings**:
  - Xem/cập nhật cài đặt hệ thống
- ✅ **System Stats**:
  - Thống kê tổng quan (users, products, bids, revenue)

#### API Endpoints:
```
# User Management
GET    /api/admin/users                    # Danh sách users
GET    /api/admin/users/:id                # Chi tiết user
PUT    /api/admin/users/:id/role           # Thay đổi role
POST   /api/admin/users/:id/ban            # Cấm user
POST   /api/admin/users/:id/unban          # Gỡ cấm user
DELETE /api/admin/users/:id                # Xóa user

# Product Management
GET    /api/admin/products                 # Tất cả sản phẩm
POST   /api/admin/products/:id/approve     # Duyệt sản phẩm
POST   /api/admin/products/:id/reject      # Từ chối sản phẩm
DELETE /api/admin/products/:id             # Xóa sản phẩm

# Upgrade Requests
GET    /api/admin/upgrades                 # Yêu cầu nâng cấp
POST   /api/admin/upgrades/:id/approve     # Duyệt yêu cầu
POST   /api/admin/upgrades/:id/reject      # Từ chối yêu cầu

# Category Management
GET    /api/admin/categories               # Danh sách categories
GET    /api/admin/categories/:id           # Chi tiết category
POST   /api/admin/categories               # Tạo category
PUT    /api/admin/categories/:id           # Cập nhật category
DELETE /api/admin/categories/:id           # Xóa category

# Bid Management
GET    /api/admin/bids                     # Lịch sử đấu giá
POST   /api/admin/bids/:id/cancel          # Hủy bid
POST   /api/admin/bids/:id/resolve-dispute # Giải quyết tranh chấp

# Spam Management
GET    /api/admin/spam-reports             # Báo cáo spam
GET    /api/admin/spam-reports/:id         # Chi tiết báo cáo
POST   /api/admin/spam-reports/:id/resolve # Xử lý spam
POST   /api/admin/spam-reports/:id/dismiss # Bỏ qua báo cáo
GET    /api/admin/spam-stats               # Thống kê spam

# System
GET    /api/admin/stats                    # Thống kê hệ thống
GET    /api/admin/settings                 # Cài đặt hệ thống
PUT    /api/admin/settings                 # Cập nhật cài đặt
```

---

## 🗄️ DATABASE SCHEMA

### **Tables (15 bảng)**

1. **profiles** - Thông tin user (bidder, seller, admin)
2. **upgrade_requests** - Yêu cầu nâng cấp bidder → seller
3. **ratings** - Đánh giá +1/-1 giữa users
4. **categories** - Danh mục 2 cấp
5. **products** - Sản phẩm đấu giá
6. **product_descriptions** - Lịch sử mô tả sản phẩm (append-only)
7. **bids** - Lịch sử đấu giá
8. **watchlist** - Danh sách yêu thích
9. **rejected_bidders** - Bidder bị từ chối
10. **questions** - Câu hỏi & trả lời
11. **orders** - Đơn hàng sau đấu giá
12. **order_chat** - Chat giữa buyer & seller
13. **system_settings** - Cài đặt hệ thống
14. **otp_verifications** - OTP email verification
15. **spam_reports** - Báo cáo spam

### **Key Features**
- ✅ Full-text search với `tsvector`
- ✅ Triggers tự động (rating, bid count, watchlist count)
- ✅ RLS (Row Level Security)
- ✅ PostgreSQL Functions (get_top_ending_soon, get_top_most_bids, etc.)

Chi tiết schema: [`Backend/DATABASE-SCHEMA.sql`](Backend/DATABASE-SCHEMA.sql)

---

## 🚀 CÀI ĐẶT VÀ CHẠY DỰ ÁN

### **1. Requirements**
- Node.js >= 18.0.0
- npm hoặc yarn
- Supabase account

### **2. Clone Repository**
```bash
git clone https://github.com/hungtmh/online-auction-system.git
cd online-auction-system/auction-system
```

### **3. Setup Backend**
```bash
cd Backend
npm install

# Tạo file .env
cp .env.example .env
# Cập nhật các biến môi trường:
# - SUPABASE_URL
# - SUPABASE_SERVICE_ROLE_KEY
# - JWT_SECRET
# - GOOGLE_CLIENT_ID
# - GOOGLE_CLIENT_SECRET
# - EMAIL_USER (Gmail)
# - EMAIL_PASSWORD (App Password)

# Chạy Backend
npm run dev  # Port 5000
```

### **4. Setup Frontend**
```bash
cd Frontend
npm install

# Tạo file .env
cp .env.example .env
# Cập nhật:
# - VITE_SUPABASE_URL
# - VITE_SUPABASE_ANON_KEY
# - VITE_API_URL=http://localhost:5000
# - VITE_RECAPTCHA_SITE_KEY

# Chạy Frontend
npm run dev  # Port 5173
```

### **5. Setup Database**
1. Tạo project trên Supabase
2. Chạy SQL trong Supabase SQL Editor:
   - `Backend/DATABASE-SCHEMA.sql`
   - `Backend/DATABASE-OTP-TABLE.sql`
   - `Backend/UPDATE-RPC-FUNCTIONS.sql`
3. Tạo Storage Buckets:
   - `product-images` (public)
   - `avatars` (public)
   - `payment-proofs` (private)

Chi tiết: [`Backend/DATABASE-SETUP.md`](Backend/DATABASE-SETUP.md)

---

## 🧪 TEST CASES - BỘ TEST CHI TIẾT

### **🔐 MODULE 1: AUTHENTICATION**

#### **TC-AUTH-001: Đăng ký tài khoản thành công**
- **Mô tả**: User đăng ký tài khoản mới với email chưa tồn tại
- **Preconditions**: Email chưa được đăng ký
- **Test Steps**:
  1. POST `/api/auth/register`
  2. Body: `{ email, password, full_name, address }`
- **Expected Result**:
  - Status: 201
  - Response: `{ success: true, requireOTPVerification: true }`
  - OTP được gửi qua email
  - User được tạo trong `profiles` với `role = 'bidder'`
- **Test Data**:
  ```json
  {
    "email": "test@example.com",
    "password": "password123",
    "full_name": "Nguyen Van A",
    "address": "123 Street, HCMC"
  }
  ```

#### **TC-AUTH-002: Đăng ký thất bại - Email đã tồn tại**
- **Test Steps**: POST `/api/auth/register` với email đã đăng ký
- **Expected Result**: Status 400, message: "Email đã được đăng ký"

#### **TC-AUTH-003: Xác thực OTP thành công**
- **Preconditions**: User đã đăng ký, OTP hợp lệ
- **Test Steps**:
  1. POST `/api/auth/verify-otp`
  2. Body: `{ email, otp_code }`
- **Expected Result**:
  - Status: 200
  - `email_confirmed_at` được cập nhật trong Supabase Auth
  - Message: "Xác thực email thành công!"

#### **TC-AUTH-004: Xác thực OTP thất bại - OTP sai**
- **Test Steps**: POST `/api/auth/verify-otp` với OTP sai
- **Expected Result**: Status 400, message: "Mã OTP không chính xác"

#### **TC-AUTH-005: Xác thực OTP thất bại - OTP hết hạn**
- **Test Steps**: POST `/api/auth/verify-otp` với OTP quá 10 phút
- **Expected Result**: Status 400, message: "Mã OTP đã hết hạn"

#### **TC-AUTH-006: Gửi lại OTP thành công**
- **Preconditions**: User chưa verify email
- **Test Steps**: POST `/api/auth/resend-otp` với `{ email }`
- **Expected Result**: Status 200, OTP mới được gửi

#### **TC-AUTH-007: Đăng nhập thành công**
- **Preconditions**: User đã verify email
- **Test Steps**:
  1. POST `/api/auth/login`
  2. Body: `{ email, password }`
- **Expected Result**:
  - Status: 200
  - Response: `{ accessToken, refreshToken (cookie), user }`
  - JWT token hợp lệ (verify với JWT_SECRET)

#### **TC-AUTH-008: Đăng nhập thất bại - Email chưa verify**
- **Test Steps**: POST `/api/auth/login` với email chưa verify
- **Expected Result**: Status 403, `requireEmailVerification: true`

#### **TC-AUTH-009: Đăng nhập thất bại - Sai mật khẩu**
- **Test Steps**: POST `/api/auth/login` với password sai
- **Expected Result**: Status 401, message: "Email hoặc mật khẩu không đúng"

#### **TC-AUTH-010: Refresh Token thành công**
- **Preconditions**: User đã đăng nhập, có refreshToken trong cookie
- **Test Steps**: POST `/api/auth/refresh`
- **Expected Result**: Status 200, accessToken mới

#### **TC-AUTH-011: Đăng xuất thành công**
- **Test Steps**: POST `/api/auth/logout`
- **Expected Result**: refreshToken cookie bị xóa

#### **TC-AUTH-012: Google OAuth thành công**
- **Test Steps**:
  1. GET `/api/auth/google` (redirect to Google)
  2. Google callback `/api/auth/google/callback`
- **Expected Result**: Redirect về frontend với access token

---

### **👥 MODULE 2: GUEST FEATURES**

#### **TC-GUEST-001: Xem danh sách sản phẩm**
- **Test Steps**: GET `/api/guest/products?page=1&limit=12`
- **Expected Result**:
  - Status: 200
  - Trả về danh sách sản phẩm với pagination
  - Chỉ sản phẩm `status = 'active'`

#### **TC-GUEST-002: Filter sản phẩm theo category**
- **Test Steps**: GET `/api/guest/products?category=<uuid>`
- **Expected Result**: Chỉ trả về sản phẩm thuộc category đó

#### **TC-GUEST-003: Xem chi tiết sản phẩm**
- **Test Steps**: GET `/api/guest/products/:id`
- **Expected Result**:
  - Thông tin sản phẩm (name, description, images, pricing)
  - Thông tin seller (full_name, rating)
  - Lịch sử bid (top 10)
  - Q&A (questions với answers)

#### **TC-GUEST-004: Tìm kiếm sản phẩm**
- **Test Steps**: GET `/api/guest/search?q=iphone`
- **Expected Result**: Sản phẩm có `name` hoặc `description` chứa "iphone"

#### **TC-GUEST-005: Xem categories**
- **Test Steps**: GET `/api/guest/categories`
- **Expected Result**: Danh sách categories 2 cấp (cha-con)

#### **TC-GUEST-006: Xem sản phẩm sắp kết thúc**
- **Test Steps**: GET `/api/guest/featured?type=ending_soon&limit=6`
- **Expected Result**: Top 6 sản phẩm gần `end_time` nhất

#### **TC-GUEST-007: Xem sản phẩm nhiều bid nhất**
- **Test Steps**: GET `/api/guest/featured?type=most_bids&limit=6`
- **Expected Result**: Top 6 sản phẩm có `bid_count` cao nhất

#### **TC-GUEST-008: Xem sản phẩm giá cao nhất**
- **Test Steps**: GET `/api/guest/featured?type=highest_price&limit=6`
- **Expected Result**: Top 6 sản phẩm có `current_price` cao nhất

#### **TC-GUEST-009: Xem profile seller**
- **Test Steps**: GET `/api/guest/sellers/:id`
- **Expected Result**: Thông tin seller (full_name, rating, product_count)

---

### **🎯 MODULE 3: BIDDER FEATURES**

#### **TC-BIDDER-001: Đặt giá đấu thành công**
- **Preconditions**: User role = bidder, đã login
- **Test Steps**:
  1. POST `/api/bidder/bids`
  2. Body: `{ product_id, bid_amount }`
  3. `bid_amount >= current_price + step_price`
- **Expected Result**:
  - Status: 201
  - Bid được lưu vào `bids` table
  - `current_price` của product được cập nhật
  - `bid_count` tăng 1

#### **TC-BIDDER-002: Đặt giá thất bại - Giá thấp hơn step_price**
- **Test Steps**: POST `/api/bidder/bids` với `bid_amount < current_price + step_price`
- **Expected Result**: Status 400, message: "Giá đấu phải lớn hơn..."

#### **TC-BIDDER-003: Đặt giá thất bại - Bidder bị từ chối**
- **Preconditions**: Bidder nằm trong `rejected_bidders`
- **Test Steps**: POST `/api/bidder/bids`
- **Expected Result**: Status 403, message: "Bạn đã bị từ chối tham gia đấu giá"

#### **TC-BIDDER-004: Đặt giá thất bại - Rating < 80%**
- **Preconditions**: Product `allow_unrated_bidders = false`, bidder rating < 80%
- **Test Steps**: POST `/api/bidder/bids`
- **Expected Result**: Status 403, message: "Rating không đủ 80%"

#### **TC-BIDDER-005: Tự động gia hạn khi bid trước 5 phút**
- **Preconditions**: Product `auto_extend = true`, bid trước 5 phút kết thúc
- **Test Steps**: POST `/api/bidder/bids`
- **Expected Result**: `end_time` được gia hạn thêm 10 phút

#### **TC-BIDDER-006: Mua ngay (Buy Now)**
- **Preconditions**: Product có `buy_now_price`
- **Test Steps**: POST `/api/bidder/bids` với `bid_amount = buy_now_price`
- **Expected Result**:
  - Product `status = 'completed'`
  - `winner_id = bidder_id`
  - Order được tạo

#### **TC-BIDDER-007: Thêm vào Watchlist**
- **Test Steps**: POST `/api/bidder/watchlist` với `{ product_id }`
- **Expected Result**:
  - Status: 201
  - `watchlist_count` của product tăng 1

#### **TC-BIDDER-008: Xóa khỏi Watchlist**
- **Test Steps**: DELETE `/api/bidder/watchlist/:productId`
- **Expected Result**: `watchlist_count` giảm 1

#### **TC-BIDDER-009: Xem Watchlist**
- **Test Steps**: GET `/api/bidder/watchlist`
- **Expected Result**: Danh sách sản phẩm trong watchlist

#### **TC-BIDDER-010: Xem lịch sử đấu giá của tôi**
- **Test Steps**: GET `/api/bidder/bids/my`
- **Expected Result**: Danh sách bid của user (winning, losing, outbid)

#### **TC-BIDDER-011: Xem lịch sử bid của sản phẩm**
- **Test Steps**: GET `/api/bidder/products/:id/bids`
- **Expected Result**: Lịch sử bid của sản phẩm (DESC by created_at)

#### **TC-BIDDER-012: Gửi câu hỏi cho seller**
- **Test Steps**: POST `/api/bidder/products/:id/questions` với `{ question }`
- **Expected Result**: Question được lưu vào `questions` table

#### **TC-BIDDER-013: Upload ảnh thanh toán**
- **Preconditions**: User đã thắng đấu giá
- **Test Steps**: POST `/api/bidder/uploads/payment-proof` (multipart/form-data)
- **Expected Result**:
  - Ảnh được upload lên Supabase Storage
  - `payment_proof_url` được cập nhật trong `orders`

---

### **🏪 MODULE 4: SELLER FEATURES**

#### **TC-SELLER-001: Đăng sản phẩm mới**
- **Preconditions**: User role = seller
- **Test Steps**:
  1. POST `/api/seller/products`
  2. Body: `{ name, description, category_id, starting_price, step_price, end_time, thumbnail_url }`
- **Expected Result**:
  - Status: 201
  - Product được tạo với `status = 'pending'`
  - Admin cần duyệt

#### **TC-SELLER-002: Upload ảnh sản phẩm**
- **Test Steps**: POST `/api/seller/uploads/images` (multipart/form-data)
- **Expected Result**:
  - Ảnh được upload lên bucket `product-images`
  - Trả về public URL

#### **TC-SELLER-003: Cập nhật sản phẩm**
- **Preconditions**: Product chưa có bid
- **Test Steps**: PUT `/api/seller/products/:id`
- **Expected Result**: Product được cập nhật

#### **TC-SELLER-004: Cập nhật sản phẩm thất bại - Đã có bid**
- **Preconditions**: Product đã có bid
- **Test Steps**: PUT `/api/seller/products/:id`
- **Expected Result**: Status 400, message: "Không thể sửa sản phẩm đã có người đấu giá"

#### **TC-SELLER-005: Xóa sản phẩm**
- **Preconditions**: Product chưa có bid
- **Test Steps**: DELETE `/api/seller/products/:id`
- **Expected Result**: Product bị xóa

#### **TC-SELLER-006: Xóa sản phẩm thất bại - Đã có bid**
- **Test Steps**: DELETE `/api/seller/products/:id` (có bid)
- **Expected Result**: Status 400

#### **TC-SELLER-007: Xem danh sách sản phẩm của tôi**
- **Test Steps**: GET `/api/seller/products?status=pending`
- **Expected Result**: Chỉ sản phẩm của seller, filter theo status

#### **TC-SELLER-008: Xem lịch sử bid của sản phẩm**
- **Test Steps**: GET `/api/seller/products/:id/bids`
- **Expected Result**: Lịch sử bid của sản phẩm (DESC)

#### **TC-SELLER-009: Xem thống kê doanh thu**
- **Test Steps**: GET `/api/seller/stats`
- **Expected Result**:
  - `total_products`
  - `sold_products`
  - `pending_products`
  - `rejected_products`

#### **TC-SELLER-010: Upload avatar**
- **Test Steps**: POST `/api/seller/profile/avatar` (multipart/form-data)
- **Expected Result**: `avatar_url` được cập nhật

#### **TC-SELLER-011: Cập nhật profile**
- **Test Steps**: PUT `/api/seller/profile` với `{ full_name, address, phone }`
- **Expected Result**: Profile được cập nhật

---

### **👨‍💼 MODULE 5: ADMIN FEATURES**

#### **TC-ADMIN-001: Xem danh sách users**
- **Test Steps**: GET `/api/admin/users?page=1&limit=20`
- **Expected Result**: Danh sách tất cả users với pagination

#### **TC-ADMIN-002: Filter users theo role**
- **Test Steps**: GET `/api/admin/users?role=bidder`
- **Expected Result**: Chỉ users có role = bidder

#### **TC-ADMIN-003: Thay đổi role user**
- **Test Steps**: PUT `/api/admin/users/:id/role` với `{ role: 'seller' }`
- **Expected Result**: User role được cập nhật

#### **TC-ADMIN-004: Cấm user**
- **Test Steps**: POST `/api/admin/users/:id/ban` với `{ reason }`
- **Expected Result**:
  - `is_banned = true`
  - `banned_reason` được lưu

#### **TC-ADMIN-005: Gỡ cấm user**
- **Test Steps**: POST `/api/admin/users/:id/unban`
- **Expected Result**: `is_banned = false`

#### **TC-ADMIN-006: Xóa user**
- **Test Steps**: DELETE `/api/admin/users/:id`
- **Expected Result**: User bị xóa (cascade)

#### **TC-ADMIN-007: Duyệt sản phẩm**
- **Preconditions**: Product `status = 'pending'`
- **Test Steps**: POST `/api/admin/products/:id/approve`
- **Expected Result**: `status = 'active'`

#### **TC-ADMIN-008: Từ chối sản phẩm**
- **Test Steps**: POST `/api/admin/products/:id/reject` với `{ reason }`
- **Expected Result**: `status = 'rejected'`, `rejected_reason` được lưu

#### **TC-ADMIN-009: Xóa sản phẩm vi phạm**
- **Test Steps**: DELETE `/api/admin/products/:id`
- **Expected Result**: Product bị xóa

#### **TC-ADMIN-010: Duyệt yêu cầu nâng cấp**
- **Preconditions**: Upgrade request `status = 'pending'`
- **Test Steps**: POST `/api/admin/upgrades/:id/approve`
- **Expected Result**:
  - User `role = 'seller'`
  - Upgrade request `status = 'approved'`

#### **TC-ADMIN-011: Từ chối yêu cầu nâng cấp**
- **Test Steps**: POST `/api/admin/upgrades/:id/reject` với `{ admin_note }`
- **Expected Result**: `status = 'rejected'`

#### **TC-ADMIN-012: Tạo category mới**
- **Test Steps**: POST `/api/admin/categories` với `{ name, slug, parent_id }`
- **Expected Result**: Category được tạo

#### **TC-ADMIN-013: Cập nhật category**
- **Test Steps**: PUT `/api/admin/categories/:id`
- **Expected Result**: Category được cập nhật

#### **TC-ADMIN-014: Xóa category**
- **Preconditions**: Category không có sản phẩm
- **Test Steps**: DELETE `/api/admin/categories/:id`
- **Expected Result**: Category bị xóa

#### **TC-ADMIN-015: Xóa category thất bại - Có sản phẩm**
- **Test Steps**: DELETE `/api/admin/categories/:id` (có sản phẩm)
- **Expected Result**: Status 400

#### **TC-ADMIN-016: Hủy bid gian lận**
- **Test Steps**: POST `/api/admin/bids/:id/cancel`
- **Expected Result**: Bid bị hủy, `current_price` được rollback

#### **TC-ADMIN-017: Xử lý báo cáo spam**
- **Test Steps**: POST `/api/admin/spam-reports/:id/resolve` với `{ action: 'ban_user' }`
- **Expected Result**: User bị ban

#### **TC-ADMIN-018: Bỏ qua báo cáo spam**
- **Test Steps**: POST `/api/admin/spam-reports/:id/dismiss`
- **Expected Result**: `status = 'dismissed'`

#### **TC-ADMIN-019: Xem thống kê hệ thống**
- **Test Steps**: GET `/api/admin/stats`
- **Expected Result**:
  - Total users, products, bids
  - Revenue stats

#### **TC-ADMIN-020: Cập nhật cài đặt hệ thống**
- **Test Steps**: PUT `/api/admin/settings` với `{ auto_extend_minutes: 15 }`
- **Expected Result**: Setting được cập nhật

---

### **📊 SUMMARY: TỔNG HỢP TEST CASES**

| Module | Số lượng Test Cases | Trạng thái |
|--------|---------------------|------------|
| Authentication | 12 | ✅ Ready to test |
| Guest Features | 9 | ✅ Ready to test |
| Bidder Features | 13 | ✅ Ready to test |
| Seller Features | 11 | ✅ Ready to test |
| Admin Features | 20 | ✅ Ready to test |
| **TOTAL** | **65** | ✅ |

---

## 👥 TEAM WORKFLOW

### **Phân công theo module**:
- **KHẢI**: Guest Features + Frontend Guest UI
- **KHOA**: Bidder Features + Frontend Bidder Dashboard
- **CƯỜNG**: Seller Features + Frontend Seller Dashboard
- **THẮNG**: Admin Features + Frontend Admin Dashboard
- **CHUNG**: Authentication, Database Schema, System Settings

### **Git Workflow**:
```bash
# Branch naming: feature/<module>-<feature-name>
git checkout -b feature/bidder-place-bid
git commit -m "feat(bidder): implement place bid API"
git push origin feature/bidder-place-bid
# Tạo Pull Request → Review → Merge
```

Chi tiết: [`TEAM-WORKFLOW.md`](TEAM-WORKFLOW.md)

---

## 📚 API DOCUMENTATION

### **Base URL**
- Development: `http://localhost:5000`
- Production: TBD

### **Authentication**
Tất cả protected routes cần header:
```
Authorization: Bearer <access_token>
```

### **Response Format**
```json
{
  "success": true,
  "message": "Success message",
  "data": { ... }
}
```

Chi tiết API: Xem từng file route trong `Backend/routes/`

---

## 📖 TÀI LIỆU THAM KHẢO

- [`QUICK-START.md`](QUICK-START.md) - Hướng dẫn khởi động nhanh
- [`DATABASE-SETUP.md`](Backend/DATABASE-SETUP.md) - Setup database
- [`OTP-SETUP-GUIDE.md`](OTP-SETUP-GUIDE.md) - Setup OTP email
- [`TESTING-OTP-GUIDE.md`](TESTING-OTP-GUIDE.md) - Test OTP flow
- [`GET-SERVICE-KEY.md`](Backend/GET-SERVICE-KEY.md) - Lấy Supabase keys
- [`GET-RECAPTCHA-KEY.md`](Frontend/GET-RECAPTCHA-KEY.md) - Setup reCAPTCHA

---

## 📝 LICENSE

MIT License - TayDuKy Team © 2024-2025

---

## 🙏 ACKNOWLEDGMENTS

- **Giảng viên hướng dẫn**: [Tên GV]
- **Tech Stack**: Express.js, React, Supabase, Tailwind CSS
- **Team**: KHẢI, KHOA, CƯỜNG, THẮNG

---

**🎯 Trạng thái dự án**: ✅ **Hoàn thành 90%** - Sẵn sàng demo và test

**📅 Cập nhật lần cuối**: 27/11/2024

**🔗 Repository**: [https://github.com/hungtmh/online-auction-system](https://github.com/hungtmh/online-auction-system)
