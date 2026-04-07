Định nghĩa tác nhân (entities) và mối quan hệ

admin:

- Trường: id, username, full_name (tuỳ), phone (tuỳ), created_at
- Mối quan hệ: 1 admin quản lý N seller

Note: `admin` accounts are provisioned by developer — developer manually creates the initial admin account using a username and password provided by the admin. Record provenance with `created_by` (developer id) and `provisioned_at` when applicable.

seller:

- Trường: id, username, email (tuỳ), full_name, phone, address (tuỳ), created_at
- Mối quan hệ: 1 seller tạo N buyer; 1 seller tạo N variable; 1 seller tạo N formula; 1 seller tạo N usage record; 1 seller tạo N sheet config

buyer:

- Trường: id, username, email (tuỳ), full_name, phone, formula_id (FK → formula, tuỳ — override formula từ sheet config), electricity_baseline, water_baseline, people_count, room_price, baseline_month (YYYY-MM), created_at
- Lưu ý: `people_count` và `room_price` có thể thay đổi theo thời gian; giá trị tại thời điểm tạo bill phải được snapshot vào `bill.line_items` để đảm bảo tính bất biến lịch sử
- Mối quan hệ: 1 buyer thuộc 1 seller; 1 buyer có thể được gán 1 formula (override, tuỳ); 1 buyer gửi N submission; 1 buyer có N bill (tối đa 1 bill mỗi billing_cycle)
- Ghi chú: buyer đại diện cho 1 đơn vị tính bill (ví dụ 1 phòng, 1 hộ). Seller có thể đặt tên buyer tuỳ ý (ví dụ "Phòng A", "Nhà chị Lan").
- Ràng buộc: `full_name` phải unique trong cùng 1 seller (không cho phép 2 buyer cùng tên thuộc 1 seller).

formula:

- Trường: id, seller*id (FK → seller), name, expression (cây biểu thức — expression tree — lưu dạng JSON. Mỗi node có type: "number" | "variable" | "formula" | "if_else" | "operation". Ví dụ: operation(+, variable(Số điện đã sử dụng) * 2500, if*else(condition: variable(Số người) > 2, then: variable(Số nước đã sử dụng) * 14000, else: 12000))), formula_refs[] (danh sách formula_id được tham chiếu — dùng để kiểm tra circular reference), created_at, updated_at
- Ràng buộc: `name` phải unique trong cùng 1 seller; `formula_refs[]` không được tạo circular reference (formula không được tham chiếu trực tiếp hoặc gián tiếp đến chính nó).
- Mối quan hệ: 1 seller có N formula; 1 formula được gán cho N buyer (override); 1 formula được dùng trong N sheet config

variable:

- Trường: id, seller_id (FK → seller), name, description (tuỳ), created_at
- Ràng buộc: `name` phải unique trong cùng 1 seller (không cho phép 2 variable cùng tên thuộc 1 seller).
- Mối quan hệ: 1 seller có N variable; 1 variable được tham chiếu trong N formula; 1 variable được gắn trong N sheet config (qua variable_bindings)

usage_record:

- Trường: id, seller_id (FK → seller), name, link_token, status (open/closed), created_at
- Ràng buộc: `name` phải unique trong cùng 1 seller.
- Mối quan hệ: 1 seller tạo N usage record; 1 usage record nhận N submission từ các buyer; 1 usage record được dùng làm old_record hoặc new_record trong N sheet config

submission:

- Trường: id, usage_record_id (FK → usage_record), buyer_id (FK → buyer), electricity_current, water_current, photo_urls[], submitted_at, status (pending/approved/discarded/manual), approved_by, approved_at, notes
- Mối quan hệ: 1 submission thuộc 1 usage record và 1 buyer; khi approved hoặc manual, giá trị được lưu vào usage record của buyer

sheet_config:

- Trường: id, seller_id (FK → seller), name, old_record_id (FK → usage_record), new_record_id (FK → usage_record), formula_id (FK → formula, tuỳ — formula mặc định áp cho buyer chưa có override), variable_bindings [{variable_id (FK → variable), binding_type (electricity_used|water_used|people_count|room_price)}], created_at
- Mối quan hệ: 1 seller tạo N sheet config; 1 sheet config gắn với 1 cặp usage record (old + new); 1 sheet config có thể gắn 1 formula mặc định; 1 sheet config được dùng để tạo N bill

bill:

- Trường: id, buyer_id (FK → buyer), sheet_config_id (FK → sheet_config), billing_cycle (YYYY-MM), issued_by (seller/admin), issued_at, line_items [{description, variable, value, amount}], total_amount, status (draft/issued/paid/void), export_url (PDF/PNG), metadata
- Lưu ý: `line_items` snapshot giá trị `people_count` và `room_price` của buyer tại thời điểm tạo bill — không tham chiếu về sau để đảm bảo tính bất biến lịch sử
- Ràng buộc: mỗi (buyer_id, billing_cycle) chỉ có tối đa 1 bill
- Mối quan hệ: 1 bill thuộc 1 buyer; 1 bill được tạo từ 1 sheet config; 1 bill có N payment

payment:

- Trường: id, bill_id, buyer_id, amount, currency, method, status (pending/paid/failed/refunded), transaction_id, paid_at
- Mối quan hệ: 1 bill có N payment (reconciliation)

audit_log / activity:

- Trường: id, actor_type (admin/seller/buyer/system), actor_id, action (approve/discard/export/create/update), target_type, target_id, timestamp, metadata

Authentication fields for user-like actors (admin/seller/buyer): `password_hash`, `last_login`, `is_active`.

Ghi chú về cardinality & hành vi:

- admin(1) — seller(N)
- seller(1) — buyer(N)
- seller(1) — variable(N)
- seller(1) — formula(N)
- seller(1) — usage_record(N)
- seller(1) — sheet_config(N)
- buyer(1) — formula(0..1) (override tuỳ chọn)
- buyer(1) — submission(N) (tối đa 1 submission per usage record)
- buyer(1) — bill(N) (unique per billing_cycle)
- sheet_config(1) — usage_record_old(1) + usage_record_new(1)
- sheet_config(1) — bill(N)

Các entity nên có trường `metadata` và `created_at/updated_at` chuẩn để mở rộng nhanh.
