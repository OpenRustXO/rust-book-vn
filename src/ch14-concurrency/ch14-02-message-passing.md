# Message passing

Một cách phổ biến để các luồng phối hợp an toàn: thay vì cùng truy cập chung một vùng dữ liệu, chúng
gửi **giá trị** cho nhau qua một kênh (channel) — "không chia sẻ bộ nhớ để giao tiếp, mà giao tiếp
bằng cách chuyển quyền sở hữu qua lại".

```rust
use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let gia_tri = String::from("xin chào");
        tx.send(gia_tri).unwrap();
    });

    let nhan_duoc = rx.recv().unwrap();
    println!("Nhận được: {nhan_duoc}");
}
```

`mpsc::channel()` (multiple producer, single consumer) trả về cặp `(Sender, Receiver)`. `tx.send(v)`
**move** quyền sở hữu `v` vào kênh — đây là điểm mấu chốt: sau khi gửi, luồng gửi **không còn quyền
truy cập** `v` nữa, nên không thể có chuyện cả luồng gửi lẫn luồng nhận cùng đọc/ghi cùng một giá
trị cùng lúc. Borrow checker biến quy tắc thiết kế "đừng chia sẻ dữ liệu đang gửi" thành một ràng
buộc bắt buộc, không phải quy ước hi vọng người viết code tự tuân theo:

```rust,compile_fail
use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let gia_tri = String::from("xin chào");
        tx.send(gia_tri).unwrap();
        println!("vẫn dùng gia_tri: {gia_tri}"); // lỗi: gia_tri đã bị move vào send
    });

    println!("{}", rx.recv().unwrap());
}
```

`rx.recv()` chặn luồng gọi tới khi có giá trị, trả `Result` — `Err` khi mọi `Sender` đã bị drop (sẽ
không bao giờ có giá trị mới gửi tới nữa). Có thể duyệt `rx` trực tiếp như iterator, tự động chặn
chờ ở mỗi lần lặp và kết thúc khi kênh đóng:

```rust
use std::sync::mpsc;
use std::thread;
use std::time::Duration;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let cac_gia_tri = vec![
            String::from("xin"),
            String::from("chào"),
            String::from("từ"),
            String::from("luồng con"),
        ];

        for gia_tri in cac_gia_tri {
            tx.send(gia_tri).unwrap();
            thread::sleep(Duration::from_millis(1));
        }
    });

    for nhan_duoc in rx {
        println!("Nhận được: {nhan_duoc}");
    }
}
```

## Nhiều luồng gửi

`Sender` implement `Clone` — nhân bản để nhiều luồng cùng gửi vào **một** `Receiver`:

```rust
use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel();
    let tx2 = tx.clone();

    thread::spawn(move || {
        tx.send(String::from("từ luồng 1")).unwrap();
    });

    thread::spawn(move || {
        tx2.send(String::from("từ luồng 2")).unwrap();
    });

    for nhan_duoc in rx {
        println!("Nhận được: {nhan_duoc}");
    }
}
```
