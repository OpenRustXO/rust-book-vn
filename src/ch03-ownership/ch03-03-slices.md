# Slice

Xét bài toán: viết một hàm tìm từ đầu tiên trong một chuỗi, trả về vị trí kết thúc của từ đó. Cách
trực tiếp nhất là trả về một `usize` — chỉ số byte nơi từ đầu tiên kết thúc:

```rust
fn tu_dau_tien(s: &String) -> usize {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return i;
        }
    }

    s.len()
}
```

Chỉ số trả về này chỉ có ý nghĩa khi đi kèm với `String` gốc — nhưng bản thân kiểu `usize` không hề
giữ mối liên kết đó. Nếu chuỗi gốc bị xoá hoặc thay nội dung sau khi có chỉ số, chỉ số cũ vẫn là một
con số hợp lệ nhưng không còn khớp với nội dung thật:

```rust
fn tu_dau_tien(s: &String) -> usize {
    let bytes = s.as_bytes();
    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return i;
        }
    }
    s.len()
}

fn main() {
    let mut s = String::from("hello world");
    let word = tu_dau_tien(&s);

    s.clear();
    // word vẫn là 5 ở đây, nhưng s giờ là chuỗi rỗng — 5 không còn ý nghĩa gì
    println!("{word}");
}
```

Chương trình trên vẫn biên dịch và chạy — không có gì báo cho biết `word` đã trở nên vô nghĩa. Đây
đúng là dạng bug mà cơ chế mượn của Rust có thể bắt được, nếu hàm trả về đúng một thứ giữ liên kết
với dữ liệu gốc thay vì một con số rời rạc. Đó là việc **slice** làm được.

## String slice

`&str` (string slice) là một tham chiếu tới một dải byte liên tiếp bên trong một `String` — không sở
hữu dữ liệu, chỉ tham chiếu tới nó, giống các tham chiếu đã gặp ở phần trước:

```rust
fn main() {
    let s = String::from("hello world");

    let hello = &s[0..5];
    let world = &s[6..11];
    println!("{hello} {world}");
}
```

Viết lại hàm tìm từ đầu tiên để trả về một slice thay vì một chỉ số rời:

```rust
fn tu_dau_tien(s: &String) -> &str {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }

    &s[..]
}
```

Vì `&str` trả về là một tham chiếu tới `s`, borrow checker áp dụng đúng quy tắc đã học ở phần trước:
không thể có tham chiếu khả biến tới `s` (ví dụ gọi `s.clear()`, nhận `&mut self`) trong lúc tham
chiếu bất biến này còn sống. Thử biên dịch phiên bản dùng slice với cùng đoạn code gây lỗi ở trên sẽ
bắt được bug ngay tại thời điểm biên dịch:

```rust,compile_fail
fn main() {
    let mut s = String::from("hello world");
    let word = tu_dau_tien(&s);

    s.clear(); // lỗi: không thể mượn khả biến `s` vì `word` đang mượn bất biến nó
    println!("{word}");
}

fn tu_dau_tien(s: &String) -> &str {
    let bytes = s.as_bytes();
    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }
    &s[..]
}
```

Đây chính là giá trị cốt lõi của slice: nó không chỉ là một cách biểu diễn dữ liệu tiện lợi, mà còn
kéo theo toàn bộ đảm bảo của hệ thống mượn — chỉ số cũ (`usize`) không thể trở nên "treo" theo nghĩa
logic vì bản thân nó không phải tham chiếu, nhưng slice thì có, và nhờ vậy trình biên dịch phát hiện
được chính xác loại lỗi vừa mô tả.

String literal (`let s = "hello";`) vốn dĩ đã có kiểu `&str` — một slice trỏ thẳng vào dữ liệu nằm
sẵn trong binary đã biên dịch, bất biến theo đúng bản chất của binary. Đây cũng là lý do các hàm nên
nhận tham số kiểu `&str` thay vì `&String` khi có thể: một `&String` tự động ép kiểu được về `&str`,
nên nhận `&str` cho phép hàm nhận cả hai, tổng quát hơn mà không mất gì.

## Slice trên các collection khác

Khái niệm slice không riêng gì `String` — áp dụng cho mọi dãy dữ liệu liên tiếp, ví dụ mảng:

```rust
fn main() {
    let a = [1, 2, 3, 4, 5];
    let slice = &a[1..3];

    println!("{:?}", slice);
    assert_eq!(slice, &[2, 3]);
}
```

`&[i32]` mang theo cả con trỏ đầu dải lẫn độ dài, giống hệt cách `&str` hoạt động với `String` —
cùng một cơ chế, áp dụng cho mọi kiểu dữ liệu là một dãy liên tiếp trong bộ nhớ.
