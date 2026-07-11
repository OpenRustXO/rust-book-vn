# String

`String` thực chất là một wrapper mỏng quanh `Vec<u8>`, kèm đảm bảo nội dung bên trong luôn là UTF-8
hợp lệ — mọi thứ đã học về `Vec` ở phần trước (cấp phát trên heap, độ dài thay đổi được, quy tắc
mượn khi mutate) đều áp dụng cho `String`.

## Tạo và cập nhật

```rust
fn main() {
    let mut s = String::new();

    let du_lieu = "nội dung ban đầu";
    let s2 = du_lieu.to_string();
    let s3 = String::from("nội dung ban đầu");

    s.push_str("xin chào");
    s.push(' ');
    s.push_str("thế giới");
    println!("{s} | {s2} | {s3}");
}
```

`push_str` nhận `&str` (mượn, không lấy quyền sở hữu chuỗi truyền vào), `push` nhận đúng một `char`.

Nối chuỗi bằng `+` gọi thẳng tới method `add(self, s: &str) -> String` — nhận `self` theo giá trị
(không phải tham chiếu), nên **lấy quyền sở hữu** toán hạng bên trái:

```rust
fn main() {
    let s1 = String::from("Xin chào, ");
    let s2 = String::from("thế giới!");
    let s3 = s1 + &s2;
    // s1 đã bị move vào add, không dùng lại được; s2 vẫn dùng được vì chỉ mượn (&s2)

    println!("{s3}");
}
```

Nối nhiều chuỗi bằng `+` liên tiếp nhanh chóng khó đọc; macro `format!` giải quyết gọn hơn và
**không lấy quyền sở hữu** bất kỳ tham số nào (giống `println!` nhưng trả về `String` thay vì in
ra):

```rust
fn main() {
    let s1 = String::from("tic");
    let s2 = String::from("tac");
    let s3 = String::from("toe");

    let s = format!("{s1}-{s2}-{s3}");
    println!("{s} (vẫn dùng được s1: {s1})");
}
```

## Vì sao không đánh chỉ số một String bằng số nguyên

```rust,compile_fail
fn main() {
    let s = String::from("chào");
    let ky_tu_dau = s[0]; // lỗi: String không implement Index<{integer}>
}
```

Đây không phải giới hạn tuỳ tiện. `String` được lưu dưới dạng byte UTF-8 — ký tự ASCII (không dấu)
chiếm đúng 1 byte, nhưng hầu hết ký tự có dấu tiếng Việt chiếm **2 đến 3 byte**. Chữ "chào" gồm bốn
ký tự nhìn thấy nhưng nhiều hơn bốn byte thật sự trong bộ nhớ, vì riêng "à" đã chiếm 2 byte — chỉ số
byte 0 và chỉ số ký tự thứ nhất không còn khớp nhau kể từ ký tự đó trở đi. Nếu `s[i]` được phép và
trả thẳng một byte, kết quả đôi khi đúng (khi `i` rơi vào một ký tự ASCII) và đôi khi vô nghĩa (khi
`i` rơi giữa hai byte của cùng một ký tự có dấu) — đúng sai phụ thuộc vào _nội dung_ chuỗi tại thời
điểm chạy, một loại lỗi cực khó nhận ra qua test. Rust chọn cấm hẳn kiểu đánh chỉ số này thay vì để
nó "thường thì đúng" — buộc lộ vấn đề ngay lúc biên dịch bằng lỗi thiếu trait `Index`, thay vì để nó
âm thầm ẩn nấp tới khi gặp đúng chuỗi có ký tự nhiều byte ở đúng vị trí oái oăm.

Tệ hơn nữa, một chuỗi Unicode có thể nhìn theo ba tầng khác nhau, và cả ba đều là cách hiểu "hợp
lệ":

- **Byte**: đơn vị lưu trữ thật sự, `s.bytes()`.
- **Ký tự** (`char`, Unicode scalar value — xem lại chương Kiểu dữ liệu): `s.chars()`. Một số ký tự
  có dấu biểu diễn được bằng **một** `char` dựng sẵn (dạng dựng sẵn — precomposed, ví dụ `'à'`),
  hoặc bằng **hai** `char` ghép lại — chữ cái gốc cộng dấu kết hợp riêng (combining character, ví dụ
  `'a'` rồi dấu huyền `'\u{0300}'` đứng sau) — cả hai cách đều hiển thị ra "à" giống hệt nhau.
- **Grapheme cluster**: điều người đọc coi là "một chữ" trên màn hình — gần với cách một người Việt
  đếm số chữ trong một từ nhất, nhưng thư viện chuẩn Rust không có sẵn khái niệm này, phải dùng
  crate ngoài (ví dụ `unicode-segmentation`) nếu cần đếm hay cắt chuỗi đúng theo "chữ" thị giác.

Vì "chỉ số 0 của chuỗi" có thể trả lời hoàn toàn khác nhau tuỳ theo hiểu theo tầng nào trong ba tầng
trên, Rust không tự chọn một cách hiểu thay người viết code — buộc phải nói rõ ràng muốn duyệt theo
byte hay theo `char`.

## Slice một String

Vẫn lấy được một phần chuỗi bằng cú pháp slice, nhưng chỉ số là **chỉ số byte**, và chương trình
panic lúc chạy nếu chỉ số đó rơi vào giữa một ký tự nhiều byte:

```rust
fn main() {
    let hello = String::from("hello");
    let s = &hello[0..3]; // "hel" — an toàn vì ASCII, mỗi ký tự đúng 1 byte
    println!("{s}");
}
```

## Duyệt qua từng phần

Chọn `.chars()` hay `.bytes()` tuỳ mục đích — hầu hết logic xử lý văn bản có ý nghĩa (đếm ký tự, in
từng ký tự) nên dùng `.chars()`; xử lý ở mức thấp (mã hoá, ghi ra network) mới cần `.bytes()`:

```rust
fn main() {
    for c in "chào".chars() {
        print!("[{c}]");
    }
    println!();

    for b in "chào".bytes() {
        print!("{b} ");
    }
    println!();
}
```
