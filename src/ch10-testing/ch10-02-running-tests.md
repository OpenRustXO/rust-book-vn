# Chạy test

`cargo test` biên dịch code ở chế độ test rồi chạy binary test sinh ra; các cờ sau dấu `--` được
truyền thẳng cho binary đó thay vì cho `cargo test` (`cargo test -- --cờ-của-binary`).

## Chạy song song và trạng thái chia sẻ

Mặc định, `cargo test` chạy **mỗi test trên một luồng riêng, song song** — nhanh hơn chạy tuần tự,
nhưng kéo theo một hệ quả dễ bị bỏ qua: nếu nhiều test cùng đọc/ghi một trạng thái chung (một file
trên đĩa, một biến môi trường, một thư mục làm việc), chúng có thể va vào nhau và cho kết quả sai
lệch tuỳ thứ tự chạy — đúng dạng lỗi khó dò vì không tái hiện được ổn định. Ép chạy tuần tự khi test
có chia sẻ trạng thái:

```bash
cargo test -- --test-threads=1
```

## Output của test

Theo mặc định, `cargo test` **ẩn** toàn bộ output (`println!`, ...) của một test **pass** — chỉ hiện
khi test đó fail, để kết quả chạy test gọn, không bị ngợp bởi log của hàng trăm test đều pass. Muốn
luôn thấy output kể cả khi pass:

```bash
cargo test -- --show-output
```

## Chạy một tập con test

Chạy test theo tên chính xác:

```bash
cargo test kiem_tra_cong
```

Hoặc theo một phần tên chung — mọi test có tên chứa chuỗi đó đều chạy, tiện khi đặt tên test có tiền
tố chung theo nhóm chức năng:

```bash
cargo test kiem_tra
```

## Bỏ qua test tốn thời gian

Đánh dấu `#[ignore]` cho test chạy lâu (tích hợp, benchmark thô) để `cargo test` bình thường bỏ qua,
chỉ chạy khi được yêu cầu rõ ràng:

```rust
#[test]
fn test_nhanh() {
    assert_eq!(2 + 2, 4);
}

#[test]
#[ignore]
fn test_cham() {
    // giả sử đây là một thao tác tốn nhiều thời gian
    assert_eq!(2 + 2, 4);
}
```

```bash
cargo test              # bỏ qua test_cham
cargo test -- --ignored # chỉ chạy các test đã đánh dấu ignore
```
