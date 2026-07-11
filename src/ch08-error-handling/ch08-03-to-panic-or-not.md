# Khi nào nên panic

Không có công thức cứng, nhưng có một câu hỏi định hướng: **người gọi hàm này có nên có cơ hội quyết
định phải làm gì tiếp theo hay không?** Nếu có, trả `Result` — để họ tự chọn: thử lại, dùng giá trị
mặc định, hay dừng hẳn. Nếu không — vì tiếp tục chạy đã là không an toàn, vô nghĩa, hoặc chính bản
thân việc gọi hàm với đầu vào như vậy đã là một lỗi lập trình — `panic!` là lựa chọn đúng.

## Prototype, ví dụ, và test

Trong code ví dụ minh hoạ hay bản nháp đang phát triển, `unwrap()`/`expect()` là lựa chọn hợp lý để
đánh dấu rõ "chỗ này cần xử lý lỗi tử tế sau" mà không phải viết `match` đầy đủ ngay — miễn nhớ quay
lại trước khi coi là code sẵn sàng dùng thật. Trong test, một lệnh gọi panic khi có lỗi chính là
điều muốn xảy ra: test cần thất bại rõ ràng, không nên âm thầm bỏ qua lỗi rồi báo test pass.

## Khi biết nhiều hơn trình biên dịch

Đôi khi logic của người viết code đảm bảo một `Result` chắc chắn là `Ok`, nhưng trình biên dịch
không có cách nào biết điều đó chỉ từ kiểu dữ liệu:

```rust
use std::net::IpAddr;

fn main() {
    let dia_chi_nha: IpAddr = "127.0.0.1"
        .parse()
        .expect("chuỗi hardcode nên chắc chắn là địa chỉ IP hợp lệ");

    println!("{dia_chi_nha}");
}
```

Chuỗi `"127.0.0.1"` là hằng viết cứng trong code, không đến từ input người dùng — không có cách nào
nó parse lỗi trừ khi tự gõ sai. `expect` với thông báo giải thích **vì sao** tin chắc là `Ok` vẫn
tốt hơn hẳn so với im lặng giả định — nếu giả định đó một ngày sai (ví dụ ai đó sửa chuỗi thành giá
trị không hợp lệ), panic sẽ chỉ thẳng đúng chỗ, kèm lý do đã tưởng nó không thể sai.

## Khi nào nên chủ động panic

Cân nhắc panic khi **cả ba** điều sau đúng:

1. Trạng thái hiện tại là bất thường, không phải một khả năng xảy ra định kỳ trong vận hành bình
   thường (khác lỗi "hết dung lượng đĩa" hay "không có mạng" — những thứ chắc chắn sẽ xảy ra vào lúc
   nào đó và cần code gọi được xử lý qua `Result`).
2. Code phía sau điểm đó **cần** giả định trạng thái đã hợp lệ để chạy tiếp an toàn.
3. Không có cách hợp lý nào mã hoá ràng buộc đó vào hệ thống kiểu để trình biên dịch tự bắt lỗi
   thay.

Gọi một hàm với giá trị vi phạm hợp đồng của nó (ví dụ số âm cho một hàm chỉ nhận số dương) là lỗi
của **code gọi**, không phải điều hàm đó nên trả `Result` để người gọi "xử lý" — không có cách xử lý
hợp lý nào ngoài việc sửa lại code gọi cho đúng.

## Mã hoá ràng buộc vào kiểu dữ liệu

Thay vì kiểm tra cùng một điều kiện rải rác ở nhiều nơi trong code, gói điều kiện đó vào một kiểu
riêng, kiểm tra **một lần duy nhất** lúc khởi tạo:

```rust
pub struct PhanTram {
    gia_tri: i32,
}

impl PhanTram {
    pub fn moi(gia_tri: i32) -> PhanTram {
        if !(0..=100).contains(&gia_tri) {
            panic!("Phần trăm phải trong khoảng 0-100, nhận được {gia_tri}");
        }

        PhanTram { gia_tri }
    }

    pub fn gia_tri(&self) -> i32 {
        self.gia_tri
    }
}

fn main() {
    let hoan_thanh = PhanTram::moi(75);
    println!("{}%", hoan_thanh.gia_tri());
}
```

Vì `gia_tri` là field riêng tư và chỉ tạo được `PhanTram` qua `moi` (đã kiểm tra ràng buộc), **mọi**
hàm khác nhận tham số kiểu `PhanTram` được quyền tin tưởng giá trị bên trong luôn nằm trong khoảng
0-100 mà không cần tự kiểm tra lại — panic xảy ra đúng một chỗ, tại điểm vi phạm hợp đồng thật sự
(lúc khởi tạo với giá trị sai), thay vì để giá trị sai âm thầm chui sâu vào logic rồi gây lỗi khó dò
ở một chỗ xa hoàn toàn không liên quan tới nguyên nhân gốc.
