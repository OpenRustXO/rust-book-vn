# Tham chiếu và việc mượn

Phần trước kết thúc với một vấn đề: muốn một hàm chỉ _dùng_ một giá trị (ví dụ đọc độ dài một
`String`) mà vẫn dùng tiếp được giá trị đó sau khi hàm chạy xong, sẽ phải move vào rồi trả ngược ra
— rất vướng víu nếu chỉ để đọc. **Tham chiếu** (reference) giải quyết đúng việc đó: cho một hàm truy
cập một giá trị mà không nhận quyền sở hữu nó.

## Mượn bất biến

`&` tạo một tham chiếu — hành động này gọi là **mượn** (borrow): tham chiếu trỏ tới giá trị nhưng
không sở hữu nó, nên khi tham chiếu ra khỏi scope, giá trị gốc không hề bị drop.

```rust
fn main() {
    let s1 = String::from("hello");
    let len = do_dai(&s1);

    println!("Độ dài của '{s1}' là {len}.");
}

fn do_dai(s: &String) -> usize {
    s.len()
}
```

`s1` vẫn dùng được sau lời gọi `do_dai` vì chỉ có một tham chiếu tới nó được truyền vào, không phải
bản thân giá trị. Giống `let`, tham chiếu mặc định cũng bất biến — không thể sửa giá trị thông qua
một tham chiếu thường:

```rust,compile_fail
fn main() {
    let s = String::from("hello");
    them_chu(&s);
}

fn them_chu(s: &String) {
    s.push_str(", world"); // lỗi: `s` là tham chiếu bất biến
}
```

## Mượn khả biến

Thêm `mut` ở cả nơi khai báo lẫn nơi tạo tham chiếu để sửa được giá trị qua tham chiếu:

```rust
fn main() {
    let mut s = String::from("hello");
    them_chu(&mut s);
    println!("{s}");
}

fn them_chu(s: &mut String) {
    s.push_str(", world");
}
```

Mượn khả biến đi kèm một ràng buộc quan trọng: **tại một thời điểm, một giá trị chỉ có thể có một
tham chiếu khả biến đang tồn tại**. Đoạn sau không biên dịch được:

```rust,compile_fail
fn main() {
    let mut s = String::from("hello");

    let r1 = &mut s;
    let r2 = &mut s;

    println!("{r1}, {r2}");
}
```

Ràng buộc này ngăn **data race** ngay tại thời điểm biên dịch thay vì để nó xảy ra lúc chạy rồi mới
phát hiện (hoặc tệ hơn, không phát hiện được và cho ra kết quả sai ngẫu nhiên). Data race xảy ra khi
từ hai trở lên con trỏ cùng truy cập một dữ liệu, ít nhất một trong số đó ghi, và không có cơ chế
nào đồng bộ hoá truy cập — hậu quả kinh điển là dữ liệu đọc được nửa cũ nửa mới, hoặc bị hỏng hoàn
toàn. Rust không đợi việc đó xảy ra rồi debug; nó cấm hẳn khả năng viết ra code như vậy.

Tương tự, không thể vừa có tham chiếu khả biến vừa có tham chiếu bất biến cùng lúc tới cùng một giá
trị — một tham chiếu bất biến giả định giá trị không đổi trong suốt vòng đời của nó, trong khi tham
chiếu khả biến có thể phá vỡ giả định đó bất cứ lúc nào:

```rust,compile_fail
fn main() {
    let mut s = String::from("hello");

    let r1 = &s;
    let r2 = &s;
    let r3 = &mut s;

    println!("{r1}, {r2}, {r3}");
}
```

Ràng buộc này chỉ tính từ nơi một tham chiếu được **tạo ra** tới **lần dùng cuối cùng** của nó,
không phải tới hết scope theo nghĩa dấu ngoặc `{}` — nên đoạn sau vẫn hợp lệ, vì `r1`/`r2` đã dùng
xong (trong lệnh `println!`) trước khi `r3` xuất hiện:

```rust
fn main() {
    let mut s = String::from("hello");

    let r1 = &s;
    let r2 = &s;
    println!("{r1} và {r2}");

    let r3 = &mut s;
    println!("{r3}");
}
```

## Tham chiếu treo

Trình biên dịch Rust đảm bảo không bao giờ tồn tại một **tham chiếu treo** (dangling reference) —
tham chiếu trỏ tới vùng nhớ đã bị giải phóng. Hàm sau cố trả về một tham chiếu tới biến cục bộ của
chính nó:

```rust,compile_fail
fn tham_chieu_treo() -> &String {
    let s = String::from("hello");
    &s
}
```

`s` bị drop ngay khi `tham_chieu_treo` kết thúc, nên tham chiếu trả về sẽ trỏ tới vùng nhớ không còn
gì ở đó — trình biên dịch từ chối biên dịch đoạn này thay vì để nó chạy và đọc phải vùng nhớ đã giải
phóng. Cách sửa đúng là trả về chính giá trị `String`, chuyển quyền sở hữu ra ngoài thay vì chỉ
mượn:

```rust
fn tra_ve_gia_tri() -> String {
    let s = String::from("hello");
    s
}
```

Tóm lại, tại một thời điểm cho một giá trị, Rust chỉ chấp nhận một trong hai điều: hoặc nhiều tham
chiếu bất biến, hoặc đúng một tham chiếu khả biến — không bao giờ cả hai cùng lúc. Toàn bộ việc kiểm
tra này diễn ra lúc biên dịch (gọi là _borrow checker_), không có chi phí gì lúc chạy.
