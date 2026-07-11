# Tổ chức dự án với package, crate, module

Khi một dự án lớn dần, gom hết code vào một file trở nên khó quản lý. Rust có ba khái niệm tổ chức
code lồng nhau, từ lớn tới nhỏ: **package**, **crate**, và **module**. Chương này chỉ nói cách tổ
chức code trong cùng một dự án — chia sẻ code giữa các dự án khác nhau (đăng package lên crates.io)
sẽ nói ở [chương Cargo nâng cao](../ch12-cargo-advanced/ch12-01-release-profiles.md).

## Crate

**Crate** là đơn vị biên dịch nhỏ nhất mà trình biên dịch Rust xử lý trong một lần gọi `rustc` — có
hai dạng:

- **Binary crate**: có hàm `main`, biên dịch ra một file thực thi được.
- **Library crate**: không có `main`, chứa chức năng để crate khác dùng lại (đây cũng là nghĩa
  thường gặp của từ "crate" khi nói tới một thư viện Rust công khai, ví dụ trên crates.io).

Trình biên dịch bắt đầu đọc một crate từ một file gọi là **crate root** — với binary crate là
`src/main.rs`, với library crate là `src/lib.rs`. Nội dung của file gốc này tạo thành module có tên
`crate`, đứng đầu toàn bộ cây module của crate đó (nói kỹ ở phần sau).

## Package

**Package** là một hay nhiều crate cùng cung cấp một nhóm chức năng, mô tả bởi file `Cargo.toml` ở
thư mục gốc — khai báo tên, phiên bản, dependency. Một package chứa **tối đa một** library crate,
nhưng có thể chứa **nhiều** binary crate:

- `src/main.rs` (nếu có) → một binary crate cùng tên với package.
- `src/lib.rs` (nếu có) → library crate cùng tên với package.
- Mỗi file trong `src/bin/` → thêm một binary crate riêng.

`cargo new ten_du_an` tạo một package mới với `Cargo.toml` và `src/main.rs` — một package chỉ có
đúng một binary crate, cấu trúc phổ biến nhất cho một chương trình độc lập.
