# Tổ chức test

Rust phân biệt hai loại test theo phạm vi: **unit test** — nhỏ, tập trung, có thể chạm cả vào code
riêng tư; và **integration test** — chạy từ bên ngoài, chỉ thấy đúng API công khai như một người
dùng thật của crate.

## Unit test

Quy ước: đặt unit test trong một module con tên `tests` ngay trong **cùng file** với code đang test,
đánh dấu `#[cfg(test)]` phía trên module đó:

```rust
pub fn cong_hai(a: i32) -> i32 {
    them_hai_noi_bo(a)
}

fn them_hai_noi_bo(a: i32) -> i32 {
    a + 2
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn no_hoat_dong() {
        assert_eq!(4, cong_hai(2));
        assert_eq!(5, them_hai_noi_bo(3)); // gọi được cả hàm riêng tư
    }
}
```

`#[cfg(test)]` báo trình biên dịch: chỉ biên dịch module này khi build ở chế độ test (`cargo test`),
loại hẳn khỏi build thường (`cargo build`) — code test không hề làm phình to hay chậm binary
release. `use super::*;` đưa toàn bộ item của module cha (kể cả các item riêng tư) vào scope của
`tests` — hợp lệ vì `tests` là module **con** của module chứa code đang test, và (nhắc lại quy tắc
riêng tư ở chương Module) con luôn nhìn thấy được item riêng tư của cha. Đây là lý do unit test gọi
thẳng được `them_hai_noi_bo` dù hàm đó không `pub` — kiểm tra được cả chi tiết cài đặt bên trong,
không chỉ API công khai.

## Integration test

Test tích hợp nằm trong thư mục `tests/` ở gốc dự án (ngang hàng `src/`), **không phải** một module
con của crate — mỗi file trong `tests/` được Cargo biên dịch thành một **crate riêng biệt**, chỉ
`use` được phần `pub` của thư viện, giống hệt góc nhìn của một người dùng bên ngoài gọi crate:

```text
ten_du_an/
├── Cargo.toml
├── src/
│   └── lib.rs
└── tests/
    └── kiem_tra_tich_hop.rs
```

```rust,ignore
// tests/kiem_tra_tich_hop.rs
use ten_du_an::cong_hai;

#[test]
fn no_hoat_dong_tu_ben_ngoai() {
    assert_eq!(4, cong_hai(2));
}
```

`cargo test` tự động chạy cả unit test lẫn mọi file trong `tests/`, không cần khai báo gì thêm.

Chỉ **library crate** (có `src/lib.rs`) mới có `tests/` — một binary crate thuần (chỉ có
`src/main.rs`) không có gì để `tests/` `use` vào, vì code trong `src/main.rs` không phải API công
khai của ai cả. Vì lý do đó, quy ước phổ biến trong các dự án Rust thực tế: đặt phần lớn logic vào
`src/lib.rs`, để `src/main.rs` chỉ còn là một lớp mỏng gọi vào thư viện đó — nhờ vậy toàn bộ logic
thật sự đều test tích hợp được, `main.rs` chỉ còn vài dòng không cần test riêng.

### Chia sẻ code dùng chung giữa các file test

Một file thường (`tests/common.rs`) bị Cargo hiểu nhầm là một file test độc lập, chạy như một crate
test riêng dù không chứa `#[test]` nào — để đặt code dùng chung (hàm setup, dữ liệu mẫu) mà không bị
Cargo coi là một bộ test, đặt trong `tests/common/mod.rs` (dùng quy ước module cũ hơn):

```text
tests/
├── common/
│   └── mod.rs
└── kiem_tra_tich_hop.rs
```

Cargo không coi các file bên trong thư mục con (`tests/common/`) là một crate test cấp cao — chỉ các
file nằm **trực tiếp** trong `tests/` mới được coi vậy.
