# Backend — NhaTro

Tài liệu này dành cho developer mới, tổng hợp toàn bộ kiến trúc, nghiệp vụ, luồng kỹ thuật và quy tắc của backend ứng dụng NhaTro.

---

## 1. Tổng quan

**NhaTro** là ứng dụng quản lý nhà trọ. Backend cung cấp REST API phục vụ 3 loại người dùng: `admin`, `seller`, `buyer`.

Mọi endpoint (trừ `POST /api/auth/login`) đều yêu cầu xác thực bằng JWT Bearer token.

---

## 2. Kiến trúc tổng thể

```
Client (web/mobile)
    │
    ▼
REST API (NestJS hoặc Express)
    │
    ├── AuthModule        → login, JWT issue/verify
    ├── UsersModule       → CRUD user (admin, seller, buyer)
    ├── HousesModule      → quản lý house, meter    ├── GroupsModule        → quản lý group (nhóm house có chung price/formula)    ├── PricesModule      → quản lý price
    ├── FormulasModule    → quản lý formula
    ├── UsagesModule      → electricity/water usage
    ├── BillsModule       → tạo và xuất bill
    ├── PaymentsModule    → trạng thái thanh toán
    └── AuditLogModule    → ghi log mọi thao tác quan trọng
    │
    ▼
Database (PostgreSQL)
```

---

## 3. Roles & Phân quyền

| Role     | Giá trị | Mô tả                                     |
| -------- | ------- | ----------------------------------------- |
| `admin`  | `1`     | Quản trị viên, do developer provision sẵn |
| `seller` | `2`     | Chủ nhà trọ, do admin tạo                 |
| `buyer`  | `3`     | Người thuê trọ, do seller hoặc admin tạo  |

### Quy tắc phân quyền tạo tài khoản

| Caller    | Có thể tạo role                            |
| --------- | ------------------------------------------ |
| `admin`   | `seller` (2), `buyer` (3)                  |
| `seller`  | `buyer` (3) chỉ                            |
| `buyer`   | Không được tạo                             |
| Bất kỳ ai | Không được tạo `admin` (1) — cấm tuyệt đối |

---

## 4. Authentication

### 4.1 Cơ chế

- Dùng **JWT** (JSON Web Token).
- Endpoint login: `POST /api/auth/login` với `email` + `password`.
- Thành công trả về `accessToken` (JWT) và thông tin user cơ bản (`id`, `email`, `role`).
- Token có thời hạn mặc định **15 phút**, cấu hình qua `JWT_ACCESS_TOKEN_EXPIRATION`.
- Secret cấu hình qua `JWT_SECRET`.

### 4.2 Luồng login

```
Client                          Server
  │                               │
  │── POST /api/auth/login ──────▶│
  │   { email, password }         │
  │                               │── Tìm user theo email trong DB
  │                               │── So sánh password với bcrypt hash
  │                               │── Nếu khớp: tạo JWT, ghi audit_log(login_success)
  │                               │── Nếu không khớp: ghi audit_log(login_failed)
  │◀──────────────────────────────│
  │   200: { accessToken, user }  │
  │   401: generic error message  │
```

### 4.3 Quy tắc bảo mật

- **Không phân biệt** "email không tồn tại" và "sai password" trong response — luôn trả về `401` với cùng một thông báo chung (tránh enumeration attack).
- Password được hash bằng **bcrypt** trước khi lưu, không bao giờ trả về raw password.
- Thiếu field (`email` hoặc `password`) trả về `400`.

### 4.4 Các endpoint auth liên quan

| Method | Path                        | Mô tả                  |
| ------ | --------------------------- | ---------------------- |
| POST   | `/api/auth/login`           | Đăng nhập, nhận JWT    |
| POST   | `/api/auth/refresh`         | Làm mới access token   |
| POST   | `/api/auth/logout`          | Đăng xuất              |
| POST   | `/api/auth/forgot-password` | Yêu cầu reset password |
| POST   | `/api/auth/reset-password`  | Đặt lại password mới   |

---

## 5. Data Model

### 5.1 users (admin / seller / buyer)

```
id              UUID, PK
email           VARCHAR, UNIQUE, NOT NULL
password_hash   VARCHAR, NOT NULL
role            INT (1=admin, 2=seller, 3=buyer), NOT NULL
full_name       VARCHAR, nullable
phone           VARCHAR, nullable
address         VARCHAR, nullable (seller)
identification  VARCHAR, nullable (buyer)
verification_status  VARCHAR (seller)
is_active       BOOLEAN, DEFAULT true
auth_provider   VARCHAR, DEFAULT 'local'
last_login      TIMESTAMP, nullable
two_factor_enabled  BOOLEAN, DEFAULT false
created_by      UUID, nullable (FK → users.id)
created_at      TIMESTAMP
updated_at      TIMESTAMP
```

