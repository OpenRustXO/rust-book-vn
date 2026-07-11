# Generic, Trait, Lifetime

Ba khái niệm của chương này giải quyết ba vấn đề khác nhau nhưng đều xoay quanh việc viết code dùng
lại được: **generic** tránh lặp code xử lý cùng logic trên nhiều kiểu cụ thể, **trait** định nghĩa
hành vi chung mà nhiều kiểu khác nhau có thể cùng implement, **lifetime** đảm bảo mọi tham chiếu
dùng lại vẫn luôn hợp lệ. Cả ba đều được trình biên dịch giải quyết tại thời điểm biên dịch — không
cái nào có chi phí lúc chạy.

## Generic

Xét hai hàm gần như giống hệt nhau, chỉ khác kiểu tham số:

```rust
fn so_lon_nhat_i32(danh_sach: &[i32]) -> &i32 {
    let mut lon_nhat = &danh_sach[0];
    for so in danh_sach {
        if so > lon_nhat {
            lon_nhat = so;
        }
    }
    lon_nhat
}

fn so_lon_nhat_char(danh_sach: &[char]) -> &char {
    let mut lon_nhat = &danh_sach[0];
    for ky_tu in danh_sach {
        if ky_tu > lon_nhat {
            lon_nhat = ky_tu;
        }
    }
    lon_nhat
}

fn main() {
    let so = vec![34, 50, 25, 100, 65];
    println!("{}", so_lon_nhat_i32(&so));
}
```

Phần thân hai hàm giống hệt nhau — chỉ kiểu `i32`/`char` khác. Generic cho viết một hàm duy nhất,
tham số hoá theo kiểu:

```rust
fn so_lon_nhat<T: PartialOrd + Copy>(danh_sach: &[T]) -> T {
    let mut lon_nhat = danh_sach[0];
    for &item in danh_sach {
        if item > lon_nhat {
            lon_nhat = item;
        }
    }
    lon_nhat
}

fn main() {
    let so = vec![34, 50, 25, 100, 65];
    println!("{}", so_lon_nhat(&so));

    let ky_tu = vec!['y', 'm', 'a', 'q'];
    println!("{}", so_lon_nhat(&ky_tu));
}
```

`T` là tham số kiểu, đứng trong `<>` ngay sau tên hàm — quy ước đặt tên một chữ hoa (`T`, `U`, ...)
khi không có ý nghĩa cụ thể hơn tên đó có thể gợi ra. `T: PartialOrd + Copy` là **ràng buộc trait**
(trait bound, nói kỹ ở phần Trait ngay sau đây): không phải `T` nhận **bất kỳ** kiểu nào, chỉ những
kiểu implement `PartialOrd` (so sánh được bằng `>`) và `Copy` (copy được thay vì move khi gán) mới
hợp lệ — thiếu ràng buộc, `so > lon_nhat` không biên dịch được vì trình biên dịch không có gì đảm
bảo `T` hỗ trợ phép so sánh đó.

### Generic trên struct và enum

```rust
struct Diem<T> {
    x: T,
    y: T,
}

fn main() {
    let nguyen = Diem { x: 5, y: 10 };
    let thuc = Diem { x: 1.0, y: 4.0 };

    println!("{} {}", nguyen.x, thuc.y);
}
```

Cả `x` và `y` cùng dùng một `T` — một instance phải có cả hai field cùng kiểu, viết
`Diem { x: 5, y: 4.0 }` không biên dịch được vì `5` là `i32` còn `4.0` là `f64`. Cần hai field khác
kiểu độc lập, khai báo hai tham số: `struct Diem<T, U> { x: T, y: U }`. `Option<T>` và
`Result<T, E>` đã gặp ở các chương trước chính là enum generic — không có gì đặc biệt hơn cách viết
ở đây.

### Method trên kiểu generic

```rust
struct Diem<T> {
    x: T,
    y: T,
}

impl<T> Diem<T> {
    fn lay_x(&self) -> &T {
        &self.x
    }
}

impl Diem<f64> {
    fn khoang_cach_tu_goc(&self) -> f64 {
        (self.x.powi(2) + self.y.powi(2)).sqrt()
    }
}

fn main() {
    let p = Diem { x: 3.0, y: 4.0 };
    println!("{} {}", p.lay_x(), p.khoang_cach_tu_goc());
}
```

`impl<T> Diem<T>` khai báo method cho **mọi** `T`. `impl Diem<f64>` (không có `<T>` sau `impl`) chỉ
khai báo method cho riêng trường hợp `T` là `f64` — hữu ích khi một phép tính (ở đây là
`khoang_cach_tu_goc`, cần `sqrt`) chỉ có ý nghĩa với một kiểu cụ thể, không tổng quát hoá được cho
mọi `T`.

### Generic không có chi phí lúc chạy

Trình biên dịch **monomorphize** (đơn hình hoá) code generic: với mỗi kiểu cụ thể được dùng thật ở
đâu đó trong code, trình biên dịch tự sinh ra một bản sao code đã thay `T` bằng kiểu cụ thể đó, y
hệt như thể người viết code tự tay viết `so_lon_nhat_i32` và `so_lon_nhat_char` riêng biệt. Nhờ vậy
dùng generic **không hề chậm hơn** viết tay từng phiên bản cụ thể — toàn bộ phần "tổng quát hoá" chỉ
tồn tại lúc biên dịch, biến mất hoàn toàn trong binary cuối cùng.
