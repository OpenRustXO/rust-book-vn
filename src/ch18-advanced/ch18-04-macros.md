# Macro

Hàm nhận **giá trị** lúc chạy, trả về **giá trị**. Macro nhận **code** (dưới dạng token) lúc biên
dịch, sinh ra **code** khác thay thế vào chỗ gọi nó — khác biệt này cho macro làm được những việc
hàm không làm được: nhận số lượng tham số khác nhau tuỳ lần gọi (`println!` nhận bao nhiêu tham số
cũng được, một hàm thường thì không), hay tự động sinh cả một implementation trait cho một kiểu
(`#[derive(Debug)]` — bản thân cũng là một loại macro).

## Macro khai báo (macro_rules!)

So khớp cấu trúc cú pháp của code truyền vào với một pattern, rồi thay bằng code mẫu tương ứng —
tương tự `match` nhưng so khớp trên **code**, không phải giá trị lúc chạy. Viết một phiên bản đơn
giản hoá của `vec!`:

```rust
macro_rules! vec_cua_toi {
    ( $( $x:expr ),* ) => {
        {
            let mut v = Vec::new();
            $(
                v.push($x);
            )*
            v
        }
    };
}

fn main() {
    let v: Vec<i32> = vec_cua_toi![1, 2, 3];
    println!("{v:?}");
}
```

`$x:expr` khớp một expression bất kỳ, đặt tên `$x`. `$(...),*` nghĩa là "lặp lại phần này, các lượt
cách nhau dấu phẩy, không giới hạn số lần" — khớp được `vec_cua_toi![1, 2, 3]` lẫn `vec_cua_toi![]`
(không phần tử) hay `vec_cua_toi![1]` (một phần tử), sinh ra đúng số lệnh `v.push` tương ứng số
expression đã truyền vào.

## Macro thủ tục (procedural macro)

Mạnh hơn `macro_rules!` nhưng phức tạp hơn nhiều: nhận trực tiếp luồng token (`TokenStream`) làm đầu
vào, chạy code Rust bất kỳ để xử lý (thường dùng crate `syn` để phân tích cú pháp thành cây cú pháp,
`quote` để sinh code trở lại), trả về một `TokenStream` khác thay thế vào chỗ gọi. Luôn định nghĩa
trong một crate riêng, khai báo `proc-macro = true` — không viết chung được với code thường trong
cùng crate, nên nằm ngoài phạm vi một ví dụ ngắn có thể chạy thử trực tiếp ở đây. Ba dạng:

- **Custom derive**: `#[derive(TenTrait)]` — tự sinh implementation của một trait, dựa trên cấu trúc
  field của struct/enum đang derive (giống hệt cách `#[derive(Debug)]` tự sinh code in ra, chỉ khác
  đây là do chính người viết crate định nghĩa cho trait của riêng mình).
- **Attribute-like**: `#[ten_attribute]` — attribute tuỳ chỉnh gắn lên bất kỳ item nào, không nhất
  thiết gắn với `derive`.
- **Function-like**: `ten_macro!(...)` — trông giống `macro_rules!` khi gọi, nhưng cú pháp bên trong
  dấu ngoặc không cần là Rust hợp lệ, macro thủ tục tự định nghĩa cách phân tích cú pháp lấy vào.

Cả ba đều là công cụ để **tự động sinh code lặp đi lặp lại** dựa trên cấu trúc code có sẵn — thứ mà
nếu viết tay sẽ phải chép lại gần như y hệt cho mỗi kiểu dữ liệu mới, đánh đổi lấy sự phức tạp lúc
biên dịch (cần một crate `proc-macro` riêng, công cụ phân tích cú pháp) để tránh sự lặp lại đó lúc
viết code.