> `admin` accounts được tạo bởi developer qua seed script. `created_by` lưu developer id khi applicable.

### 5.2 group

```
id          UUID, PK
seller_id   UUID, FK → users.id (role=seller)
name        VARCHAR, NOT NULL
type        INT, NOT NULL  -- 1=house_group | 2=user_group (future)
created_at  TIMESTAMP
updated_at  TIMESTAMP
```

> `type` phân biệt mục đích của group. Hiện tại chỉ `type=1` (house group) được hỗ trợ; `type=2` (user group) dành cho mở rộng sau. API `/api/groups` được thiết kế để tái sử dụng cho cả hai loại mà không cần endpoint riêng.

### 5.3 house

```
id          UUID, PK
seller_id   UUID, FK → users.id (role=seller)
group_id    UUID, FK → group.id, nullable
name        VARCHAR
address     VARCHAR
unit        VARCHAR (số phòng / tên phòng)
area        DECIMAL (m²)
status      ENUM('active', 'vacant', 'maintenance')
deposit     DECIMAL
metadata    JSONB
created_at  TIMESTAMP
updated_at  TIMESTAMP
```

> Nếu `group_id` được gán, house kế thừa price và formula của group. Nếu `group_id` là null, house dùng price/formula được gán trực tiếp.

### 5.4 tenancy (lịch sử thuê phòng)

```
id          UUID, PK
house_id    UUID, FK → house.id
buyer_id    UUID, FK → users.id (role=buyer)
start_date  DATE
end_date    DATE, nullable (null = đang thuê)
role        VARCHAR
```

> Dùng bảng này thay vì `active_house_id` đơn giản để theo dõi lịch sử.

### 5.5 meter

```
id                 UUID, PK
house_id           UUID, FK → house.id
meter_type         ENUM('electric', 'water')
external_meter_id  VARCHAR (số đồng hồ thực tế)
installed_at       TIMESTAMP
metadata           JSONB
```

### 5.6 price (bảng giá)

```
id          UUID, PK
seller_id   UUID, FK → users.id (role=seller), NOT NULL
name        VARCHAR, NOT NULL    -- tên bảng giá do seller tự đặt, ví dụ: "tien dien"
unit_price  DECIMAL(15,4), NOT NULL  -- must be > 0
metadata    JSONB
created_at  TIMESTAMP
updated_at  TIMESTAMP
```

> Field cốt lõi: `name` (tên bảng giá) và `unit_price` (giá trị). Không có `price_type` — `name` là nhận diện duy nhất. `seller_id` được server gán tự động từ JWT.

### 5.7 formula

```
id          UUID, PK
seller_id   UUID, FK → users.id
name        VARCHAR
expression  TEXT / JSONB (DSL hoặc rules engine)
parameters  JSONB
description TEXT
created_at  TIMESTAMP
updated_at  TIMESTAMP
```

> Formula định nghĩa cách tính bill từ usage và price. Có thể là biểu thức toán học hoặc cấu trúc JSON rules.

### 5.8 group_price / group_formula (assignment tables)

```
-- group_price
group_id     UUID, FK → group.id
price_id     UUID, FK → price.id
assigned_at  TIMESTAMP

-- group_formula
group_id     UUID, FK → group.id
formula_id   UUID, FK → formula.id
assigned_at  TIMESTAMP
```

> Price và formula được gán vào `group`. Tất cả house trong group kế thừa cấu hình này.

### 5.9 house_price / house_formula (assignment tables — per-house override)

```
house_id     UUID, FK → house.id
price_id     UUID, FK → price.id
-- hoặc formula_id
assigned_at  TIMESTAMP
```

> Dùng khi house không thuộc group nào, hoặc cần override giá riêng. Khi house thuộc group, cấu hình của group ưu tiên hơn.

### 5.10 usage (electricity_usage / water_usage)

```
id                  UUID, PK
house_id            UUID, FK → house.id
meter_id            UUID, FK → meter.id
recorded_by         ENUM('buyer', 'seller')
recorded_by_id      UUID, FK → users.id
current_reading     DECIMAL
previous_reading    DECIMAL, nullable
reading_photo_urls  TEXT[] (mảng URL ảnh đồng hồ)
timestamp           TIMESTAMP
status              ENUM('pending', 'approved', 'discarded')
approved_by         UUID, FK → users.id, nullable
approved_at         TIMESTAMP, nullable
notes               TEXT
created_at          TIMESTAMP
updated_at          TIMESTAMP
```

