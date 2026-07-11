# Send và Sync

Hai trait đánh dấu (marker trait — không có method nào, chỉ tồn tại để gắn nhãn cho kiểu) là nền
tảng thật sự khiến "concurrency không sợ hãi" thành hiện thực: `Send` và `Sync`. Phần lớn thời gian
không cần biết tới chúng, vì trình biên dịch **tự động** implement cả hai cho mọi kiểu cấu thành
hoàn toàn từ các phần đã là `Send`/`Sync` — nhưng chính hai trait này là lý do những lỗi như "dùng
`Rc` giữa nhiều luồng" bị chặn ngay lúc biên dịch ở chương trước.

## Send: chuyển quyền sở hữu qua luồng khác an toàn

Một kiểu implement `Send` nếu quyền sở hữu giá trị của nó **chuyển được** sang một luồng khác an
toàn — hầu hết kiểu trong Rust là `Send`. `Rc<T>` là ngoại lệ đáng chú ý: **không** implement
`Send`, vì bộ đếm bên trong nó không đồng bộ hoá — chuyển một `Rc` sang luồng khác trong khi luồng
gốc vẫn giữ bản sao khác sẽ mở đường cho hai luồng cùng tăng/giảm đếm không an toàn. Đây chính xác
là lý do `thread::spawn` đòi hỏi closure phải `'static` và mọi giá trị move vào phải `Send` — cố
move một `Rc<T>` vào một closure cho `thread::spawn` là lỗi biên dịch, không phải lỗi runtime ẩn.

## Sync: chia sẻ tham chiếu qua luồng khác an toàn

Một kiểu implement `Sync` nếu `&T` (tham chiếu bất biến tới nó) an toàn để nhiều luồng cùng cầm —
nói cách khác, `T` là `Sync` khi và chỉ khi `&T` là `Send`. `Mutex<T>` là `Sync` (đó là toàn bộ lý
do nó tồn tại — tự đồng bộ hoá truy cập). `RefCell<T>` thì **không** — nó chỉ kiểm tra quy tắc mượn
lúc chạy trong ngữ cảnh một luồng duy nhất, không có cơ chế đồng bộ nào chống lại hai luồng cùng gọi
`borrow_mut()` đồng thời. Đây là lý do chia sẻ trạng thái khả biến giữa các luồng cần `Mutex<T>`,
chứ không dùng lại được `RefCell<T>`.

## Vì sao điều này quan trọng

`Send` và `Sync` biến "kiểu dữ liệu này có an toàn để dùng giữa nhiều luồng hay không" từ một câu
hỏi phải tự nhớ và kiểm tra thủ công, thành một **thuộc tính của hệ thống kiểu** mà trình biên dịch
tự kiểm tra ở mọi nơi kiểu đó được dùng — cùng triết lý với quyền sở hữu và borrow checker: thay vì
tài liệu hoá quy tắc rồi hy vọng không ai viết sai, mã hoá thẳng quy tắc đó vào kiểu để trình biên
dịch từ chối biên dịch code vi phạm.

Cả hai trait đều **tự động derive** — không tự tay implement cho kiểu của mình trong phần lớn trường
hợp, trừ khi đang viết code cấp thấp dùng `unsafe` để tự quản lý một thứ vốn không an toàn giữa
nhiều luồng theo mặc định (nói ở
[chương Tính năng nâng cao](../ch18-advanced/ch18-01-unsafe-rust.md)) — tự implement `Send`/`Sync`
thủ công là một cam kết "tôi tự đảm bảo điều này an toàn", trình biên dịch không tự kiểm tra giúp
được nữa một khi đã dùng `unsafe impl`.
