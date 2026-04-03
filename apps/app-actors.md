Định nghĩa tác nhân (entities) và mối quan hệ

admin:

- Trường: id, email, full_name (tuỳ), phone (tuỳ), created_at
- Mối quan hệ: 1 admin quản lý N seller

Note: `admin` accounts are provisioned by developer (developer will create/administer initial admin accounts). Record provenance with `created_by` (developer id) and `provisioned_at` when applicable.

seller:

- Trường: id, email, full_name, phone, address (tuỳ), verification_status, created_at
- Mối quan hệ: 1 seller quản lý N house; 1 seller tạo N price; 1 seller tạo N formula; 1 seller có thể tạo/approve N usage và issue N bill

buyer:

- Trường: id, email, full_name, phone, identification (tuỳ), created_at
- Mối quan hệ: 1 buyer có thể thuê N house (historical); 1 buyer tạo N usage; 1 buyer có N payment
- Ghi chú: có thể để `active_house_id` để biểu thị house đang thuê hiện tại

group:

- Trường: id, seller_id, name, type (1=house_group, 2=user_group), created_at, updated_at
- Mối quan hệ: 1 seller có N group; 1 group (type=1) chứa N house; 1 group có thể gán N price; 1 group có thể gán 1 formula
- Ghi chú: `type` cho phép tái sử dụng API group cho nhiều mục đích; hiện tại chỉ `type=1` được hỗ trợ. `price` và `formula` gán vào `group` được kế thừa bởi tất cả `house` trong group đó

house:

- Trường: id, seller_id, group_id (tuỳ, FK → group), name, address, unit (phòng/số), area, status (active/vacant/maintenance), deposit, metadata, created_at
- Mối quan hệ: 1 house thuộc 1 seller; 1 house thuộc tối đa 1 group; 1 house có N meter; 1 house có N bill (mỗi billing_cycle tối đa 1 bill); 1 house có N historical tenants (buyer)

meter:

- Trường: id, house_id, meter_type (electric/water), external_meter_id, installed_at, metadata
- Mối quan hệ: 1 house có N meter; 1 meter có N usage

price:

- Trường: id, seller_id, name, price_type (electric/water/house), unit (kWh/m3/month), unit_price (decimal), currency, effective_from, effective_to, metadata
- Mối quan hệ: 1 seller có N price; 1 price có thể gán vào N house

formula:

- Trường: id, seller_id, name, expression (DSL hoặc JSON rules), parameters, description, created_at
- Mối quan hệ: 1 seller có N formula; 1 formula có thể gán vào N house

usage (electricity_usage / water_usage):

- Trường: id, house_id, meter_id, recorded_by (buyer|seller), recorded_by_id, current_reading, previous_reading (tuỳ), reading_photo_urls[], timestamp, status (pending/approved/discarded), approved_by, approved_at, notes
- Mối quan hệ: 1 usage thuộc 1 house và 1 meter; 1 usage có thể dẫn tới line items trong 1 bill khi được approve

bill:

- Trường: id, house_id, billing_cycle (YYYY-MM), issued_by (seller/admin), issued_at, line_items [{description, quantity, unit, unit_price, amount}], subtotal, tax, total_amount, status (draft/issued/paid/void), export_url (PDF/PNG), metadata
- Ràng buộc: mỗi (house, billing_cycle) chỉ có tối đa 1 bill
- Mối quan hệ: 1 bill liên quan tới N usage (qua line_items) và N payment

payment:

- Trường: id, bill_id, buyer_id, amount, currency, method, status (pending/paid/failed/refunded), transaction_id, paid_at
- Mối quan hệ: 1 bill có N payment (reconciliation)

audit_log / activity:

- Trường: id, actor_type (admin/seller/buyer/system), actor_id, action (approve/discard/export/create/update), target_type, target_id, timestamp, metadata

Authentication fields for user-like actors (admin/seller/buyer): `password_hash`, `auth_provider`, `last_login`, `is_active`, `two_factor_enabled` (tuỳ).

Ghi chú về cardinality & hành vi:

- admin(1) — seller(N)
- seller(1) — house(N)
- house(1) — meter(N)
- house(1) — bill(N) (but unique per billing_cycle)
- buyer(N) — house(N) (historical many-to-many) ; consider `tenancy` table to track periods: tenancy {buyer_id, house_id, start_date, end_date, role}
- price và formula được tạo theo seller và gán vào house (many-to-many via assignment object)

Các entity nên có trường `metadata` và `created_at/updated_at` chuẩn để mở rộng nhanh.

Kế tiếp: nếu bạn muốn, tôi có thể chuyển các phần trên thành schema JSON mẫu hoặc ER-diagram, hoặc chia thành ticket kỹ thuật (title + mô tả + acceptance criteria) và lưu vào `apps/technical-features.md`.