### 5.11 bill

```
id              UUID, PK
house_id        UUID, FK → house.id
billing_cycle   VARCHAR (format: YYYY-MM)
issued_by       UUID, FK → users.id (seller/admin)
issued_at       TIMESTAMP
line_items      JSONB [{description, quantity, unit, unit_price, amount}]
subtotal        DECIMAL
tax             DECIMAL
total_amount    DECIMAL
status          ENUM('draft', 'issued', 'paid', 'void')
export_url      VARCHAR (URL ảnh/PDF đã export)
metadata        JSONB
created_at      TIMESTAMP
updated_at      TIMESTAMP
```

> **Ràng buộc quan trọng:** `UNIQUE(house_id, billing_cycle)` — mỗi house chỉ có tối đa 1 bill mỗi kỳ.

### 5.12 payment

```
id              UUID, PK
bill_id         UUID, FK → bill.id
buyer_id        UUID, FK → users.id
amount          DECIMAL
currency        VARCHAR, DEFAULT 'VND'
method          VARCHAR
status          ENUM('pending', 'paid', 'failed', 'refunded')
transaction_id  VARCHAR
paid_at         TIMESTAMP, nullable
created_at      TIMESTAMP
```

### 5.13 audit_log

```
id          UUID, PK
actor_type  ENUM('admin', 'seller', 'buyer', 'system')
actor_id    UUID
action      VARCHAR (login_success, login_failed, approve, discard, export, create, update, ...)
target_type VARCHAR (user, house, usage, bill, ...)
target_id   UUID, nullable
timestamp   TIMESTAMP
metadata    JSONB
```

---

## 6. Luồng nghiệp vụ chính

### 6.1 Setup ban đầu (do developer)

```
1. Chạy migrations: pnpm migrate
2. Chạy seed:       pnpm seed
   → Tạo admin account: admin@nhatro.com / Admin@123
   → Idempotent: chạy nhiều lần không tạo trùng
3. Admin đổi mật khẩu sau lần đăng nhập đầu tiên
```

### 6.2 Onboarding seller

```
admin đăng nhập
  → POST /api/users { email, password, role: 2 }
  → Seller account được tạo, isActive=true
  → Seller có thể đăng nhập ngay lập tức
```

### 6.3 Seller thiết lập nhà trọ

```
seller đăng nhập
  → Tạo group (tuỳ chọn): POST /api/groups { name }
  → Tạo house:   POST /api/houses
  → Tạo meter:   POST /api/houses/:id/meters  (electric, water)
  → Tạo price:   POST /api/prices  (electric price, water price, house price)
  → Tạo formula: POST /api/formulas

  --- Nếu dùng group ---
  → Gán price và formula vào group:
        POST /api/groups/:id/prices
        POST /api/groups/:id/formula
  → Thêm house vào group:
        PATCH /api/houses/:id { group_id }

  --- Nếu không dùng group (cấu hình riêng từng phòng) ---
  → Gán price và formula trực tiếp vào house:
        POST /api/houses/:id/prices
        POST /api/houses/:id/formula

  → Tạo buyer:   POST /api/users { role: 3 }
  → Gán buyer vào house (tạo tenancy):
        POST /api/houses/:id/tenants
```

### 6.4 Cuối kỳ — Ghi usage

Có 2 nguồn tạo usage:

**Buyer tự ghi:**

```
buyer đăng nhập
  → POST /api/usages
    {
      house_id, meter_id,
      current_reading, reading_photo_urls[],
      timestamp
    }
  → status = 'pending'
  → Seller nhận thông báo (in-app/email)
```

**Seller tự ghi thay:**

```
seller đăng nhập
  → POST /api/usages (với recorded_by=seller)
  → status = 'pending' (hoặc tự approve luôn)
```

### 6.5 Seller review usage → Approve / Discard

```
seller mở màn hình quản lý house
  → GET /api/houses/:id/usages?billing_cycle=YYYY-MM&status=pending
  → Với từng usage:
      PATCH /api/usages/:id  { action: 'approve' }
        → status = 'approved', approved_by, approved_at ghi lại
        → ghi audit_log
      PATCH /api/usages/:id  { action: 'discard' }
        → status = 'discarded'
        → ghi audit_log
```

### 6.6 Xuất bill

