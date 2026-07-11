# Deref và Drop

Hai trait làm nên phần lớn "phép màu" khiến con trỏ thông minh dùng được tự nhiên như tham chiếu
thường: `Deref` (dùng `*` để lấy ra giá trị bên trong) và `Drop` (tự chạy dọn dẹp khi ra khỏi
scope).

## Deref

```rust
fn main() {
    let x = 5;
    let y = Box::new(x);

    assert_eq!(5, x);
    assert_eq!(5, *y);
}
```

`*y` hoạt động được vì `Box<T>` implement trait `Deref`. Viết một con trỏ thông minh của riêng mình
để thấy rõ cơ chế:

```rust
use std::ops::Deref;

struct HopCuaToi<T>(T);

impl<T> HopCuaToi<T> {
    fn new(x: T) -> HopCuaToi<T> {
        HopCuaToi(x)
    }
}

impl<T> Deref for HopCuaToi<T> {
    type Target = T;

    fn deref(&self) -> &T {
        &self.0
    }
}

fn main() {
    let x = 5;
    let y = HopCuaToi::new(x);

    assert_eq!(5, *y);
}
```

`*y` thực chất được trình biên dịch khai triển thành `*(y.deref())` — gọi `deref()` để lấy một tham
chiếu `&T`, rồi giải tham chiếu tham chiếu đó. Chỉ cần implement đúng một method `deref`, toàn bộ cú
pháp `*` tự động hoạt động theo.

### Deref coercion

Trình biên dịch tự động chèn thêm lời gọi `deref()` khi truyền một `&HopCuaToi<T>` vào chỗ đang cần
`&T` (hoặc `&U` nếu `T` cũng implement `Deref` sang `U`, áp dụng liên tiếp nhiều bước) — gọi là
**deref coercion**:

```rust
use std::ops::Deref;

struct HopCuaToi<T>(T);

impl<T> Deref for HopCuaToi<T> {
    type Target = T;

    fn deref(&self) -> &T {
        &self.0
    }
}

fn chao(ten: &str) {
    println!("Xin chào, {ten}!");
}

fn main() {
    let m = HopCuaToi(String::from("Rust"));
    chao(&m);
}
```

`&m` có kiểu `&HopCuaToi<String>`, `chao` cần `&str` — trình biên dịch tự chèn hai bước deref
(`&HopCuaToi<String>` → `&String` → `&str`) để khớp, không cần người gọi tự viết `&(*m)[..]`. Đây là
lý do vì sao các hàm nhận `&str` gọi được cả với `&String` lẫn `&Box<String>` mà không cần ép kiểu
tường minh ở nơi gọi.

## Drop

`Drop` chạy code dọn dẹp đúng một lần, tự động, khi giá trị ra khỏi scope — không cần gọi thủ công:

```rust
struct ThongBaoKhiDrop {
    du_lieu: String,
}

impl Drop for ThongBaoKhiDrop {
    fn drop(&mut self) {
        println!("Đang dọn dẹp `{}`!", self.du_lieu);
    }
}

fn main() {
    let _a = ThongBaoKhiDrop {
        du_lieu: String::from("cái đầu tiên"),
    };
    let _b = ThongBaoKhiDrop {
        du_lieu: String::from("cái thứ hai"),
    };
    println!("Đã tạo xong các instance.");
}
```

Giá trị bị drop theo thứ tự **ngược** với thứ tự tạo (`_b` trước `_a`) — giống ngăn xếp, phần tử vào
sau ra trước.

Không thể tự tay gọi `gia_tri.drop()` — trình biên dịch cấm gọi trực tiếp method này, vì nếu cho
phép, giá trị đó vẫn sẽ bị drop **thêm một lần nữa** tự động khi hết scope, gây double free. Cần
drop sớm chủ động (ví dụ để giải phóng một khoá trước khi scope kết thúc), gọi hàm tự do
`std::mem::drop(gia_tri)` — nhận quyền sở hữu giá trị rồi thả ngay, không đụng tới method `drop` của
trait.
