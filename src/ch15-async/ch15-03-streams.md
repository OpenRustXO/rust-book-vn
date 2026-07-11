# Stream

`Iterator` cho một chuỗi giá trị lấy được ngay lập tức, từng bước; **Stream** là phiên bản bất đồng
bộ của ý tưởng đó — một chuỗi giá trị mà mỗi bước lấy tiếp theo có thể phải **chờ** (một gói tin
mạng, một dòng dữ liệu đọc dần từ đĩa), tương ứng `poll_next` trả `Poll::Pending` khi phần tử kế
tiếp chưa sẵn sàng, `Poll::Ready(Some(gia_tri))` khi có, `Poll::Ready(None)` khi hết:

```rust,ignore
trait Stream {
    type Item;
    fn poll_next(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Option<Self::Item>>;
}
```

Trait `Stream` **chưa** ổn định trong thư viện chuẩn tại thời điểm viết sách này — dùng qua crate
ngoài (`futures::stream::Stream`, hoặc tương đương do runtime cung cấp), tương tự tình huống
`Future` tự chạy được cần một executor ngoài `std`.

## Duyệt qua Stream

Runtime/crate cung cấp method tiện dụng để duyệt Stream gần giống `for` với `Iterator`, chỉ khác mỗi
bước cần `.await`:

```rust,ignore
use futures::stream::{self, StreamExt};

async fn vi_du() {
    let mut stream = stream::iter(vec![1, 2, 3]);

    while let Some(gia_tri) = stream.next().await {
        println!("{gia_tri}");
    }
}
```

## Kết hợp adaptor như Iterator

Stream hỗ trợ các adaptor quen thuộc từ `Iterator` (`map`, `filter`, `take`, ...) — vẫn giữ nguyên
tính lười: một chuỗi `.map(...).filter(...)` trên Stream không tự chạy gì cho tới khi có thứ gì đó
thật sự `.await` từng phần tử ra, y hệt tinh thần lười của `Iterator` đã học ở chương Closure và
Iterator, chỉ khác mỗi bước giờ có thể tạm dừng chờ thay vì luôn có sẵn ngay:

```rust,ignore
use futures::stream::{self, StreamExt};

async fn vi_du() {
    let tong: i32 = stream::iter(vec![1, 2, 3, 4])
        .map(|x| x * 2)
        .filter(|x| {
            let giu_lai = *x > 2;
            async move { giu_lai }
        })
        .fold(0, |tong, x| async move { tong + x })
        .await;

    println!("{tong}");
}
```

Cùng một tư duy xử lý dữ liệu theo chuỗi phép biến đổi khai báo được (declarative) như `Iterator`,
mở rộng sang ngữ cảnh mỗi bước có thể cần chờ — không phải một mô hình tư duy hoàn toàn mới.
