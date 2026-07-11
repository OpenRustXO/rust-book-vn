# Quyền sở hữu

Đây là tính năng đặc trưng nhất của Rust — cơ chế cho phép quản lý bộ nhớ an toàn mà không cần một
garbage collector chạy nền, và không bắt người viết code tự gọi free thủ công như C. Toàn bộ việc
kiểm tra diễn ra tại thời điểm biên dịch, không có chi phí lúc chạy.

## Vì sao cần quyền sở hữu

Một chương trình phải quản lý bộ nhớ nó dùng lúc chạy, theo hai vùng khác nhau:

- **Stack**: lưu theo thứ tự LIFO (vào sau ra trước), chỉ chứa được dữ liệu có kích thước biết trước
  và cố định tại thời điểm biên dịch. Cấp phát/giải phóng trên stack chỉ là dịch chuyển một con trỏ,
  cực nhanh.
- **Heap**: chứa được dữ liệu có kích thước không biết trước hoặc có thể thay đổi lúc chạy (ví dụ
  một chuỗi người dùng nhập vào). Cấp phát trên heap phải nhờ bộ cấp phát (allocator) tìm một vùng
  trống đủ lớn, trả về con trỏ tới đó — chậm hơn stack, và vùng nhớ này phải được giải phóng lại
  đúng một lần khi không còn dùng nữa.

C giao toàn bộ việc giải phóng heap cho người viết code tự gọi `free`, dễ dẫn tới hai lỗi kinh điển:
quên gọi (rò rỉ bộ nhớ) hoặc gọi hai lần / dùng lại sau khi đã giải phóng (undefined behavior).
Nhiều ngôn ngữ khác né hai lỗi đó bằng garbage collector quét bộ nhớ lúc chạy để tự dọn — an toàn
nhưng có chi phí runtime. Rust chọn cách thứ ba: một tập quy tắc để trình biên dịch **tự suy ra
chính xác thời điểm giải phóng**, kiểm tra toàn bộ lúc biên dịch, không cần theo dõi gì lúc chạy.

## Ba quy tắc

1. Mỗi giá trị trong Rust có đúng một biến gọi là **owner** (chủ sở hữu) của nó.
2. Tại một thời điểm, chỉ có một owner.
3. Khi owner ra khỏi scope, giá trị bị **drop** — bộ nhớ được giải phóng ngay lập tức, không cần
   chờ.

Xét kiểu `String` — khác với string literal (`&str`, có độ dài cố định biết trước, nằm sẵn trong
binary), `String` cấp phát trên heap để chứa nội dung có thể thay đổi và tăng độ dài lúc chạy:

```rust
fn main() {
    let mut s = String::from("hello");
    s.push_str(", world!");
    println!("{s}");
}
```

Khi `s` ra khỏi scope (kết thúc `main`), Rust tự động gọi một hàm dọn dẹp đặc biệt tên `drop` để trả
lại vùng heap đó — không cần người viết code nhớ gọi gì cả, và việc này xảy ra ở đúng một chỗ xác
định trước (cuối scope), không phải đoán khi nào garbage collector rảnh để quét.

## Move: gán giá trị không phải lúc nào cũng là copy

```rust,compile_fail
fn main() {
    let s1 = String::from("hello");
    let s2 = s1;

    println!("{s1}, world!"); // lỗi: value borrowed here after move
}
```

Đoạn trên **không biên dịch được**. `String` gồm một con trỏ tới dữ liệu trên heap, độ dài, và dung
lượng đã cấp phát — ba giá trị này nằm trên stack. Khi gán `s1` cho `s2`, Rust copy ba giá trị đó
(chứ không copy toàn bộ nội dung trên heap), nghĩa là sau dòng đó cả `s1` và `s2` đều có con trỏ trỏ
tới cùng một vùng heap.

Nếu để cả hai biến cùng hợp lệ, khi cả hai ra khỏi scope, Rust sẽ gọi `drop` hai lần trên cùng một
vùng heap — giải phóng hai lần (double free), một trong những lỗi bộ nhớ nghiêm trọng nhất. Để tránh
điều đó, Rust coi `s1` **không còn hợp lệ** ngay sau khi gán cho `s2` — đây gọi là một **move**:
quyền sở hữu chuyển từ `s1` sang `s2`, không phải nhân đôi dữ liệu. Dùng `s1` sau khi đã move là lỗi
biên dịch, bắt được ngay chứ không phải một bug ẩn chờ tới lúc chạy mới lộ ra.

Nếu thật sự cần cả hai biến cùng sở hữu hai bản dữ liệu độc lập trên heap, gọi `.clone()` tường minh
— chi phí sao chép sâu (deep copy) khi đó là chủ đích, nhìn vào code là thấy ngay, không bị giấu đi
trong một phép gán trông vô hại:

```rust
fn main() {
    let s1 = String::from("hello");
    let s2 = s1.clone();

    println!("s1 = {s1}, s2 = {s2}");
}
```

## Kiểu Copy: khi gán chỉ đơn thuần là copy

Các kiểu scalar đã gặp ở chương trước (số nguyên, số thực, `bool`, `char`) và tuple chỉ chứa toàn
kiểu Copy có kích thước cố định biết trước, nằm hoàn toàn trên stack — sao chép chúng chỉ là copy
vài byte, rẻ tới mức không cần phân biệt "move" hay "copy". Những kiểu này implement trait `Copy`:
gán chúng cho biến khác **không** làm biến cũ mất hiệu lực:

```rust
fn main() {
    let x = 5;
    let y = x;

    println!("x = {x}, y = {y}");
}
```

Một kiểu chỉ có thể implement `Copy` nếu bản thân nó và mọi thành phần bên trong đều không cần chạy
logic dọn dẹp gì đặc biệt lúc bị drop — `String` quản lý một vùng heap nên không thể là `Copy`.

## Ownership và hàm

Truyền một giá trị vào hàm tuân theo đúng quy tắc move/copy như khi gán cho biến; trả một giá trị ra
khỏi hàm cũng chuyển quyền sở hữu ra ngoài:

```rust
fn main() {
    let s = String::from("hello");
    lay_quyen_so_huu(s);
    // s không còn hợp lệ ở đây — quyền sở hữu đã chuyển vào hàm

    let x = 5;
    khong_lay_quyen_so_huu(x);
    println!("x vẫn dùng được vì i32 là Copy: {x}");
}

fn lay_quyen_so_huu(chuoi: String) {
    println!("{chuoi}");
} // chuoi ra khỏi scope, drop vùng heap nó sở hữu

fn khong_lay_quyen_so_huu(so: i32) {
    println!("{so}");
}
```

Luôn phải move quyền sở hữu vào rồi trả ngược ra chỉ để dùng tiếp một giá trị sẽ rất bất tiện — đó
là lý do Rust có tham chiếu, nói ở phần tiếp theo, cho phép một hàm _dùng_ một giá trị mà không cần
nhận quyền sở hữu nó.