```
seller bấm "Xuất bill"
  → POST /api/bills
    { house_id, billing_cycle }

Server thực hiện:
  1. Kiểm tra chưa có bill cho (house_id, billing_cycle) — nếu có rồi trả 409
  2. Lấy tất cả usage với status='approved' của house trong billing_cycle
  3. Tính consumption cho mỗi usage:
       consumption = current_reading - previous_reading
  4. Áp price tương ứng (electric_price × electric_consumption, ...)
  5. Chạy formula để tổng hợp line_items
  6. Tính subtotal, tax, total_amount
  7. Lưu bill với status='draft'
  8. Render bill ra hình ảnh/PDF → lưu export_url
  9. Cập nhật bill.status = 'issued'
  10. Ghi audit_log

Response: bill object kèm export_url
Buyer nhận thông báo bill mới
```

### 6.7 Thanh toán

```
buyer thanh toán (in-app hoặc ngoài app)
  → POST /api/payments { bill_id, amount, method }
  → payment.status = 'pending' → 'paid'
  → bill.status cập nhật = 'paid' khi tổng payment đủ total_amount
```

---

## 7. API Endpoints tổng hợp

### Auth

| Method | Path                        | Auth | Mô tả                  |
| ------ | --------------------------- | ---- | ---------------------- |
| POST   | `/api/auth/login`           | ✗    | Đăng nhập, nhận JWT    |
| POST   | `/api/auth/refresh`         | ✓    | Refresh access token   |
| POST   | `/api/auth/logout`          | ✓    | Đăng xuất              |
| POST   | `/api/auth/forgot-password` | ✗    | Yêu cầu reset password |
| POST   | `/api/auth/reset-password`  | ✗    | Đặt lại password       |

### Users

| Method | Path             | Auth | Permission                           |
| ------ | ---------------- | ---- | ------------------------------------ |
| POST   | `/api/users`     | ✓    | admin → seller/buyer; seller → buyer |
| GET    | `/api/users/:id` | ✓    | Xem thông tin user                   |
| PATCH  | `/api/users/:id` | ✓    | Cập nhật thông tin user              |

### Groups

| Method | Path                      | Auth | Actor        |
| ------ | ------------------------- | ---- | ------------ |
| POST   | `/api/groups`             | ✓    | seller/admin |
| GET    | `/api/groups`             | ✓    | seller (own) |
| GET    | `/api/groups/:id`         | ✓    | seller/admin |
| PATCH  | `/api/groups/:id`         | ✓    | seller/admin |
| POST   | `/api/groups/:id/prices`  | ✓    | seller       |
| POST   | `/api/groups/:id/formula` | ✓    | seller       |

### Houses

| Method | Path                      | Auth | Actor                                  |
| ------ | ------------------------- | ---- | -------------------------------------- |
| POST   | `/api/houses`             | ✓    | seller/admin                           |
| GET    | `/api/houses`             | ✓    | seller (own)                           |
| GET    | `/api/houses/:id`         | ✓    | seller/buyer                           |
| PATCH  | `/api/houses/:id`         | ✓    | seller/admin                           |
| POST   | `/api/houses/:id/meters`  | ✓    | seller                                 |
| POST   | `/api/houses/:id/prices`  | ✓    | seller (override khi không dùng group) |
| POST   | `/api/houses/:id/formula` | ✓    | seller (override khi không dùng group) |
| POST   | `/api/houses/:id/tenants` | ✓    | seller                                 |

### Bảng giá (Prices) & Formulas

| Method | Path              | Auth | Actor        | Ghi chú                                  |
| ------ | ----------------- | ---- | ------------ | ---------------------------------------- |
| POST   | `/api/prices`     | ✓    | seller/admin | Tạo bảng giá; `seller_id` tự động từ JWT |
| GET    | `/api/prices`     | ✓    | seller (own) | Chỉ trả bảng giá của seller đang gọi     |
| GET    | `/api/prices/:id` | ✓    | seller/admin | Xem chi tiết 1 bảng giá                  |
| PATCH  | `/api/prices/:id` | ✓    | seller/admin | Cập nhật bảng giá                        |
| POST   | `/api/formulas`   | ✓    | seller/admin |                                          |
| GET    | `/api/formulas`   | ✓    | seller (own) |                                          |

### Usages

| Method | Path              | Auth | Actor                    |
| ------ | ----------------- | ---- | ------------------------ |
| POST   | `/api/usages`     | ✓    | buyer/seller             |
| GET    | `/api/usages`     | ✓    | seller (filter house)    |
| PATCH  | `/api/usages/:id` | ✓    | seller (approve/discard) |

### Bills

