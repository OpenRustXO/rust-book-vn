# Enum và khớp mẫu

Struct gom nhiều giá trị liên quan lại thành một; **enum** diễn tả điều ngược lại — một giá trị chỉ
có thể là **một trong số** các khả năng đã liệt kê, không phải cả một tập giá trị cùng lúc.

## Định nghĩa enum

```rust
enum DiaChiIp {
    V4,
    V6,
}

fn main() {
    let phien_ban_bon = DiaChiIp::V4;
    let phien_ban_sau = DiaChiIp::V6;
}
```

Một địa chỉ IP chỉ có thể là bản 4 hoặc bản 6, không bao giờ là cả hai — enum diễn tả đúng ràng buộc
"loại trừ lẫn nhau" đó ngay trong hệ thống kiểu, thay vì phải tự quản lý bằng một cờ boolean hay một
số nguyên tuỳ ý và hy vọng không dùng sai.

Mỗi biến thể (variant) có thể mang theo dữ liệu riêng, kể cả khác kiểu nhau giữa các variant — điều
struct không làm được vì một struct chỉ có một tập field cố định:

```rust
enum DiaChiIp {
    V4(u8, u8, u8, u8),
    V6(String),
}

fn main() {
    let nha = DiaChiIp::V4(127, 0, 0, 1);
    let vong_lap = DiaChiIp::V6(String::from("::1"));
}
```

Dữ liệu đi thẳng vào variant, không cần một struct riêng rồi bọc enum bên ngoài trỏ tới struct đó.
Variant còn có thể mang dữ liệu có cấu trúc như một struct thu nhỏ (named field) hoặc như tuple:

```rust
enum ThongDiep {
    Thoat,
    DiChuyen { x: i32, y: i32 },
    Ghi(String),
    DoiMau(i32, i32, i32),
}

fn main() {
    let cac_thong_diep = [
        ThongDiep::Thoat,
        ThongDiep::DiChuyen { x: 10, y: 20 },
        ThongDiep::Ghi(String::from("xin chào")),
        ThongDiep::DoiMau(255, 0, 0),
    ];
}
```

Bốn variant này, nếu viết bằng struct, sẽ là bốn kiểu struct hoàn toàn khác nhau (`Thoat`,
`DiChuyen`, `Ghi`, `DoiMau`) — enum gói chúng vào chung một kiểu `ThongDiep`, cho phép viết một hàm
nhận tham số kiểu `ThongDiep` xử lý được cả bốn trường hợp, thay vì phải viết bốn hàm hoặc dùng
trait object.

Giống struct, enum cũng nhận method qua `impl`:

```rust
enum ThongDiep {
    Thoat,
    Ghi(String),
}

impl ThongDiep {
    fn mo_ta(&self) -> &str {
        match self {
            ThongDiep::Thoat => "thoát",
            ThongDiep::Ghi(_) => "ghi dữ liệu",
        }
    }
}

fn main() {
    let td = ThongDiep::Ghi(String::from("log"));
    println!("{}", td.mo_ta());
}
```

(`match` dùng ở đây sẽ được giải thích kỹ ở phần tiếp theo.)

## Option: thay thế cho null

Rust không có `null`. Thay vào đó, thư viện chuẩn định nghĩa enum `Option<T>` để diễn tả "có giá trị
hoặc không":

```rust
enum Option<T> {
    None,
    Some(T),
}
```

(`Option<T>` đã có sẵn trong prelude, không cần khai báo lại hay đưa `Option::` vào scope tường minh
— dùng thẳng `Some`/`None`.)

Vấn đề `null` từng gây ra vô số lỗi runtime kinh điển: bất kỳ giá trị nào cũng có thể ngầm là
`null`, và trình biên dịch không bắt buộc kiểm tra trước khi dùng — lỗi chỉ lộ ra lúc chạy, đúng chỗ
code cố dùng giá trị không tồn tại. `Option<T>` sửa vấn đề đó bằng cách tách hẳn "có giá trị kiểu
`T`" (`Some(T)`) khỏi "không có giá trị" (`None`) thành hai biến thể của cùng một kiểu — một `T`
trần và một `Option<T>` là hai kiểu khác nhau, nên trình biên dịch **từ chối biên dịch** nếu dùng
`Option<T>` như thể chắc chắn có giá trị mà chưa xử lý trường hợp `None`:

```rust,compile_fail
fn main() {
    let so: Option<i8> = Some(5);
    let x: i8 = 5;

    let tong = x + so; // lỗi: không thể cộng i8 với Option<i8>
}
```

Muốn lấy giá trị `T` bên trong một `Option<T>`, phải xử lý tường minh cả hai khả năng — thường bằng
`match` (phần tiếp theo) hoặc các method như `unwrap_or`, `map`. Chi phí gõ thêm vài dòng xử lý đổi
lấy một đảm bảo chắc chắn: không thể quên kiểm tra "giá trị có tồn tại không" ở một chỗ nào đó rồi
để lỗi lộ ra lúc chạy — trình biên dịch không cho qua nếu thiếu.
