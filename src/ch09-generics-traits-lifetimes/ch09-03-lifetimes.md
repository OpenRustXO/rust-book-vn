# Lifetime

Mọi tham chiếu trong Rust có một **lifetime** — thời hạn sống, tức phạm vi code mà tham chiếu đó còn
hợp lệ. Phần lớn thời gian trình biên dịch tự suy luận được (elision, nói ở phần sau), giống suy
luận kiểu — nhưng đôi khi một chữ ký hàm có nhiều cách gán lifetime hợp lệ, và trình biên dịch cần
được nói rõ nên chọn cách nào.

## Vấn đề lifetime giải quyết

```rust,compile_fail
fn dai_nhat(x: &str, y: &str) -> &str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}

fn main() {}
```

Lỗi biên dịch: "missing lifetime specifier". Hàm trả về **một trong hai** tham chiếu tham số, tuỳ
điều kiện `if` — nhưng chữ ký hàm không nói rõ tham chiếu trả về gắn với `x` hay `y`. Borrow checker
cần biết chính xác điều đó để xác nhận: tham chiếu trả về có chắc chắn còn hợp lệ tại nơi gọi hàm
hay không, tuỳ vào `x`, `y` sống được bao lâu ở chỗ gọi. Không tự suy ra được từ code, nên trình
biên dịch từ chối đoán.

Sửa bằng cách gắn cùng một tham số lifetime `'a` cho cả hai tham số và giá trị trả về:

```rust
fn dai_nhat<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}

fn main() {
    let s1 = String::from("chuỗi dài hơn");
    let s2 = String::from("ngắn");
    let ket_qua = dai_nhat(s1.as_str(), s2.as_str());
    println!("Chuỗi dài nhất là {ket_qua}");
}
```

`'a` không **tạo ra** hay **kéo dài** lifetime nào — nó chỉ mô tả một ràng buộc cho borrow checker:
"tham chiếu trả về hợp lệ trong đúng khoảng thời gian mà **cả** `x` lẫn `y` cùng hợp lệ" (giao của
hai lifetime, chọn cái ngắn hơn). Nhờ ràng buộc đó, borrow checker bắt được đúng lúc dùng sai:

```rust,compile_fail
fn dai_nhat<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}

fn main() {
    let s1 = String::from("chuỗi dài hơn");
    let ket_qua;
    {
        let s2 = String::from("ngắn");
        ket_qua = dai_nhat(s1.as_str(), s2.as_str());
    }
    println!("Chuỗi dài nhất là {ket_qua}"); // lỗi: s2 không còn sống tới đây
}
```

`ket_qua` mang lifetime bị ràng buộc bởi cả `s1` và `s2` — dù giá trị _thực tế_ trả về ở lần chạy
này tình cờ luôn là `s1` (chuỗi dài hơn), trình biên dịch không chạy thử để biết điều đó, nó chỉ
nhìn chữ ký hàm và phải chấp nhận khả năng xấu nhất: `ket_qua` có thể là `s2`, nên không cho
`ket_qua` sống lâu hơn `s2`.

## Lifetime trên struct

Một struct giữ tham chiếu (thay vì sở hữu dữ liệu) phải khai báo lifetime cho field đó — struct
không thể sống lâu hơn dữ liệu nó tham chiếu tới:

```rust
struct TrichDan<'a> {
    phan: &'a str,
}

fn main() {
    let tieu_thuyet = String::from("Gọi tôi là Ishmael. Một vài năm trước.");
    let cau_dau = tieu_thuyet.split('.').next().unwrap();
    let td = TrichDan { phan: cau_dau };

    println!("{}", td.phan);
}
```

## Lifetime elision: vì sao không phải lúc nào cũng thấy `'a`

Rất nhiều hàm nhận và trả về tham chiếu mà không cần viết lifetime tường minh — nhờ vài quy tắc suy
luận cố định (elision rules) áp dụng tự động khi chữ ký hàm khớp một trong các khuôn mẫu phổ biến:
đặc biệt, **nếu chỉ có đúng một tham số kiểu tham chiếu**, lifetime của nó tự động gán cho mọi tham
chiếu trong giá trị trả về:

```rust
fn tu_dau_tien(s: &str) -> &str {
    s.split_whitespace().next().unwrap_or("")
}

fn main() {
    let s = String::from("hello world");
    println!("{}", tu_dau_tien(&s));
}
```

Chỉ khi có từ hai tham chiếu đầu vào trở lên và các quy tắc suy luận không xác định được duy nhất
một cách gán hợp lý (như hàm `dai_nhat` ở trên) mới cần viết `'a` tường minh.

## Lifetime 'static

`'static` là lifetime đặc biệt: tham chiếu sống được **suốt vòng đời chương trình**. Mọi string
literal đều có kiểu `&'static str`, vì nội dung của nó nằm sẵn trong binary đã biên dịch, tồn tại
liên tục từ lúc chương trình chạy tới khi kết thúc:

```rust
fn main() {
    let s: &'static str = "Tôi sống suốt chương trình.";
    println!("{s}");
}
```

Thông báo lỗi gợi ý thêm `'static` để "sửa nhanh" một lỗi lifetime nên được cân nhắc kỹ chứ không áp
dụng máy móc — ép một tham chiếu thường thành `'static` thường là dấu hiệu đang cố giữ một tham
chiếu sống lâu hơn dữ liệu gốc của nó thật sự cho phép, chứ không phải cách sửa đúng gốc rễ vấn đề.
