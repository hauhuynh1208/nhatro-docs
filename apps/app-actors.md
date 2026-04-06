Định nghĩa tác nhân (entities) và mối quan hệ

admin:

- Trường: id, username, full_name (tuỳ), phone (tuỳ), created_at
- Mối quan hệ: 1 admin quản lý N seller

Note: `admin` accounts are provisioned by developer — developer manually creates the initial admin account using a username and password provided by the admin. Record provenance with `created_by` (developer id) and `provisioned_at` when applicable.

seller:

- Trường: id, username, email (tuỳ), full_name, phone, address (tuỳ), created_at
- Mối quan hệ: 1 seller tạo N buyer; 1 seller tạo N formula; 1 seller tạo N container

buyer:

- Trường: id, username, email (tuỳ), full_name, phone, formula_id (FK → formula), electricity_baseline, water_baseline, people_count, baseline_month (YYYY-MM), created_at
- Mối quan hệ: 1 buyer thuộc 1 seller; 1 buyer được gán 1 formula; 1 buyer gửi N submission; 1 buyer có N bill (tối đa 1 bill mỗi billing_cycle)
- Ghi chú: buyer đại diện cho 1 đơn vị tính bill (ví dụ 1 phòng, 1 hộ). Seller có thể đặt tên buyer tuỳ ý (ví dụ "Phòng A", "Nhà chị Lan").

formula:

- Trường: id, seller_id (FK → seller), name, expression (biểu thức text có chứa {{variable}}, ví dụ: {{Số điện đã sử dụng}} * 2500 + {{Số nước đã sử dụng}} * 14000), created_at, updated_at
- Mối quan hệ: 1 seller có N formula; 1 formula được gán cho N buyer

container:

- Trường: id, seller_id (FK → seller), billing_cycle (YYYY-MM), link_token, status (open/closed), created_at
- Mối quan hệ: 1 seller tạo N container; 1 container nhận N submission từ các buyer

submission:

- Trường: id, container_id (FK → container), buyer_id (FK → buyer), electricity_current, water_current, photo_urls[], submitted_at, status (pending/approved/discarded), approved_by, approved_at, notes
- Mối quan hệ: 1 submission thuộc 1 container và 1 buyer; khi approved, giá trị này được dùng để tính bill cho buyer trong billing_cycle tương ứng

bill:

- Trường: id, buyer_id (FK → buyer), billing_cycle (YYYY-MM), issued_by (seller/admin), issued_at, line_items [{description, variable, value, amount}], total_amount, status (draft/issued/paid/void), export_url (PDF/PNG), metadata
- Ràng buộc: mỗi (buyer_id, billing_cycle) chỉ có tối đa 1 bill
- Mối quan hệ: 1 bill thuộc 1 buyer; 1 bill liên quan tới 1 submission (đã approved) và N payment

payment:

- Trường: id, bill_id, buyer_id, amount, currency, method, status (pending/paid/failed/refunded), transaction_id, paid_at
- Mối quan hệ: 1 bill có N payment (reconciliation)

audit_log / activity:

- Trường: id, actor_type (admin/seller/buyer/system), actor_id, action (approve/discard/export/create/update), target_type, target_id, timestamp, metadata

Authentication fields for user-like actors (admin/seller/buyer): `password_hash`, `last_login`, `is_active`.

Ghi chú về cardinality & hành vi:

- admin(1) — seller(N)
- seller(1) — buyer(N)
- seller(1) — formula(N)
- seller(1) — container(N)
- buyer(1) — formula(1) (gán 1 formula duy nhất)
- buyer(1) — submission(N) (tối đa 1 submission per container)
- buyer(1) — bill(N) (unique per billing_cycle)
- formula được tạo theo seller và gán trực tiếp vào từng buyer

Các entity nên có trường `metadata` và `created_at/updated_at` chuẩn để mở rộng nhanh.
