# Struct

`struct` gom nhiều giá trị liên quan thành một kiểu có tên, giống tuple ở chương trước nhưng mỗi
phần tử có **tên field** riêng — không cần nhớ thứ tự để biết `.0` là gì, `.1` là gì.

## Định nghĩa và khởi tạo struct

```rust
struct NguoiDung {
    ten_dang_nhap: String,
    email: String,
    so_lan_dang_nhap: u64,
    dang_hoat_dong: bool,
}

fn main() {
    let user1 = NguoiDung {
        ten_dang_nhap: String::from("tuanem"),
        email: String::from("tuanem@example.com"),
        so_lan_dang_nhap: 1,
        dang_hoat_dong: true,
    };

    println!("{}", user1.email);
}
```

Field của một instance đã tạo được truy cập bằng dấu chấm (`user1.email`). Nếu instance được khai
báo `mut`, có thể gán lại field bằng chính cú pháp đó — lưu ý tính khả biến áp dụng cho **toàn bộ**
instance, Rust không cho khai báo chỉ một field là `mut` trong khi các field khác bất biến.

Khi tên tham số trùng tên field, có thể viết gọn (field init shorthand) thay vì lặp lại
`field: field`:

```rust
struct NguoiDung {
    ten_dang_nhap: String,
    email: String,
    so_lan_dang_nhap: u64,
    dang_hoat_dong: bool,
}

fn tao_nguoi_dung(ten_dang_nhap: String, email: String) -> NguoiDung {
    NguoiDung {
        ten_dang_nhap,
        email,
        so_lan_dang_nhap: 1,
        dang_hoat_dong: true,
    }
}

fn main() {
    let user1 = tao_nguoi_dung(String::from("tuanem"), String::from("tuanem@example.com"));
    println!("{}", user1.ten_dang_nhap);
}
```

### Struct update syntax

Tạo một instance mới phần lớn giống một instance đã có, chỉ khác vài field, dùng `..instance_cu` ở
cuối để lấy các field còn lại từ instance đó:

```rust
struct NguoiDung {
    ten_dang_nhap: String,
    email: String,
    so_lan_dang_nhap: u64,
    dang_hoat_dong: bool,
}

fn main() {
    let user1 = NguoiDung {
        ten_dang_nhap: String::from("tuanem"),
        email: String::from("tuanem@example.com"),
        so_lan_dang_nhap: 1,
        dang_hoat_dong: true,
    };

    let user2 = NguoiDung {
        email: String::from("khac@example.com"),
        ..user1
    };

    println!("{}", user2.ten_dang_nhap);
}
```

Cú pháp này chỉ là gán field, nên tuân theo đúng quy tắc move/copy đã học ở chương Quyền sở hữu:
`ten_dang_nhap` là `String` (không phải `Copy`), nên field đó bị move từ `user1` sang `user2` —
`user1` vẫn dùng được các field khác, nhưng `user1.ten_dang_nhap` thì không, vì phần đó đã chuyển
quyền sở hữu sang `user2`.

## Tuple struct

Khi tên field sẽ chỉ là thừa thãi (thứ tự đã đủ nói lên ý nghĩa), dùng tuple struct — struct có tên
kiểu nhưng field không tên, truy cập bằng chỉ số như tuple:

```rust
struct Diem(i32, i32, i32);

fn main() {
    let goc = Diem(0, 0, 0);
    println!("{} {} {}", goc.0, goc.1, goc.2);
}
```

`Diem(i32, i32, i32)` và một tuple `(i32, i32, i32)` trần không thể dùng thay cho nhau dù cấu trúc
dữ liệu bên dưới giống hệt — `struct Diem` là một kiểu riêng biệt, một hàm khai báo nhận `Diem` sẽ
từ chối một tuple `(i32, i32, i32)` thường, giúp trình biên dịch phân biệt hai khái niệm dù về mặt
bit là như nhau.

## Unit-like struct

Một struct không có field nào cả:

```rust
struct DanhDauDaXuLy;

fn main() {
    let _danh_dau = DanhDauDaXuLy;
}
```

Hữu ích khi cần một kiểu để implement một trait (chương sau) nhưng bản thân kiểu đó không cần lưu dữ
liệu gì — bản thân sự tồn tại của instance đã mang đủ ý nghĩa.

## In một struct để debug

Chú thích `#[derive(Debug)]` phía trên struct để có thể in toàn bộ instance bằng specifier `{:?}`
(hoặc `{:#?}` để in xuống dòng, dễ đọc hơn với struct nhiều field) — hữu ích khi debug, không dùng
`{}` vì `{}` yêu cầu kiểu implement `Display`, trait dành cho định dạng hiển thị cho người dùng cuối
chứ không phải để debug:

```rust
#[derive(Debug)]
struct HinhChuNhat {
    rong: u32,
    cao: u32,
}

fn main() {
    let hcn = HinhChuNhat {
        rong: 30,
        cao: 50,
    };

    println!("{hcn:?}");
    println!("{hcn:#?}");
}
```
