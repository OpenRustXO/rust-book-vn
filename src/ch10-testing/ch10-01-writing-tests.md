# Kiểm thử tự động

Rust coi test là một phần của ngôn ngữ, không phải công cụ ngoài gắn thêm vào: `#[test]` là một
attribute có sẵn, `cargo test` biên dịch và chạy mọi hàm đánh dấu attribute đó.

## Viết test

Một hàm trở thành test bằng cách thêm `#[test]` ngay phía trên:

```rust
pub fn cong(a: i32, b: i32) -> i32 {
    a + b
}

#[test]
fn kiem_tra_cong() {
    assert_eq!(4, cong(2, 2));
}
```

`cargo test` tìm mọi hàm `#[test]` trong crate, chạy từng hàm trong một luồng riêng. Test được coi
là **fail** nếu hàm panic (do một macro assert thất bại, hoặc panic thủ công) — không panic nghĩa là
**pass**.

## Các macro assert

- `assert!(dieu_kien)` — panic nếu `dieu_kien` là `false`.
- `assert_eq!(trai, phai)` — panic nếu hai vế không bằng nhau, in ra **cả hai giá trị** trong thông
  báo lỗi (yêu cầu cả hai kiểu implement `PartialEq` để so sánh và `Debug` để in ra khi fail).
- `assert_ne!(trai, phai)` — ngược lại, panic nếu hai vế **bằng nhau**.

```rust
#[derive(Debug, PartialEq)]
struct DiemSo {
    gia_tri: i32,
}

#[test]
fn diem_so_bang_nhau() {
    let a = DiemSo { gia_tri: 10 };
    let b = DiemSo { gia_tri: 10 };
    assert_eq!(a, b);
}
```

So với `assert!(a == b)`, `assert_eq!(a, b)` cho thông báo lỗi hữu ích hơn hẳn khi fail — in ra
chính xác hai giá trị khác nhau ở đâu, thay vì chỉ nói "điều kiện sai".

## Thông báo lỗi tuỳ chỉnh

Mọi macro assert nhận thêm tham số tuỳ chọn sau điều kiện chính, truyền thẳng vào `format!`:

```rust
pub fn chao(ten: &str) -> String {
    format!("Chào, {ten}!")
}

#[test]
fn loi_chao_chua_ten() {
    let ket_qua = chao("Mai");
    assert!(
        ket_qua.contains("Mai"),
        "Lời chào không chứa tên, giá trị thật sự là: `{ket_qua}`"
    );
}
```

## Kiểm tra panic với should_panic

`#[should_panic]` đảo ngược tiêu chí pass/fail: test **pass** khi hàm panic, **fail** nếu hàm chạy
xong không panic:

```rust
pub struct PhanTram {
    gia_tri: i32,
}

impl PhanTram {
    pub fn moi(gia_tri: i32) -> PhanTram {
        if !(0..=100).contains(&gia_tri) {
            panic!("Phần trăm phải trong khoảng 0-100, nhận được {gia_tri}");
        }
        PhanTram { gia_tri }
    }
}

#[test]
#[should_panic]
fn tao_voi_gia_tri_vuot_khoang_thi_panic() {
    PhanTram::moi(200);
}
```

`#[should_panic]` không kiểm tra **panic đó có phải panic mình mong đợi hay không** — bất kỳ panic
nào trong hàm cũng khiến test pass, kể cả một panic hoàn toàn không liên quan (ví dụ do bug khác gây
ra trước khi chạy tới đoạn logic thật sự muốn kiểm tra). Thêm `expected` để chỉ chấp nhận panic có
thông báo chứa đúng chuỗi mong đợi, thu hẹp lại đúng trường hợp cần test:

```rust,ignore
#[test]
#[should_panic(expected = "trong khoảng 0-100")]
fn tao_voi_gia_tri_vuot_khoang_thi_panic() {
    PhanTram::moi(200);
}
```

## Dùng Result trong test

Một test có thể trả `Result<(), E>` thay vì panic, cho phép dùng toán tử `?` ngay trong thân test —
tiện khi test gọi nhiều thao tác có thể lỗi và muốn lan lỗi ra thay vì `unwrap` từng bước:

```rust
#[test]
fn phan_tich_so_thanh_cong() -> Result<(), std::num::ParseIntError> {
    let so: i32 = "42".parse()?;
    assert_eq!(so, 42);
    Ok(())
}
```

Test fail khi hàm trả `Err`. Không thể kết hợp `#[should_panic]` với kiểu test trả `Result` — hai cơ
chế báo fail (panic vs `Err`) không tương thích nhau trong cùng một test.
