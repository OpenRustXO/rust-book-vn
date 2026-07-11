# Publish crate lên crates.io

## Doc comment

`///` viết tài liệu cho item **ngay bên dưới** nó, hỗ trợ cú pháp Markdown, hiển thị đẹp khi chạy
`cargo doc --open` (sinh trang HTML tài liệu, mở luôn trong trình duyệt):

````rust
/// Cộng thêm 1 vào số truyền vào.
///
/// # Examples
///
/// ```
/// let ket_qua = ch12::cong_mot(5);
/// assert_eq!(ket_qua, 6);
/// ```
pub fn cong_mot(x: i32) -> i32 {
    x + 1
}
````

Các mục thường gặp trong doc comment, đánh dấu bằng heading Markdown: `# Examples` (ví dụ minh hoạ
cách gọi), `# Panics` (những tình huống hàm sẽ panic), `# Errors` (ý nghĩa từng loại lỗi nếu hàm trả
`Result`), `# Safety` (điều kiện phải đảm bảo trước khi gọi, bắt buộc có với hàm `unsafe`, nói ở
[chương Tính năng nâng cao](../ch18-advanced/ch18-01-unsafe-rust.md)).

Khối code trong ví dụ ở `# Examples` **không chỉ là minh hoạ chết** — `cargo test` tự động biên dịch
và chạy mọi khối code trong doc comment như một test riêng (gọi là doctest). Tài liệu sai lệch với
code thật sẽ khiến `cargo test` đỏ, không thể âm thầm để tài liệu lỗi thời.

`//!` khác `///` ở chỗ nó viết tài liệu cho item **chứa** nó (thường dùng ở đầu `src/lib.rs` để mô
tả tổng quan cả crate), không phải item phía dưới:

```rust,ignore
//! Crate này cung cấp các hàm tiện ích xử lý số.

/// Cộng thêm 1 vào số truyền vào.
pub fn cong_mot(x: i32) -> i32 {
    x + 1
}
```

## Chuẩn bị publish

Trước khi `cargo publish`, `Cargo.toml` cần đủ metadata bắt buộc:

```toml
[package]
name = "ten-crate-cua-ban"
version = "0.1.0"
edition = "2024"
description = "Mô tả ngắn gọn crate làm gì"
license = "MIT OR Apache-2.0"
```

Đăng nhập bằng API token lấy từ trang crates.io (`cargo login <token>`), rồi:

```bash
cargo publish
```

## Phiên bản đã đăng là vĩnh viễn

Một phiên bản đã `cargo publish` **không thể xoá hay ghi đè** — đây là đảm bảo quan trọng cho mọi dự
án khác đang phụ thuộc vào đúng phiên bản đó: `Cargo.lock` của họ trỏ tới một mã nguồn cố định,
không bao giờ bị đổi ngầm dưới chân. Phát hiện lỗi sau khi publish thì phát hành bản vá ở phiên bản
mới hơn.

Nếu một phiên bản có vấn đề nghiêm trọng cần ngăn dự án **mới** phụ thuộc vào (nhưng không muốn phá
vỡ các dự án **đã** khoá cứng phiên bản đó), dùng `cargo yank` thay vì xoá:

```bash
cargo yank --vers 1.0.1
```

`yank` không xoá gì — chỉ ngăn các dự án **mới** chọn phiên bản đó khi thêm dependency lần đầu; dự
án đã có `Cargo.lock` khoá đúng phiên bản này từ trước vẫn build bình thường.
