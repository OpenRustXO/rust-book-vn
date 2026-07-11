# Pattern nâng cao

**Pattern** (mẫu) là cú pháp để so khớp cấu trúc của một giá trị, đồng thời lấy phần cần thiết ra
thành biến. `match` (chương Enum) là nơi dùng pattern rõ ràng nhất, nhưng không phải nơi duy nhất —
chương này hệ thống lại toàn bộ chỗ pattern xuất hiện và các cú pháp pattern chưa gặp.

## Nơi dùng pattern

**`let` cũng là một pattern.** `let x = 5;` thực chất là so khớp giá trị `5` với pattern `x` —
pattern đơn giản nhất, một tên biến, luôn khớp với bất kỳ giá trị nào. Pattern phức tạp hơn
destructure ngay tại `let`:

```rust
fn main() {
    let (x, y, z) = (1, 2, 3);
    println!("{x} {y} {z}");
}
```

**Tham số hàm** cũng là pattern, destructure được ngay trong danh sách tham số:

```rust
fn in_toa_do(&(x, y): &(i32, i32)) {
    println!("Toạ độ: ({x}, {y})");
}

fn main() {
    let diem = (3, 5);
    in_toa_do(&diem);
}
```

**`for`** dùng pattern cho biến lặp — `(chi_so, gia_tri)` destructure trực tiếp tuple mà
`.enumerate()` trả ra:

```rust
fn main() {
    let v = vec!['a', 'b', 'c'];

    for (chi_so, gia_tri) in v.iter().enumerate() {
        println!("{chi_so}: {gia_tri}");
    }
}
```

`if let`/`while let` (đã gặp ở chương Enum) cũng dùng pattern, nhưng khác `let`/`for`/tham số hàm ở
một điểm quan trọng — nói ở phần tiếp theo.
