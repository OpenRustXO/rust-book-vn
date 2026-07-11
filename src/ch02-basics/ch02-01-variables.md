# Khái niệm lập trình cơ bản

Chương này đi qua những khối xây dựng nhỏ nhất của một chương trình Rust: cách khai báo giá trị,
kiểu dữ liệu có sẵn, cách gói code thành hàm, và cách rẽ nhánh/lặp. Đây là những khái niệm phổ quát
ở hầu hết ngôn ngữ lập trình, nên phần này chỉ tập trung vào chỗ Rust khác biệt hoặc có quy tắc
riêng, không giảng lại từ đầu khái niệm "biến là gì", "hàm là gì".

## Biến và tính khả biến

Trong Rust, biến khai báo bằng `let` mặc định là **bất biến** — một khi đã gán giá trị, không thể
gán lại:

```rust,compile_fail
fn main() {
    let x = 5;
    println!("Giá trị của x là: {x}");
    x = 6;
    println!("Giá trị của x là: {x}");
}
```

Biên dịch đoạn trên báo lỗi ngay tại dòng `x = 6`: gán lại một biến bất biến. Đây không phải giới
hạn ngôn ngữ mà là lựa chọn thiết kế có chủ đích: khi một giá trị được khai báo bất biến, cả người
đọc code lẫn trình biên dịch đều biết chắc nó sẽ không đổi ở bất kỳ đâu về sau, không cần dò theo
toàn bộ hàm để xác nhận. Lợi ích này càng rõ khi nhiều phần code cùng thấy một giá trị — tính bất
biến loại bỏ hẳn khả năng một phần code âm thầm thay đổi thứ mà phần khác đang dựa vào.

Muốn một biến có thể đổi giá trị, khai báo rõ bằng `mut`:

```rust
fn main() {
    let mut x = 5;
    println!("Giá trị của x là: {x}");
    x = 6;
    println!("Giá trị của x là: {x}");
}
```

`mut` không chỉ là công tắc bật/tắt lỗi biên dịch — nó là tín hiệu đọc-được cho biết "giá trị này sẽ
đổi", giúp người đọc code khoanh vùng đúng chỗ cần chú ý khi tìm lỗi liên quan tới trạng thái.

### Hằng số

`const` cũng khai báo một giá trị không đổi, nhưng khác `let` (không có `mut`) ở ba điểm: luôn phải
chú thích kiểu tường minh, giá trị phải tính được ngay tại thời điểm biên dịch (không thể là kết quả
một lời gọi hàm chỉ biết lúc chạy), và có thể khai báo ở phạm vi toàn cục (ngoài mọi hàm), trong khi
`let` chỉ hợp lệ bên trong một hàm hoặc block. Theo quy ước, tên hằng số viết hoa toàn bộ, các từ
cách nhau bằng `_`:

```rust
const SO_GIAY_MOI_GIO: u32 = 60 * 60;
```

### Shadowing

Khai báo một `let` mới cùng tên với biến đã có sẽ **shadowing** (che khuất) biến cũ — từ điểm đó trở
đi, tên đó trỏ tới giá trị mới:

```rust
fn main() {
    let x = 5;
    let x = x + 1;

    {
        let x = x * 2;
        println!("Giá trị của x trong scope trong: {x}");
    }

    println!("Giá trị của x: {x}");
}
```

Shadowing khác `mut` ở bản chất: mỗi `let` tạo ra một biến hoàn toàn mới (vẫn bất biến theo mặc
định), không phải ghi đè giá trị của biến cũ. Nhờ vậy shadowing cho phép đổi cả kiểu dữ liệu giữa
các bước biến đổi — điều `mut` không cho phép, vì một biến `mut` phải giữ nguyên kiểu suốt vòng đời
của nó:

```rust
fn main() {
    let spaces = "   ";
    let spaces = spaces.len();
    println!("Số khoảng trắng: {spaces}");
}
```

Ở đây `spaces` đi từ kiểu `&str` sang `usize` qua hai lần `let` — hợp lệ vì đây là hai biến khác
nhau tình cờ trùng tên. Nếu thử làm điều này với `mut spaces` thay vì shadowing, trình biên dịch sẽ
báo lỗi sai kiểu ngay tại phép gán.
