# Workspace

**Workspace** gom nhiều package cùng chia sẻ **một** `Cargo.lock` và **một** thư mục `target/` chung
— hữu ích khi một dự án tách thành nhiều crate liên quan (ví dụ một thư viện lõi và vài binary dùng
thư viện đó), thay vì quản lý rời rạc từng crate với dependency và bản build riêng.

## Cấu trúc

```text
workspace-cua-toi/
├── Cargo.toml
├── them_hai/
│   ├── Cargo.toml
│   └── src/
│       └── lib.rs
└── them_mot/
    ├── Cargo.toml
    └── src/
        └── main.rs
```

`Cargo.toml` ở gốc chỉ khai báo các thành viên, **không có** phần `[package]` riêng (gọi là virtual
manifest — bản thân thư mục gốc không phải một crate, chỉ là nơi gom nhóm):

```toml
[workspace]
members = ["them_hai", "them_mot"]
```

Mỗi thành viên vẫn có `Cargo.toml` riêng như một package bình thường. Muốn một thành viên dùng crate
khác trong cùng workspace, khai báo dependency qua đường dẫn tương đối:

```toml
[dependencies]
them_hai = { path = "../them_hai" }
```

## Lợi ích của Cargo.lock và target/ dùng chung

Toàn bộ workspace chia sẻ **một** `Cargo.lock` — mọi thành viên dùng cùng một phiên bản cho bất kỳ
dependency ngoài nào cả hai cùng cần, tránh tình huống hai crate trong cùng dự án âm thầm kéo hai
phiên bản khác nhau của cùng một thư viện (tốn công biên dịch hai lần, và có thể gây lỗi kiểu không
tương thích nếu thư viện đó có kiểu dữ liệu đi qua ranh giới giữa hai crate). Thư mục `target/` dùng
chung cũng có nghĩa dependency chung giữa các thành viên chỉ biên dịch **một lần**, không lặp lại
cho mỗi crate.

## Lệnh Cargo trong workspace

Chạy ở thư mục gốc workspace:

```bash
cargo build          # build tất cả thành viên
cargo test           # chạy test của tất cả thành viên
cargo run -p them_mot # chạy đúng binary của thành viên "them_mot"
```

Cờ `-p <tên-package>` (viết tắt của `--package`) nhắm một thành viên cụ thể — cần thiết khi
workspace có nhiều binary crate, vì Cargo không tự đoán được muốn chạy cái nào nếu không chỉ rõ.
