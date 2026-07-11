# Concurrency với async

## .await tuần tự không tự nhiên đồng thời

Nhiều điểm `.await` liên tiếp trong cùng một `async fn` chạy **tuần tự** — điểm thứ hai chỉ bắt đầu
sau khi điểm thứ nhất đã `Ready` hoàn toàn, giống hệt gọi hàm thường nối tiếp nhau. Tự bản thân
`async`/`.await` không tạo ra tính đồng thời — nó chỉ là cách viết một future tạm dừng được ở những
chỗ đang chờ, không phải phép màu tự động chạy song song mọi thứ có `.await`.

## Chạy nhiều future đồng thời

Muốn nhiều future thật sự tiến triển xen kẽ nhau (không cái nào phải xong hẳn mới tới lượt cái tiếp
theo), cần một trong hai cách:

- **spawn mỗi future thành một task riêng** trên runtime — bộ lập lịch của runtime tự luân phiên
  poll từng task độc lập.
- **gộp nhiều future vào một tổ hợp poll cùng nhau** trong một task duy nhất, thường qua macro
  `join!`/`select!`.

Cả hai cơ chế này đều **không có trong thư viện chuẩn** — chúng do crate runtime (`tokio`,
`async-std`, `futures`) cung cấp, vì việc điều phối nhiều future đúng đắn (đặc biệt là poll đan xen
an toàn, không vi phạm quy tắc `Pin`) là một bài toán runtime cần giải quyết trọn vẹn, không phải
thứ `std` tự làm thay. Với một runtime như `tokio`, code trông như sau (minh hoạ cú pháp, không chạy
được nếu thiếu crate `tokio`):

```rust,ignore
use tokio::join;

async fn tai_trang(url: &str) -> String {
    // giả lập gọi mạng
    format!("nội dung của {url}")
}

async fn tai_ca_hai() {
    let (a, b) = join!(tai_trang("a.example"), tai_trang("b.example"));
    println!("{a} — {b}");
}
```

`join!` chạy cả hai future **đồng thời**, chỉ trả về khi **cả hai** đã `Ready` — khác hẳn viết
`tai_trang("a...").await; tai_trang("b...").await;` (tuần tự, cái thứ hai đợi cái thứ nhất xong hẳn
mới bắt đầu).

`select!` chạy nhiều future đồng thời nhưng trả về ngay khi **future đầu tiên** hoàn thành, huỷ các
future còn lại — hữu ích cho các tình huống như đặt timeout (chạy đua giữa "thao tác chính" và "đồng
hồ đếm giờ", cái nào xong trước thắng):

```rust,ignore
use tokio::select;
use tokio::time::{sleep, Duration};

async fn vi_du_timeout() {
    select! {
        ket_qua = tai_du_lieu() => println!("Xong: {ket_qua}"),
        _ = sleep(Duration::from_secs(5)) => println!("Hết thời gian chờ"),
    }
}

async fn tai_du_lieu() -> String {
    String::from("dữ liệu")
}
```
