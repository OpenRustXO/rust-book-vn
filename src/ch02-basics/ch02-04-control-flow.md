# Luồng điều khiển

## if là một expression

`if` không cần đặt điều kiện trong ngoặc, nhưng bắt buộc điều kiện phải là `bool` — khác C, nơi `0`
được coi là false và mọi giá trị khác 0 được coi là true. Viết `if number { ... }` với `number` là
số nguyên chứ không phải `bool` là lỗi biên dịch ngay, không đợi tới lúc chạy mới lộ ra hành vi sai.

Vì `if`/`else` là một expression (giống khối `{}` ở phần Hàm), có thể dùng trực tiếp ở vế phải của
`let`:

```rust
fn main() {
    let dieu_kien = true;
    let so = if dieu_kien { 5 } else { 6 };
    println!("Giá trị của số: {so}");
}
```

Vì cả hai nhánh phải cùng góp phần tạo ra giá trị của cùng một expression, **kiểu trả về của mọi
nhánh `if`/`else if`/`else` phải giống nhau** — trộn `{ 5 }` và `{ "sáu" }` là lỗi biên dịch, vì
trình biên dịch không thể biết `so` sẽ mang kiểu gì.

## Lặp với loop, while, for

**`loop`** lặp vô hạn cho tới khi gặp `break` tường minh. Vì `loop` cũng là một expression, `break`
có thể mang theo giá trị — đó chính là kết quả của cả vòng lặp:

```rust
fn main() {
    let mut dem = 0;

    let ket_qua = loop {
        dem += 1;
        if dem == 10 {
            break dem * 2;
        }
    };

    println!("Kết quả: {ket_qua}");
}
```

Khi lồng nhiều `loop`, `break`/`continue` mặc định chỉ tác động vòng lặp trong cùng. Gắn **nhãn**
(`'nhan:`) trước một vòng lặp để `break`/`continue` từ bên trong nhắm đúng vòng lặp ngoài:

```rust
fn main() {
    let mut dem = 0;
    'dem_ngoai: loop {
        let mut dem_trong = 10;
        loop {
            if dem_trong == 9 {
                break;
            }
            if dem == 2 {
                break 'dem_ngoai;
            }
            dem_trong -= 1;
        }
        dem += 1;
    }
    println!("Kết thúc với dem = {dem}");
}
```

**`while`** lặp khi điều kiện còn đúng — hợp lý khi điều kiện dừng không tự nhiên là "đã duyệt hết
một tập giá trị" mà là một điều kiện tuỳ ý.

**`for`** lặp qua từng phần tử của một collection, là cách lặp thành ngữ và phổ biến nhất trong Rust
— an toàn hơn dùng `while` với chỉ số thủ công (không có rủi ro tính sai điều kiện dừng gây truy cập
ngoài phạm vi mảng), và không có chi phí kiểm tra biên ở mỗi vòng như cách lặp bằng chỉ số:

```rust
fn main() {
    let mang = [10, 20, 30, 40, 50];

    for phan_tu in mang {
        println!("Giá trị: {phan_tu}");
    }

    for so in (1..4).rev() {
        println!("{so}!");
    }
    println!("Bắt đầu!");
}
```

`(1..4)` tạo một `Range` từ 1 đến 3 (không bao gồm 4), `.rev()` đảo chiều thứ tự duyệt. `for` xuất
hiện lại xuyên suốt cuốn sách này, đặc biệt gắn liền với `Iterator` — trait sẽ được nói kỹ ở
[chương Closure và Iterator](../ch11-functional/ch11-02-iterators.md).
