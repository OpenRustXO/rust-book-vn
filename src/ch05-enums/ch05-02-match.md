# match

`match` so khớp một giá trị lần lượt với từng **pattern**, chạy code ở nhánh khớp đầu tiên. Khác
`if`, điều kiện không cần là `bool` — pattern có thể là một biến thể enum, một giá trị literal, một
khoảng, hay một cấu trúc để destructure dữ liệu ra ngay trong lúc so khớp.

```rust
enum DongTien {
    MotXu,
    NamXu,
    MuoiXu,
    HaiNamXu,
}

fn gia_tri_xu(dong_tien: DongTien) -> u8 {
    match dong_tien {
        DongTien::MotXu => 1,
        DongTien::NamXu => 5,
        DongTien::MuoiXu => 10,
        DongTien::HaiNamXu => 25,
    }
}

fn main() {
    println!("{}", gia_tri_xu(DongTien::MuoiXu));
}
```

Mỗi nhánh là một cặp `pattern => expression`. Giống `if`/`else`, `match` cũng là một expression —
giá trị của nhánh khớp chính là giá trị của cả `match`, nên `gia_tri_xu` không cần `return`.

## Destructure ngay trong pattern

Nếu variant mang dữ liệu, pattern có thể lấy luôn dữ liệu đó ra thành biến dùng được trong nhánh:

```rust
enum ThongDiep {
    DiChuyen { x: i32, y: i32 },
    Ghi(String),
}

fn xu_ly(td: ThongDiep) {
    match td {
        ThongDiep::DiChuyen { x, y } => {
            println!("Di chuyển tới ({x}, {y})");
        }
        ThongDiep::Ghi(noi_dung) => {
            println!("Ghi: {noi_dung}");
        }
    }
}

fn main() {
    xu_ly(ThongDiep::DiChuyen { x: 1, y: 2 });
    xu_ly(ThongDiep::Ghi(String::from("log")));
}
```

## match với Option

Xử lý `Option<T>` bằng `match` là cách thành ngữ để lấy giá trị `T` ra một cách an toàn:

```rust
fn cong_mot(x: Option<i32>) -> Option<i32> {
    match x {
        None => None,
        Some(i) => Some(i + 1),
    }
}

fn main() {
    let nam = Some(5);
    let sau = cong_mot(nam);
    let khong_gi = cong_mot(None);

    println!("{sau:?} {khong_gi:?}");
}
```

## match bắt buộc đầy đủ (exhaustive)

Trình biên dịch buộc một `match` phải liệt kê **hết mọi khả năng** của kiểu đang so khớp — thiếu một
biến thể là lỗi biên dịch, không phải bug ẩn tới lúc chạy mới lộ ra khi rơi vào đúng nhánh bị bỏ
sót:

```rust,compile_fail
fn cong_mot(x: Option<i32>) -> Option<i32> {
    match x {
        Some(i) => Some(i + 1),
        // thiếu nhánh None — lỗi: non-exhaustive patterns
    }
}
```

Đây là lý do `match` (chứ không phải chỉ `if`) là công cụ chính để xử lý enum trong Rust: mỗi lần
thêm một variant mới vào enum ở đâu đó trong codebase, mọi `match` cũ chưa xử lý variant đó sẽ báo
lỗi biên dịch ngay, chỉ thẳng những chỗ cần cập nhật — thay vì im lặng bỏ qua trường hợp mới.

Khi không cần liệt kê hết, dùng `_` để bắt mọi giá trị còn lại (hoặc một tên biến để vừa bắt vừa
dùng được giá trị đó):

```rust
fn mo_ta_xu(gia_tri: u8) -> &'static str {
    match gia_tri {
        1 => "một xu",
        5 => "năm xu",
        10 => "mười xu",
        khac => {
            println!("Không có mệnh giá {khac} xu tiêu chuẩn");
            "không xác định"
        }
    }
}

fn main() {
    println!("{}", mo_ta_xu(10));
    println!("{}", mo_ta_xu(2));
}
```

Nếu không cần dùng tới giá trị bắt được, thay `khac` bằng `_` để trình biên dịch không cảnh báo biến
không dùng tới.
