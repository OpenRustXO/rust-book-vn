# Lập trình bất đồng bộ

Thread (chương trước) phù hợp cho công việc chiếm dụng CPU thật sự, chạy song song. Nhiều tác vụ
thực tế lại chủ yếu **chờ** — chờ phản hồi mạng, chờ đọc đĩa — mà một luồng OS dành riêng cho mỗi
tác vụ chờ như vậy khá lãng phí (mỗi luồng OS tốn bộ nhớ stack riêng, tốn chi phí chuyển ngữ cảnh).
Async giải quyết đúng bài toán đó: chạy hàng nghìn tác vụ đang chờ cùng lúc trên một số lượng nhỏ
luồng thật, bằng cách tạm dừng một tác vụ đang chờ và nhường chỗ cho tác vụ khác chạy tiếp.

## Future: một giá trị sẽ có sau

```rust,ignore
pub trait Future {
    type Output;
    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
}
```

`Future` chỉ có một method: `poll`, trả `Poll::Ready(gia_tri)` nếu đã xong, `Poll::Pending` nếu chưa
— và **tự nó không làm gì cả** nếu không ai gọi `poll`. Đây là điểm khác biệt cốt lõi so với luồng
(thread): một luồng OS tự chạy ngay khi được tạo; một `Future` chỉ là một mô tả "công việc sẽ làm",
cần một bên khác (executor) liên tục gọi `poll` cho tới khi `Ready`.

`async fn`/khối `async {}` là đường cú pháp: trình biên dịch tự biến phần thân thành một kiểu vô
danh implement `Future`, mỗi điểm `.await` là một chỗ có thể tạm dừng (trả `Pending`) và tiếp tục
đúng chỗ đó ở lượt `poll` sau — không cần tự tay viết `poll` bằng tay trong phần lớn trường hợp.

## std không có sẵn executor

Khác `thread::spawn` (chạy được ngay vì luồng OS là tài nguyên hệ điều hành cấp sẵn), thư viện chuẩn
Rust **chỉ định nghĩa** trait `Future` và cú pháp `async`/`.await` — không đi kèm executor nào để
thật sự chạy chúng. Trong dự án thực tế, việc chạy future dựa vào một crate runtime ngoài (phổ biến
nhất: `tokio`, `async-std`). Để thấy rõ bản chất cơ chế polling mà không phụ thuộc crate ngoài, viết
một executor tối giản chỉ bằng thư viện chuẩn:

```rust
use std::future::Future;
use std::pin::Pin;
use std::sync::Arc;
use std::task::{Context, Poll, Wake, Waker};

struct WakerKhongLamGi;

impl Wake for WakerKhongLamGi {
    fn wake(self: Arc<Self>) {}
}

fn chay<F: Future>(future: F) -> F::Output {
    let mut future = Box::pin(future);
    let waker = Waker::from(Arc::new(WakerKhongLamGi));
    let mut cx = Context::from_waker(&waker);

    loop {
        match future.as_mut().poll(&mut cx) {
            Poll::Ready(gia_tri) => return gia_tri,
            Poll::Pending => {}
        }
    }
}

async fn chao() -> i32 {
    42
}

fn main() {
    println!("{}", chay(chao()));
}
```

`chay` liên tục gọi `poll` trong một vòng lặp cho tới khi `Ready` — một executor thật không lặp bận
(busy-loop) như vậy: nó dùng `Waker` để **được báo lại** đúng lúc future sẵn sàng poll tiếp (ví dụ
hệ điều hành báo dữ liệu mạng đã tới), rồi ngủ chờ giữa các lần đó thay vì poll liên tục tốn CPU.
`WakerKhongLamGi` ở trên bỏ qua hoàn toàn cơ chế đó để giữ ví dụ tối giản.

Xem rõ vòng lặp poll hoạt động khi một future thật sự trả `Pending` trước khi `Ready`:

```rust
use std::future::Future;
use std::pin::Pin;
use std::sync::Arc;
use std::task::{Context, Poll, Wake, Waker};

struct ChoMotLuotPoll {
    da_poll: bool,
}

impl Future for ChoMotLuotPoll {
    type Output = &'static str;

    fn poll(mut self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output> {
        if self.da_poll {
            Poll::Ready("xong")
        } else {
            self.da_poll = true;
            cx.waker().wake_by_ref();
            Poll::Pending
        }
    }
}

struct WakerKhongLamGi;
impl Wake for WakerKhongLamGi {
    fn wake(self: Arc<Self>) {}
}

fn chay<F: Future>(future: F) -> F::Output {
    let mut future = Box::pin(future);
    let waker = Waker::from(Arc::new(WakerKhongLamGi));
    let mut cx = Context::from_waker(&waker);
    let mut so_lan_poll = 0;

    loop {
        so_lan_poll += 1;
        match future.as_mut().poll(&mut cx) {
            Poll::Ready(gia_tri) => {
                println!("Đã Ready sau {so_lan_poll} lượt poll");
                return gia_tri;
            }
            Poll::Pending => {}
        }
    }
}

fn main() {
    let ket_qua = chay(ChoMotLuotPoll { da_poll: false });
    println!("{ket_qua}");
}
```

`ChoMotLuotPoll` trả `Pending` ở lượt poll đầu, `Ready` ở lượt thứ hai — implement `poll` thủ công
chính xác là điều một khối `async {}` làm giúp một cách tự động và phức tạp hơn nhiều (theo dõi được
tạm dừng ở `.await` nào, khôi phục đúng chỗ đó ở lượt sau), nhưng bản chất cơ chế underneath là như
nhau.
