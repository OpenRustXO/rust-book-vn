# Iterator

Iterator xử lý một chuỗi giá trị lần lượt từng phần tử, theo trait `Iterator` trong thư viện chuẩn —
chỉ cần đúng một method bắt buộc:

```rust,ignore
pub trait Iterator {
    type Item;
    fn next(&mut self) -> Option<Self::Item>;
}
```

`next()` trả `Some(gia_tri)` cho từng phần tử, `None` khi hết. `Self::Item` là kiểu phần tử — mỗi
kiểu implement `Iterator` tự quyết định `Item` cụ thể của mình.

## Tính lười

```rust
fn main() {
    let v = vec![1, 2, 3];
    let mut iter = v.iter();

    println!("{:?}", iter.next());
    println!("{:?}", iter.next());
    println!("{:?}", iter.next());
    println!("{:?}", iter.next());
}
```

Tạo một iterator (`v.iter()`) **không làm gì cả** — không có phần tử nào được xử lý cho tới khi
`next()` (trực tiếp hoặc gián tiếp qua `for`, hay các adaptor dưới đây) thật sự được gọi. Đây gọi là
tính **lười** (lazy): định nghĩa một chuỗi phép biến đổi trên iterator không tốn chi phí gì cho tới
khi có thứ gì đó thật sự "tiêu thụ" (consume) nó.

`v.iter()` cho iterator các tham chiếu bất biến (`&i32`, không lấy quyền sở hữu `v`);
`v.into_iter()` lấy quyền sở hữu `v`, cho iterator các giá trị sở hữu hẳn (`i32`); `v.iter_mut()`
cho tham chiếu khả biến, sửa được từng phần tử ngay trong lúc duyệt.

## Adaptor tiêu thụ

Một số method gọi `next()` lặp lại cho tới khi hết, "tiêu thụ" hẳn iterator để tính ra một giá trị
duy nhất:

```rust
fn main() {
    let v = vec![1, 2, 3];
    let tong: i32 = v.iter().sum();
    println!("{tong}");
}
```

`for` cũng tiêu thụ iterator theo cách tương tự, gọi `next()` ở mỗi vòng lặp — cách viết
`for x in v.iter()` chẳng qua là đường cú pháp gọn cho đúng cơ chế `next()` này.

## Adaptor biến đổi

Một số method khác **không** tiêu thụ ngay mà trả về một iterator mới, đã áp thêm một phép biến đổi
— vẫn lười, chưa chạy gì cho tới khi có một adaptor tiêu thụ (hoặc `for`) phía sau:

```rust
fn main() {
    let v = vec![1, 2, 3];
    let gap_doi: Vec<i32> = v.iter().map(|x| x * 2).collect();
    println!("{gap_doi:?}");
}
```

`map` không tự chạy — nó trả về một iterator mới, mô tả "sẽ nhân đôi từng phần tử khi được hỏi tới".
`collect()` mới là bước thật sự tiêu thụ, gom kết quả vào một collection cụ thể (ở đây là
`Vec<i32>`, suy luận từ chú thích kiểu — `collect` generic trên kiểu đích nên gần như luôn cần một
gợi ý kiểu ở đâu đó, qua `let` hoặc cú pháp turbofish `::<Vec<i32>>()`).

`filter` giữ lại phần tử thoả một điều kiện, thường kết hợp với closure capture biến từ scope ngoài:

```rust
#[derive(Debug)]
struct SanPham {
    ten: String,
    gia: u32,
}

fn loc_theo_ngan_sach(danh_sach: Vec<SanPham>, ngan_sach: u32) -> Vec<SanPham> {
    danh_sach
        .into_iter()
        .filter(|sp| sp.gia <= ngan_sach)
        .collect()
}

fn main() {
    let san_pham = vec![
        SanPham {
            ten: String::from("Bàn phím"),
            gia: 500,
        },
        SanPham {
            ten: String::from("Chuột"),
            gia: 150,
        },
    ];

    let phu_hop = loc_theo_ngan_sach(san_pham, 200);
    println!("{phu_hop:?}");
}
```

`filter` capture `ngan_sach` từ scope ngoài closure — kết hợp iterator với closure là cách rất phổ
biến để viết logic xử lý dữ liệu ngắn gọn mà không cần vòng lặp `for` tường minh.

## Hiệu năng: trừu tượng hoá chi phí bằng không

Chuỗi `map`/`filter`/`collect` trông như tạo ra nhiều bước trung gian, nhưng trình biên dịch tối ưu
hoá toàn bộ chuỗi adaptor xuống thành mã máy **tương đương hệt** một vòng lặp `for` viết tay, không
có overhead gọi hàm hay cấp phát trung gian thừa. Đây là ví dụ điển hình cho triết lý "trừu tượng
hoá chi phí bằng không" (zero-cost abstraction) của Rust — dùng iterator không đánh đổi hiệu năng để
lấy sự ngắn gọn, khác với cảm giác trực giác rằng code "trừu tượng hơn" ắt phải "chậm hơn".
