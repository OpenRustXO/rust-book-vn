# Cargo và crates.io nâng cao

Chương này đi qua các tính năng của Cargo phục vụ giai đoạn dự án đã lớn hơn một file `main.rs` đơn
lẻ: tuỳ chỉnh cách biên dịch, viết tài liệu, chia sẻ crate lên crates.io, và tổ chức nhiều crate
liên quan trong một workspace.

## Release profile

Cargo biên dịch theo hai **profile** có sẵn, mỗi profile một tập giá trị mặc định khác nhau:

- `dev` — dùng khi chạy `cargo build`/`cargo run` (không cờ gì thêm): ưu tiên biên dịch nhanh, chấp
  nhận binary chạy chậm hơn. Thuận tiện cho vòng lặp sửa-chạy-lại liên tục lúc phát triển.
- `release` — dùng khi thêm cờ `--release`: ưu tiên binary chạy nhanh, chấp nhận biên dịch lâu hơn
  (trình biên dịch bỏ nhiều công sức tối ưu hơn). Dùng khi build bản phát hành thật.

Ghi đè giá trị mặc định của từng profile trong `Cargo.toml`:

```toml
[profile.dev]
opt-level = 0

[profile.release]
opt-level = 3
```

`opt-level` (0-3) điều khiển mức độ tối ưu hoá — mặc định `dev` là 0 (tối ưu tối thiểu, biên dịch
nhanh nhất), `release` là 3 (tối ưu tối đa). Có thể chỉnh riêng, ví dụ tăng `opt-level` của `dev`
lên 1 khi code đang phát triển có đoạn tính toán nặng, cần chạy đủ nhanh để test trong lúc code vẫn
chưa hoàn thiện, mà chưa muốn trả chi phí biên dịch đầy đủ của `release`.
