# Refutability

Pattern chia làm hai loại theo khả năng **không khớp**:

- **Bất khả bác** (irrefutable): khớp với **mọi** giá trị có thể có, không bao giờ thất bại. Một tên
  biến trần (`x`) là bất khả bác — bất kỳ giá trị nào gán cho `let x = ...` cũng khớp.
- **Khả bác** (refutable): có ít nhất một giá trị khiến nó **không** khớp. `Some(x)` là khả bác — nó
  không khớp nếu giá trị thực tế là `None`.

`let`, tham số hàm, và `for` **chỉ chấp nhận pattern bất khả bác** — hợp lý, vì cả ba đều không có
cơ chế "làm gì đó nếu không khớp", nên một pattern có thể thất bại ở đó sẽ không biết phải xử lý ra
sao:

```rust,compile_fail
fn main() {
    let mot_so: Option<i32> = Some(5);
    let Some(x) = mot_so; // lỗi: `Some(x)` khả bác, mot_so có thể là None
    println!("{x}");
}
```

`if let`/`while let` ngược lại, **chỉ nhận pattern khả bác mới có ý nghĩa** — đó chính là lý do
chúng tồn tại: chạy một nhánh code khi khớp, bỏ qua khi không. Dùng chúng với một pattern bất khả
bác vẫn biên dịch được nhưng vô nghĩa (luôn khớp, không cần điều kiện), trình biên dịch cảnh báo
thay vì báo lỗi:

```rust
fn main() {
    let x = 5;

    if let x = x {
        println!("Pattern này luôn khớp: {x}");
    }
}
```

`let-else` (chương Enum) là cách dùng một pattern **khả bác** ở vị trí giống `let`, nhưng vẫn hợp lệ
nhờ buộc nhánh không khớp phải rẽ hẳn khỏi luồng chính (`return`/`break`/`panic!`) — nhờ vậy phần
còn lại của hàm vẫn luôn chạy với giả định pattern **đã khớp**, giữ được tính bất khả bác cho phần
code phía sau `let-else`, dù bản thân pattern viết ra là khả bác.
