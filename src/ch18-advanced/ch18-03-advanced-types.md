# Type nâng cao

## Type alias

`type` tạo một **bí danh** — tên khác cho một kiểu đã có, không phải một kiểu mới:

```rust
type Kilomet = i32;

fn main() {
    let x: i32 = 5;
    let y: Kilomet = 5;
    println!("{}", x + y); // hợp lệ: Kilomet chỉ là i32 với tên khác
}
```

Khác **newtype pattern** (`struct Kilomet(i32)` ở phần trước) — tuple struct tạo một kiểu **thật sự
mới**, không thể cộng thẳng với `i32` nếu thiếu trait phù hợp; `type Kilomet = i32` chỉ là một tên
gọi khác của đúng một kiểu, dùng lẫn lộn được với `i32` ở mọi nơi. Type alias hữu ích để rút gọn một
kiểu dài, lặp lại nhiều lần trong code — ví dụ kiểu closure boxed hay gặp khi lưu callback:

```rust
type ThunkNangTru = Box<dyn Fn() + Send + 'static>;

fn dang_ky(f: ThunkNangTru) {
    f();
}

fn main() {
    dang_ky(Box::new(|| println!("đã chạy")));
}
```

## Never type

`!` là kiểu của một expression **không bao giờ trả về giá trị nào cả** — `panic!`, một `loop` không
`break`, `continue`, hay `std::process::exit` đều có kiểu `!`. Điều đặc biệt: `!` ép kiểu được thành
**bất kỳ** kiểu nào khác, nên một nhánh `match` trả `!` (ví dụ gọi `panic!` hoặc `return` sớm) vẫn
phối hợp được với các nhánh khác trả một kiểu cụ thể, không gây lỗi "các nhánh khác kiểu":

```rust
fn doan(so: i32) -> i32 {
    let mo_ta = match so {
        1 => "một",
        _ => return -1, // nhánh này có kiểu `!`, ép được thành i32 để khớp nhánh kia
    };
    println!("{mo_ta}");
    so
}

fn main() {
    println!("{}", doan(1));
    println!("{}", doan(9));
}
```

## Kiểu có kích thước động (DST) và Sized

`str` (không phải `&str`) và `dyn Trait` (không phải `&dyn Trait`/`Box<dyn Trait>`) là **kiểu có
kích thước không biết trước lúc biên dịch** (dynamically sized type) — không thể có một biến kiểu
`str` trần, luôn phải đứng sau một con trỏ (`&str`, `Box<str>`) mang thêm thông tin về độ dài/vtable
đi kèm. Mọi tham số generic ngầm định mang ràng buộc `T: Sized` (kích thước cố định biết trước) trừ
khi nới lỏng tường minh bằng `?Sized`:

```rust
fn in_ra<T: ?Sized + std::fmt::Debug>(gia_tri: &T) {
    println!("{gia_tri:?}");
}

fn main() {
    in_ra("một chuỗi str động");
    in_ra(&42);
}
```

`?Sized` cho phép `T` là cả kiểu `Sized` bình thường lẫn kiểu không xác định kích thước như `str` —
đổi lại, tham số phải luôn nhận qua một dạng con trỏ (`&T`), không bao giờ nhận `T` theo giá trị,
đúng như bản chất DST không thể tồn tại trực tiếp trên stack như một giá trị trần.
