# Trả lời câu hỏi bổ sung — Buổi 9

### 1. Giá trị dẫn xuất (`errors` và `valid` ở `CheckoutScreen`)
`errors` và `valid` được suy ra trực tiếp từ `name`, `phone`, `address` và `items` trong mỗi lần render nên luôn đồng bộ tuyệt đối với dữ liệu hiện tại của form. Nếu lưu `errors` và `valid` bằng `useState` riêng, lập trình viên phải tự cập nhật lại chúng ở mọi nơi thay đổi `name`, `phone`, `address` hoặc giỏ hàng; chỉ cần quên cập nhật ở một chỗ là giao diện sẽ hiển thị thông báo lỗi sai hoặc mở/khoá nút "Xác nhận đặt hàng" không đúng thực tế.

### 2. Thứ tự gọi `clear()` trong `submit` của `CheckoutScreen`
Nếu gọi `clear()` trước `await createOrder(...)`, khi xảy ra lỗi mạng hoặc lỗi máy chủ, giỏ hàng của người dùng đã bị xoá sạch trong khi đơn hàng chưa hề được tạo trên server, khiến người dùng mất toàn bộ danh sách sách đã chọn. Đặt `clear()` ngay sau `await createOrder(...)` trong khối `try` đảm bảo chỉ khi server đã ghi nhận đơn hàng thành công thì ứng dụng mới xoá giỏ hàng; nếu `createOrder` ném lỗi, luồng nhảy thẳng xuống `catch` và giữ nguyên giỏ hàng để người dùng bấm thử lại.

### 3. `PUT` hay `PATCH` khi cập nhật hồ sơ
Nếu dùng `PUT /users/1` chỉ với body `{ name, phone }`, `json-server` sẽ thay thế toàn bộ bản ghi `user` bằng object mới, làm mất các trường quan trọng khác như `email` và `password` (chỉ còn lại `id`, `name`, `phone`). Bài thực hành chọn `PATCH /users/:id` vì `PATCH` chỉ cập nhật những trường được gửi lên và giữ nguyên các trường còn lại của người dùng trong `db.json`.

### 4. Cập nhật lạc quan (Optimistic Update) ở màn Thông báo
- **Nếu bỏ đoạn hoàn lại `read: false` trong `catch`:** Khi mất mạng, giao diện vẫn hiển thị thông báo là đã đọc (nền trắng, mất chấm đỏ) trong khi trên server vẫn là `read: false`; lần sau mở lại màn hình, thông báo đó lại hiện trạng thái chưa đọc khiến người dùng khó hiểu.
- **Chức năng nên dùng / không nên dùng:** Nên áp dụng cập nhật lạc quan cho các thao tác nhẹ, tần suất cao và ít rủi ro như **thả tim / bỏ yêu thích sách (`HeartButton`)** hoặc đánh dấu đã đọc thông báo. Không nên áp dụng cho các thao tác quan trọng cần xác nhận giao dịch như **Đặt hàng / Thanh toán (`Checkout`)** hoặc **Đăng nhập / Đăng ký**, vì phải chắc chắn server xử lý thành công mới được thông báo và chuyển bước.

### 5. Code bổ sung: Bộ lọc trạng thái ở màn Đơn hàng
Đã tích hợp trực tiếp thanh chip lọc trạng thái (`Tất cả` / `Chờ xác nhận` / `Đang giao` / `Hoàn tất` / `Đã huỷ`) vào `src/screens/OrdersScreen.tsx`:
```tsx
const STATUS_FILTERS: { key: "all" | OrderStatus; label: string }[] = [
  { key: "all", label: "Tất cả" },
  { key: "pending", label: "Chờ xác nhận" },
  { key: "shipping", label: "Đang giao" },
  { key: "completed", label: "Hoàn tất" },
  { key: "cancelled", label: "Đã huỷ" },
];

const [statusFilter, setStatusFilter] = useState<"all" | OrderStatus>("all");
const filteredOrders = orders.filter((o) => statusFilter === "all" || o.status === statusFilter);
```
