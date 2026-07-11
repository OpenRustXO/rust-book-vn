# HashMap

`HashMap<K, V>` lưu cặp khoá-giá trị, tra cứu theo khoá thay vì theo chỉ số liên tiếp như `Vec`.
Khác `Vec`/`String`, `HashMap` không có trong prelude — phải đưa vào scope tường minh:

```rust
use std::collections::HashMap;

fn main() {
    let mut diem: HashMap<String, i32> = HashMap::new();

    diem.insert(String::from("Đội Xanh"), 10);
    diem.insert(String::from("Đội Đỏ"), 50);

    let ten = String::from("Đội Xanh");
    let diem_doi_xanh = diem.get(&ten).copied().unwrap_or(0);
    println!("{diem_doi_xanh}");
}
```

`get` trả về `Option<&V>` — `None` khi khoá không tồn tại, cùng triết lý với `Vec::get`: việc tra
cứu không thấy là một khả năng bình thường, không phải lỗi để panic. `.copied()` chuyển
`Option<&i32>` thành `Option<i32>` (vì `i32` là `Copy`), `.unwrap_or(0)` cho một giá trị mặc định
khi là `None`.

Duyệt qua toàn bộ cặp bằng `for`, thứ tự **không đảm bảo** giống lần chèn (khác `Vec`, HashMap không
giữ thứ tự):

```rust
use std::collections::HashMap;

fn main() {
    let mut diem = HashMap::new();
    diem.insert(String::from("Đội Xanh"), 10);
    diem.insert(String::from("Đội Đỏ"), 50);

    for (ten, gia_tri) in &diem {
        println!("{ten}: {gia_tri}");
    }
}
```

## Quyền sở hữu khi insert

Với kiểu implement `Copy` (như `i32`), giá trị được copy vào map. Với kiểu sở hữu dữ liệu trên heap
(như `String`), `insert` **move** giá trị vào map — quy tắc move/copy quen thuộc từ chương Quyền sở
hữu áp dụng y hệt, chỉ khác chỗ đích là bên trong một `HashMap` thay vì một biến:

```rust,compile_fail
use std::collections::HashMap;

fn main() {
    let ten_doi = String::from("Đội Xanh");
    let mut diem = HashMap::new();
    diem.insert(ten_doi, 10);

    println!("{ten_doi}"); // lỗi: ten_doi đã bị move vào diem
}
```

## Cập nhật giá trị

Ba tình huống thường gặp: ghi đè, chỉ ghi nếu khoá chưa có giá trị, và cập nhật dựa trên giá trị cũ.

Ghi đè đơn giản là gọi `insert` lại với cùng khoá — giá trị cũ bị thay thế. Chỉ ghi khi khoá **chưa
có** dùng `entry`, tránh phải tự kiểm tra `contains_key` rồi mới `insert` (dễ viết sai, và phải tra
cứu khoá hai lần):

```rust
use std::collections::HashMap;

fn main() {
    let mut diem = HashMap::new();
    diem.insert(String::from("Xanh"), 10);

    diem.entry(String::from("Xanh")).or_insert(50); // đã có, không đổi
    diem.entry(String::from("Vàng")).or_insert(50); // chưa có, chèn 50

    println!("{diem:?}");
}
```

Cập nhật dựa trên giá trị cũ — ví dụ đếm số lần xuất hiện của mỗi từ trong một câu — kết hợp
`entry`/`or_insert` với việc mượn khả biến giá trị trả về:

```rust
use std::collections::HashMap;

fn main() {
    let van_ban = "con mèo ngồi trên thảm con chó ngồi dưới bàn";
    let mut dem = HashMap::new();

    for tu in van_ban.split_whitespace() {
        let so_dem = dem.entry(tu).or_insert(0);
        *so_dem += 1;
    }

    println!("{dem:?}");
}
```

`or_insert` trả về `&mut V` — một tham chiếu khả biến trỏ thẳng vào giá trị bên trong map (chèn `0`
trước nếu khoá chưa có), nên `*so_dem += 1` sửa trực tiếp giá trị nằm trong `HashMap`, không phải
một bản sao.
