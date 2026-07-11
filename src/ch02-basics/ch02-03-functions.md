# Hàm

Hàm khai báo bằng `fn`, tên theo quy ước snake_case (chữ thường, cách nhau bằng `_`). Vị trí khai
báo không quan trọng — có thể gọi một hàm trước khi nó được định nghĩa ở phía dưới trong cùng phạm
vi, miễn trình biên dịch thấy được định nghĩa đó ở đâu đó:

```rust
fn main() {
    println!("Hello, world!");
    chao(5);
}

fn chao(x: i32) {
    println!("Giá trị truyền vào: {x}");
}
```

Khác với biến cục bộ (thường suy luận được kiểu), **tham số hàm luôn phải chú thích kiểu tường
minh**. Đây là lựa chọn thiết kế có chủ đích: chữ ký hàm trở thành một dạng tài liệu — chỉ cần đọc
`fn chao(x: i32)` là biết ngay hàm nhận gì, không cần suy luận ngược từ phần thân hàm hay từ nơi
gọi.

## Statement và expression

Phần thân hàm là một chuỗi **statement**, có thể kết thúc bằng một **expression**. Phân biệt hai
khái niệm này là chìa khoá để hiểu vì sao Rust viết được những đoạn code như dưới đây mà không cần
`return`:

- **Statement** thực hiện một hành động nhưng không trả về giá trị. `let y = 6;` là statement — bản
  thân phép gán không phải là một giá trị, nên không thể viết `let x = (let y = 6);`.
- **Expression** được tính toán ra một giá trị. `5 + 6`, một lời gọi hàm, hay một khối `{}` đều là
  expression.

Điểm dễ nhầm nhất: một khối `{}` cũng là expression, và giá trị của nó là dòng cuối cùng bên trong —
**miễn dòng đó không có dấu `;`**. Thêm `;` biến một expression thành statement, huỷ luôn giá trị
của nó:

```rust
fn main() {
    let y = {
        let x = 3;
        x + 1
    };

    println!("Giá trị của y: {y}");
}
```

`x + 1` không có dấu `;` nên là giá trị của cả khối, gán được cho `y`. Nếu viết `x + 1;` thì khối đó
trả về `()` (unit) thay vì `4`.

## Giá trị trả về

Kiểu trả về khai báo sau `->`. Giá trị trả về chính là expression cuối cùng trong thân hàm — không
cần `return` ở cuối hàm, và cũng theo đúng quy tắc "không dấu `;`" vừa nói ở trên vì thân hàm cũng
là một khối:

```rust
fn cong_mot(x: i32) -> i32 {
    x + 1
}

fn main() {
    let x = cong_mot(5);
    println!("Giá trị của x: {x}");
}
```

Vẫn có thể `return` sớm khi cần thoát hàm trước khi chạy tới dòng cuối, nhưng ở dòng cuối cùng, một
expression trần (không `return`, không `;`) là cách viết thành ngữ Rust dùng phổ biến nhất.
