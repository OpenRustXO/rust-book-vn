# Xử lý lỗi

Rust chia lỗi thành hai nhóm, xử lý bằng hai cơ chế khác hẳn nhau: lỗi **có thể phục hồi**
(recoverable) — hợp lý để báo cho người gọi biết rồi để họ quyết định tiếp tục thế nào, dùng
`Result<T, E>`; và lỗi **không thể phục hồi** (unrecoverable) — dấu hiệu một bug hoặc một trạng thái
không còn cách nào xử lý tiếp an toàn, dùng `panic!`.

## panic! và lỗi không thể phục hồi

`panic!` in ra thông báo lỗi, dọn dẹp (unwind) ngăn xếp gọi hàm, rồi thoát chương trình:

```rust,should_panic
fn main() {
    panic!("gặp sự cố nghiêm trọng");
}
```

Phần lớn thời gian panic không tới từ gọi macro trực tiếp mà từ một thao tác mà bản thân nó đã panic
khi gặp điều kiện bất thường — chỉ số vượt quá độ dài mảng (đã gặp ở chương Collection) là ví dụ
điển hình nhất.

Mặc định, đặt biến môi trường `RUST_BACKTRACE=1` khi chạy chương trình để in ra backtrace — danh
sách mọi hàm đã gọi tới điểm panic, giúp xác định chính xác chuỗi lời gọi gây ra lỗi, không chỉ điểm
panic cuối cùng.

### unwind hay abort

Khi panic, Rust mặc định **unwind**: đi ngược lên từng hàm trong ngăn xếp gọi, dọn dẹp dữ liệu của
từng hàm đó (chạy các logic `drop` tương ứng) trước khi thoát — an toàn nhưng tốn công sức dọn dẹp.
Có thể chuyển sang **abort**: thoát ngay lập tức, để hệ điều hành thu hồi toàn bộ bộ nhớ, không dọn
dẹp gì — nhị phân nhỏ hơn, thoát nhanh hơn, đổi lại không có cơ hội chạy logic dọn dẹp tuỳ chỉnh
nào. Cấu hình trong `Cargo.toml`:

```toml
[profile.release]
panic = "abort"
```
