# Con trỏ thông minh

**Con trỏ thông minh** (smart pointer) là một struct hoạt động như con trỏ (trỏ tới dữ liệu ở chỗ
khác) nhưng mang thêm khả năng — thường implement `Deref` (dùng được như tham chiếu thường) và
`Drop` (tự dọn dẹp khi ra khỏi scope). `String` và `Vec<T>` thực chất cũng là con trỏ thông minh
theo định nghĩa này; chương này tập trung vào ba cái quan trọng nhất chưa gặp: `Box<T>`, `Rc<T>`,
`RefCell<T>`.

## Box\<T\>

`Box<T>` là con trỏ thông minh đơn giản nhất: đặt một giá trị lên heap, giữ một con trỏ tới đó trên
stack — một chủ sở hữu duy nhất, không thêm chi phí nào khác ngoài một lượt gián tiếp qua con trỏ.

```rust
fn main() {
    let b = Box::new(5);
    println!("b = {b}");
}
```

### Kiểu đệ quy cần Box

Một enum tự chứa chính nó (không qua con trỏ) khiến trình biên dịch **không thể tính được kích thước
cố định** — kích thước của kiểu đó phụ thuộc vào chính nó, đệ quy vô hạn:

```rust,compile_fail
enum DanhSach {
    Node(i32, DanhSach),
    Rong,
}

fn main() {}
```

`Box<T>` có kích thước cố định (đúng bằng kích thước một con trỏ) bất kể `T` là gì — thay `DanhSach`
trực tiếp bằng `Box<DanhSach>` phá vỡ chuỗi đệ quy vô hạn về kích thước, vì giờ mỗi `Node` chỉ cần
chứa một con trỏ cỡ cố định, không phải toàn bộ phần còn lại của danh sách nằm inline:

```rust
enum DanhSach {
    Node(i32, Box<DanhSach>),
    Rong,
}

use DanhSach::{Node, Rong};

fn main() {
    let ds = Node(1, Box::new(Node(2, Box::new(Node(3, Box::new(Rong))))));

    let mut hien_tai = &ds;
    while let Node(gia_tri, tiep_theo) = hien_tai {
        println!("{gia_tri}");
        hien_tai = tiep_theo;
    }
}
```

`Box<T>` cũng hữu ích khi cần chuyển quyền sở hữu một giá trị lớn mà không muốn copy toàn bộ dữ liệu
— move một `Box<T>` chỉ move con trỏ (rẻ), dữ liệu thật trên heap không hề bị di chuyển.