| Method | Path             | Auth | Actor        |
| ------ | ---------------- | ---- | ------------ |
| POST   | `/api/bills`     | ✓    | seller/admin |
| GET    | `/api/bills`     | ✓    | seller/buyer |
| GET    | `/api/bills/:id` | ✓    | seller/buyer |

### Payments

| Method | Path            | Auth | Actor        |
| ------ | --------------- | ---- | ------------ |
| POST   | `/api/payments` | ✓    | buyer        |
| GET    | `/api/payments` | ✓    | seller/buyer |

---

## 8. HTTP Status Codes chuẩn

| Code | Tình huống                                       |
| ---- | ------------------------------------------------ |
| 200  | Thành công (GET, PATCH, login)                   |
| 201  | Tạo mới thành công (POST tạo resource)           |
| 400  | Request không hợp lệ (thiếu field, sai format)   |
| 401  | Chưa xác thực hoặc sai credentials               |
| 403  | Đã xác thực nhưng không đủ quyền                 |
| 404  | Resource không tồn tại                           |
| 409  | Conflict (email trùng, bill đã tồn tại trong kỳ) |
| 500  | Lỗi server                                       |

---

## 9. Quy tắc nghiệp vụ quan trọng (Business Rules)

1. **Mỗi `(house, billing_cycle)` chỉ có tối đa 1 bill** — enforce bằng UNIQUE constraint ở DB và kiểm tra ở service layer.
2. **Không bao giờ tạo admin qua API** — role `1` bị cấm ở mọi endpoint tạo user, kể cả admin gọi.
3. **Password không bao giờ được trả về** trong bất kỳ response nào.
4. **Login error không phân biệt** email/password sai — cùng một thông báo `401` để tránh user enumeration.
5. **Usage phải được approve** trước khi đưa vào tính bill. Usage `discarded` bị bỏ qua hoàn toàn.
6. **Audit log bắt buộc** cho: login success/failed, approve/discard usage, tạo/xuất bill, tạo user.
7. **Seller chỉ thấy data của mình** — house, price, formula, usage, bill đều scoped theo `seller_id`.
8. **Buyer chỉ thấy data liên quan** — chỉ xem bill và usage của house mình đang thuê.
9. **Formula + Price phải được gán** (qua group hoặc trực tiếp) trước khi xuất bill — nếu thiếu thì trả lỗi rõ ràng.
10. **Seed script idempotent** — chạy nhiều lần không tạo trùng admin.
11. **Một house chỉ thuộc tối đa 1 group** — `group_id` là nullable FK, không phải mảng.
12. **Price/formula của group ưu tiên hơn per-house assignment** — khi house thuộc group, billing engine dùng config của group.

---

## 10. Cấu hình môi trường (.env)

```env
# Database
DATABASE_URL=postgresql://user:password@localhost:5432/nhatro

# JWT
JWT_SECRET=your-secret-key
JWT_ACCESS_TOKEN_EXPIRATION=15m
JWT_REFRESH_TOKEN_EXPIRATION=7d

# App
PORT=3000
NODE_ENV=development
```

---

## 11. Setup & Chạy project

```bash
# Cài dependencies
pnpm install

# Apply migrations
pnpm migrate

# Tạo admin account lần đầu
pnpm seed
# → admin@nhatro.com / Admin@123

# Chạy development server
pnpm dev
```

> **Sau khi seed**, đổi password admin ngay qua `POST /api/auth/reset-password` hoặc UI.

---

## 12. Sơ đồ quan hệ (tóm tắt)

```
developer ──provisions──▶ admin(1)
admin(1) ─────creates────▶ seller(N)
seller(1) ────creates────▶ buyer(N)
seller(1) ────creates────▶ group(N)
seller(1) ────manages────▶ house(N)
group(1) ────contains───▶ house(N)  [house.group_id]
house(1) ─────has──────▶ meter(N)
seller(1) ────creates────▶ price(N)
seller(1) ────creates────▶ formula(N)
group ◄─assigned─────── price (N:M via group_price)   ← ưu tiên hơn
group ◄─assigned─────── formula (N:M via group_formula)
house ◄─assigned─────── price (N:M via house_price)    ← override khi không dùng group
house ◄─assigned─────── formula (N:M via house_formula)
buyer(N) ────tenancy────▶ house(N)  [via tenancy table]
meter(1) ─────has────────▶ usage(N)
usage(N) ────(approved)──▶ bill(1) line_items  [per billing_cycle]
bill(1) ──────has────────▶ payment(N)
all actions ─────────────▶ audit_log
```
