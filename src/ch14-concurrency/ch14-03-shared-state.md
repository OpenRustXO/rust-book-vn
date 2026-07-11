# Shared state

Message passing tránh chia sẻ trực tiếp, nhưng đôi khi nhiều luồng thật sự cần cùng đọc/ghi một vùng
dữ liệu — ví dụ một bộ đếm dùng chung. `Mutex<T>` (mutual exclusion) cho phép điều đó: tại một thời
điểm, chỉ đúng một luồng được quyền truy cập dữ liệu bên trong.

```rust
use std::sync::Mutex;

fn main() {
    let m = Mutex::new(5);

    {
        let mut so = m.lock().unwrap();
        *so += 1;
    }

    println!("m = {m:?}");
}
```

`m.lock()` chặn luồng gọi cho tới khi giành được khoá, trả `Result<MutexGuard<T>, _>` — `Err` khi
một luồng khác đã panic trong lúc giữ khoá đó (gọi là "poisoned", tránh việc luồng sau nhận một
trạng thái dữ liệu có thể đã dở dang). `MutexGuard<T>` implement `Deref` (dùng `*so` để truy cập dữ
liệu, giống `Ref`/`RefMut` của `RefCell` ở chương trước) và `Drop` — khoá **tự động nhả** ngay khi
`MutexGuard` ra khỏi scope, không cần tự tay gọi hàm mở khoá.

## Arc\<T\>: Rc an toàn giữa nhiều luồng

Chia sẻ một `Mutex<T>` giữa nhiều luồng cần nhiều chủ sở hữu — nhưng `Rc<T>` (chương Con trỏ thông
minh) không an toàn giữa nhiều luồng (bộ đếm của nó không đồng bộ hoá, hai luồng cùng tăng/giảm đếm
đồng thời chính là một data race). `Arc<T>` (Atomic Reference Counted) làm y hệt việc `Rc<T>` làm,
nhưng dùng phép toán đếm an toàn giữa nhiều luồng — đánh đổi lấy một chi phí nhỏ lúc chạy so với
`Rc<T>`, nên chỉ dùng `Arc` khi thật sự cần chia sẻ giữa nhiều luồng:

```rust
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    let dem = Arc::new(Mutex::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        let dem = Arc::clone(&dem);
        let handle = thread::spawn(move || {
            let mut gia_tri = dem.lock().unwrap();
            *gia_tri += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Kết quả: {}", *dem.lock().unwrap());
}
```

`Arc<Mutex<T>>` song song hoàn toàn với `Rc<RefCell<T>>` đã gặp ở chương trước — cùng một ý tưởng
(nhiều chủ sở hữu cùng có thể sửa dữ liệu dùng chung), chỉ khác `Arc`/`Mutex` là phiên bản an toàn
giữa nhiều luồng, còn `Rc`/`RefCell` chỉ dùng được trong một luồng. Rủi ro cũng đổi khác tương ứng:
dùng sai `RefCell` gây panic lúc chạy; dùng sai `Mutex` (ví dụ hai khoá lồng nhau theo thứ tự ngược
nhau ở hai luồng) có thể gây **deadlock** — cả hai luồng cùng chờ nhau vĩnh viễn. Rust không phát
hiện được deadlock tại thời điểm biên dịch, đây vẫn là trách nhiệm thiết kế của người viết code.
