# Concurrency không sợ hãi

Chạy nhiều đoạn code độc lập cùng lúc luôn tiềm ẩn một lớp lỗi đặc trưng: hai luồng cùng đọc/ghi một
dữ liệu mà không đồng bộ (data race), thứ tự thực thi không cố định gây kết quả khác nhau giữa các
lần chạy, deadlock giữa nhiều khoá. Rust không loại bỏ được sự phức tạp cố hữu của lập trình đồng
thời, nhưng tái dùng đúng hệ thống quyền sở hữu/mượn đã học để bắt phần lớn lỗi đó **tại thời điểm
biên dịch** — đây là ý nghĩa của "concurrency không sợ hãi": lỗi đồng thời phổ biến trở thành lỗi
biên dịch thay vì bug ẩn chỉ lộ ra ngẫu nhiên lúc chạy.

## Thread

`thread::spawn` chạy một closure trên một luồng hệ điều hành mới, song song với luồng gọi nó:

```rust
use std::thread;
use std::time::Duration;

fn main() {
    let handle = thread::spawn(|| {
        for i in 1..3 {
            println!("số {i} từ luồng con");
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..3 {
        println!("số {i} từ luồng chính");
    }

    handle.join().unwrap();
}
```

`thread::spawn` trả về một `JoinHandle` — `.join()` chặn luồng gọi nó lại cho tới khi luồng kia chạy
xong. Thiếu `.join()`, chương trình có thể kết thúc (luồng `main` chạy xong, tiến trình thoát) trước
khi luồng con kịp hoàn thành — không có gì đảm bảo luồng con luôn chạy xong nếu không chờ tường
minh.

### Vì sao closure truyền vào thread::spawn thường cần move

```rust,compile_fail
use std::thread;

fn main() {
    let v = vec![1, 2, 3];

    let handle = thread::spawn(|| {
        println!("Vector: {v:?}");
    });

    handle.join().unwrap();
}
```

Lỗi biên dịch: trình biên dịch không biết luồng con sẽ chạy lâu bao lâu so với luồng `main` — nếu
closure chỉ mượn `v`, không gì đảm bảo `v` còn sống đủ lâu để luồng con đọc nó (`main` có thể drop
`v` trước khi luồng con kịp chạy tới dòng đó). Thêm `move` ép closure lấy quyền sở hữu `v`, loại bỏ
hẳn khả năng đó:

```rust
use std::thread;

fn main() {
    let v = vec![1, 2, 3];

    let handle = thread::spawn(move || {
        println!("Vector: {v:?}");
    });

    handle.join().unwrap();
}
```
