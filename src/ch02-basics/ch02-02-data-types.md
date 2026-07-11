# Kiểu dữ liệu

Rust là ngôn ngữ định kiểu tĩnh: kiểu của mọi giá trị phải xác định được tại thời điểm biên dịch.
Phần lớn thời gian trình biên dịch tự suy luận ra kiểu từ giá trị và cách dùng, không cần viết tường
minh. Suy luận chỉ bó tay khi một giá trị có thể ứng với nhiều kiểu đích hợp lệ — ví dụ parse một
chuỗi thành số:

```rust,compile_fail
fn main() {
    let guess = "42".parse().expect("Không phải số");
}
```

`parse` có thể trả về bất kỳ kiểu số nào implement trait tương ứng, nên trình biên dịch không biết
chọn kiểu nào. Phải chú thích tường minh:

```rust
fn main() {
    let guess: u32 = "42".parse().expect("Không phải số");
    println!("{guess}");
}
```

## Kiểu vô hướng (scalar)

Một giá trị scalar biểu diễn đúng một giá trị đơn lẻ. Rust có bốn kiểu scalar chính: số nguyên, số
thực dấu phẩy động, boolean, và ký tự.

**Số nguyên** có dấu (`i8`, `i16`, `i32`, `i64`, `i128`, `isize`) và không dấu (`u8`...`u128`,
`usize`), số trong tên là số bit chiếm dụng — quyết định khoảng giá trị biểu diễn được. `isize`/
`usize` có kích thước bằng kiến trúc máy (64 bit trên macOS hiện đại), dùng chủ yếu để đánh chỉ mục
(index) vào collection. Không chú thích gì, mặc định là `i32` — thường là lựa chọn nhanh nhất trên
đa số kiến trúc.

Một chi tiết dễ bị bỏ qua nhưng quan trọng: khi phép tính vượt quá khoảng giá trị của kiểu (integer
overflow), hành vi khác nhau giữa hai profile biên dịch. Build debug (mặc định của `cargo build`
hoặc `cargo run`) sẽ panic ngay khi overflow xảy ra. Build release (cờ `--release`) **không** panic
— giá trị "cuộn vòng" (wrap) theo số học modulo, ví dụ một `u8` cộng vượt 255 sẽ quay về 0. Sự khác
biệt này tồn tại vì kiểm tra overflow có chi phí runtime; Rust chấp nhận bắt lỗi sớm khi đang phát
triển (debug) nhưng ưu tiên tốc độ khi đã release. Muốn xử lý overflow một cách tường minh, chủ động
thay vì phó mặc cho hành vi mặc định của profile, dùng các method như `wrapping_add`, `checked_add`,
`saturating_add` thay vì toán tử `+` trần.

**Số thực dấu phẩy động** có `f32` và `f64` (theo chuẩn IEEE 754), mặc định là `f64` — trên các CPU
hiện đại, `f64` và `f32` có tốc độ tính toán tương đương nhau nhưng `f64` cho độ chính xác cao hơn.

**Boolean** (`bool`) chỉ có hai giá trị `true`/`false`, chiếm 1 byte.

**Ký tự** (`char`) biểu diễn một Unicode scalar value, chiếm 4 byte — rộng hơn nhiều so với 1 byte
ASCII. Một `char` có thể biểu diễn chữ cái có dấu, chữ Hán, emoji, hay bất kỳ ký tự nào trong chuẩn
Unicode, không riêng gì bảng chữ cái Latin:

```rust
fn main() {
    let c = 'z';
    let chu_a: char = 'A';
    let icon = '😻';
    println!("{c} {chu_a} {icon}");
}
```

## Kiểu phức hợp (compound)

Kiểu phức hợp gộp nhiều giá trị vào một kiểu duy nhất. Rust có hai kiểu phức hợp nguyên thuỷ: tuple
và mảng (array).

**Tuple** gom một số lượng giá trị cố định, có thể khác kiểu nhau, thành một giá trị:

```rust
fn main() {
    let tup: (i32, f64, u8) = (500, 6.4, 1);

    let (x, y, z) = tup;
    println!("Giá trị của y là: {y}");

    println!("Phần tử đầu: {}", tup.0);
}
```

Có thể lấy từng phần tử ra bằng destructuring (`let (x, y, z) = tup`) hoặc truy cập trực tiếp bằng
chỉ số sau dấu chấm (`tup.0`). Một tuple không chứa gì cả — `()` — gọi là _unit_; đây cũng là kiểu
trả về mặc định của một biểu thức không trả về giá trị có ý nghĩa nào.

**Mảng** gom nhiều giá trị **cùng kiểu**, độ dài cố định ngay tại thời điểm biên dịch — khác với
kiểu `Vec` sẽ gặp ở [chương Collection](../ch07-collections/ch07-01-vectors.md), có thể tăng giảm
kích thước lúc chạy. Vì độ dài cố định, một mảng được cấp phát trên stack thay vì heap (khái niệm
stack/heap sẽ rõ hơn ở chương Quyền sở hữu) — phù hợp khi biết chắc số lượng phần tử không đổi, ví
dụ tên các tháng trong năm:

```rust
fn main() {
    let thang: [&str; 3] = ["Một", "Hai", "Ba"];
    let so_khoi_tao = [3; 5]; // tương đương [3, 3, 3, 3, 3]

    println!("Tháng đầu tiên: {}", thang[0]);
    println!("{:?}", so_khoi_tao);
}
```

Truy cập một chỉ số vượt quá độ dài mảng sẽ panic ngay lúc chạy (runtime) thay vì âm thầm đọc vùng
nhớ không thuộc về mảng — Rust luôn kiểm tra chỉ số hợp lệ trước khi truy cập, kể cả khi việc đó có
thêm chi phí kiểm tra ở mỗi lần truy cập.
