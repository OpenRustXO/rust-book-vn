# Collection phổ biến

Khác mảng và tuple (kích thước cố định, biết trước lúc biên dịch), các collection trong chương này
lưu dữ liệu trên heap — số lượng phần tử tăng giảm được lúc chạy. Ba collection dùng nhiều nhất:
`Vec<T>` (danh sách), `String` (chuỗi UTF-8), và `HashMap<K, V>` (bảng băm khoá-giá trị).

## Vector

`Vec<T>` lưu nhiều giá trị **cùng kiểu** `T`, liền nhau trong bộ nhớ, độ dài thay đổi được:

```rust
fn main() {
    let mut v: Vec<i32> = Vec::new();
    v.push(5);
    v.push(6);
    v.push(7);

    println!("{v:?}");
}
```

Chú thích kiểu `Vec<i32>` cần thiết ở `Vec::new()` vì chưa có phần tử nào để trình biên dịch suy ra
`T`. Có sẵn giá trị ngay từ đầu, dùng macro `vec!` — trình biên dịch suy luận được `T` từ chính giá
trị truyền vào:

```rust
fn main() {
    let v = vec![1, 2, 3];
    println!("{v:?}");
}
```

### Đọc phần tử

Hai cách đọc một phần tử theo chỉ số, khác nhau ở cách xử lý chỉ số vượt quá độ dài:

```rust
fn main() {
    let v = vec![1, 2, 3, 4, 5];

    let thu_ba: &i32 = &v[2];
    println!("Phần tử thứ ba: {thu_ba}");

    let thu_ba: Option<&i32> = v.get(2);
    match thu_ba {
        Some(gia_tri) => println!("Phần tử thứ ba: {gia_tri}"),
        None => println!("Không có phần tử thứ ba"),
    }
}
```

`v[2]` **panic** ngay nếu chỉ số vượt quá độ dài — hợp lý khi truy cập ngoài phạm vi là một lỗi
logic nghiêm trọng, chương trình nên dừng lại thay vì chạy tiếp với dữ liệu sai. `v.get(2)` trả về
`Option<&i32>` — `None` khi vượt quá độ dài thay vì panic, hợp lý khi việc "không có phần tử ở vị
trí đó" là một khả năng bình thường cần xử lý, ví dụ chỉ số tới từ input người dùng.

### Vòng đời tham chiếu và việc mutate

Borrow checker áp dụng đúng các quy tắc đã học ở chương Quyền sở hữu cho phần tử bên trong `Vec`:

```rust,compile_fail
fn main() {
    let mut v = vec![1, 2, 3, 4, 5];

    let phan_tu_dau = &v[0];

    v.push(6);

    println!("Phần tử đầu: {phan_tu_dau}");
}
```

Lỗi này thoạt nhìn khó hiểu — thêm một phần tử ở cuối sao lại ảnh hưởng tới tham chiếu ở đầu? Bản
chất: `Vec` lưu phần tử liền nhau trong một vùng cấp phát; nếu vùng đó hết chỗ, `push` phải cấp phát
một vùng mới lớn hơn rồi copy toàn bộ phần tử cũ sang, giải phóng vùng cũ. Nếu điều đó xảy ra,
`phan_tu_dau` sẽ trỏ vào vùng nhớ đã giải phóng — một tham chiếu treo. Borrow checker ngăn khả năng
đó bằng đúng quy tắc quen thuộc: không thể có tham chiếu bất biến (`phan_tu_dau`) và tham chiếu khả
biến (`v.push` cần mượn khả biến `v`) cùng lúc.

### Lưu nhiều kiểu khác nhau bằng enum

Một `Vec` chỉ chứa được đúng một kiểu `T` — nhưng `T` có thể là một enum, và enum thì gom được nhiều
"loại" dữ liệu khác nhau vào chung một kiểu (đã học ở chương Enum):

```rust
enum OGiaTri {
    So(i32),
    Chu(String),
}

fn main() {
    let hang: Vec<OGiaTri> = vec![
        OGiaTri::So(3),
        OGiaTri::Chu(String::from("xin chào")),
        OGiaTri::So(10),
    ];

    for o in &hang {
        match o {
            OGiaTri::So(n) => println!("số: {n}"),
            OGiaTri::Chu(s) => println!("chữ: {s}"),
        }
    }
}
```

Cách này đòi hỏi biết trước toàn bộ tập kiểu có thể xuất hiện tại thời điểm biên dịch. Khi tập kiểu
không biết trước (ví dụ cho phép plugin bên ngoài định nghĩa thêm), cần tới trait object — nói ở
[chương Trait object và hướng đối tượng](../ch16-oop/ch16-01-trait-objects.md).
