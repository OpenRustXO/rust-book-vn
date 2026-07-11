# Rc và RefCell

## Rc\<T\>: nhiều chủ sở hữu

Quy tắc quyền sở hữu từ đầu sách tới giờ luôn giả định **một** chủ sở hữu. Một số cấu trúc dữ liệu
cần nhiều phần code cùng hợp lệ sở hữu chung một giá trị — ví dụ nhiều node trong một cấu trúc dạng
đồ thị cùng tham chiếu tới một node dùng chung. `Rc<T>` (Reference Counted) cho phép điều đó bằng
cách đếm số chủ sở hữu hiện tại, chỉ drop dữ liệu thật khi đếm về 0:

```rust
use std::rc::Rc;

fn main() {
    let a = Rc::new(String::from("dữ liệu dùng chung"));
    println!("Đếm sau khi tạo a = {}", Rc::strong_count(&a));

    let b = Rc::clone(&a);
    println!("Đếm sau khi tạo b = {}", Rc::strong_count(&a));

    {
        let c = Rc::clone(&a);
        println!("Đếm sau khi tạo c = {}", Rc::strong_count(&a));
    }

    println!("Đếm sau khi c ra khỏi scope = {}", Rc::strong_count(&a));
}
```

`Rc::clone(&a)` chỉ tăng bộ đếm tham chiếu — không copy dữ liệu bên trong. Quy ước dùng
`Rc::clone(&a)` thay vì `a.clone()` dù cả hai làm cùng một việc: cách viết tường minh giúp người đọc
nhận ra ngay đây là một lượt tăng đếm rẻ, không phải một deep clone tốn kém như `.clone()` thường
ngụ ý ở hầu hết kiểu khác. `Rc<T>` chỉ an toàn dùng trong ngữ cảnh **đơn luồng** — chia sẻ giữa
nhiều luồng cần `Arc<T>`, nói ở [chương Concurrency](../ch14-concurrency/ch14-01-threads.md).

## RefCell\<T\>: khả biến nội tại

Borrow checker kiểm tra quy tắc mượn (một khả biến, hoặc nhiều bất biến) **lúc biên dịch**.
`RefCell<T>` chuyển việc kiểm tra đó sang **lúc chạy** — cho phép sửa dữ liệu ngay cả qua một tham
chiếu bất biến tới `RefCell`, đổi lại nếu vi phạm quy tắc mượn, chương trình **panic lúc chạy** thay
vì báo lỗi biên dịch:

```rust
use std::cell::RefCell;

fn main() {
    let so = RefCell::new(5);

    {
        let mut so_mut = so.borrow_mut();
        *so_mut += 10;
    }

    println!("{}", so.borrow());
}
```

`borrow()` trả về `Ref<T>` (như mượn bất biến), `borrow_mut()` trả `RefMut<T>` (như mượn khả biến) —
`RefCell` tự đếm số lượt mượn đang tồn tại lúc chạy, y hệt logic borrow checker vẫn làm, chỉ khác
thời điểm kiểm tra:

```rust,should_panic
use std::cell::RefCell;

fn main() {
    let so = RefCell::new(5);

    let _muon_1 = so.borrow_mut();
    let _muon_2 = so.borrow_mut(); // panic lúc chạy: đã có một mượn khả biến khác

    println!("sẽ không chạy tới đây");
}
```

Đánh đổi này có ý nghĩa trong tình huống cụ thể: một giá trị **nên** hiện ra bất biến với code gọi
nó (giữ đúng hợp đồng API), nhưng nội bộ cần tự sửa một trạng thái (ví dụ bộ đếm số lần gọi để phục
vụ test, hay một cache). Trình biên dịch không đủ thông tin tĩnh để xác nhận cách dùng đó luôn an
toàn, nhưng người viết code biết chắc theo logic chương trình — `RefCell` cho phép thực hiện điều
đó, trả giá bằng khả năng panic nếu logic đó sai.

### Kết hợp Rc\<RefCell\<T\>\>

Nhiều chủ sở hữu (`Rc`) và mỗi chủ sở hữu đều sửa được dữ liệu dùng chung (`RefCell`) là một tổ hợp
rất phổ biến trong code đơn luồng:

```rust
use std::cell::RefCell;
use std::rc::Rc;

fn main() {
    let gia_tri_chung = Rc::new(RefCell::new(5));

    let a = Rc::clone(&gia_tri_chung);
    let b = Rc::clone(&gia_tri_chung);

    *a.borrow_mut() += 10;
    *b.borrow_mut() += 100;

    println!("Giá trị cuối: {}", gia_tri_chung.borrow());
}
```
