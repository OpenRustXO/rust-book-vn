# Công cụ phát triển hữu ích

Ba công cụ sau đều cài được qua `rustup component add`, không cần thêm gì ngoài rustup đã cài ở
[chương Bắt đầu](../ch01-install/ch01-01-getting-started.md):

```bash
rustup component add rustfmt clippy rust-analyzer
```

## rustfmt

Format code Rust theo một chuẩn thống nhất — loại bỏ hẳn tranh cãi về style cá nhân (đặt ngoặc chỗ
nào, thụt lề bao nhiêu):

```bash
cargo fmt         # format toàn bộ crate
cargo fmt --check # chỉ kiểm tra, không sửa — dùng trong CI
```

## Clippy

Bộ lint mở rộng, bắt được nhiều lỗi và cách viết chưa tối ưu mà `rustc` thường không cảnh báo — ví
dụ dùng `.clone()` thừa, so sánh số thực dấu phẩy động bằng `==`, dùng vòng lặp thủ công thay vì
iterator adaptor sẵn có:

```bash
cargo clippy
```

Đáng chạy thường xuyên bên cạnh `cargo build` — nhiều gợi ý của Clippy phản ánh đúng thành ngữ
(idiom) mà cộng đồng Rust coi là cách viết chuẩn, không chỉ là sở thích cá nhân.

## rust-analyzer

Language server cho Rust — cắm vào editor (VS Code, Neovim, ...) để có gợi ý kiểu ngay khi gõ, nhảy
tới định nghĩa, tự động sửa lỗi nhỏ, xem tài liệu inline. Hầu hết editor hiện đại hỗ trợ qua
extension riêng (ví dụ extension "rust-analyzer" chính thức trên VS Code) — extension đó tự tìm và
dùng binary đã cài qua `rustup component add` ở trên.

## cargo doc

Sinh tài liệu HTML từ doc comment (`///`, `//!`, chương Cargo nâng cao) cho crate hiện tại và toàn
bộ dependency:

```bash
cargo doc --open
```

## cargo-binstall

Đã giới thiệu ở chương Bắt đầu — cài nhanh các công cụ dòng lệnh viết bằng Rust (kể cả các công cụ
khác không nằm trong danh sách này) bằng binary dựng sẵn thay vì compile từ mã nguồn.
