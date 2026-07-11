# Design pattern hướng đối tượng

Rust không có kế thừa giữa các kiểu (một struct "kế thừa" field/method của struct khác) — chia sẻ
hành vi dùng trait (định nghĩa chung, mỗi kiểu tự implement hoặc dùng bản mặc định) kết hợp
composition (một struct chứa struct khác làm field) thay cho kế thừa. Nhiều design pattern hướng đối
tượng kinh điển vẫn áp dụng được, chỉ đổi cách hiện thực cho hợp với mô hình trait + composition
này. Ví dụ tiêu biểu: **state pattern** — hành vi của một giá trị thay đổi tuỳ trạng thái nội tại,
không cần `if` kiểm tra trạng thái rải rác khắp nơi.

## Ví dụ: quy trình xuất bản bài viết

Một bài viết đi qua ba trạng thái: **Nháp** → **Chờ duyệt** → **Đã xuất bản**, chỉ hiển thị nội dung
khi đã xuất bản. Mỗi trạng thái là một kiểu implement chung một trait `TrangThai`, giữ trong bài
viết qua một trait object:

```rust
trait TrangThai {
    fn yeu_cau_duyet(self: Box<Self>) -> Box<dyn TrangThai>;
    fn duyet(self: Box<Self>) -> Box<dyn TrangThai>;
    fn noi_dung<'a>(&self, _bai_viet: &'a BaiViet) -> &'a str {
        ""
    }
}

struct Nhap;
impl TrangThai for Nhap {
    fn yeu_cau_duyet(self: Box<Self>) -> Box<dyn TrangThai> {
        Box::new(ChoDuyet)
    }
    fn duyet(self: Box<Self>) -> Box<dyn TrangThai> {
        self // chưa qua chờ duyệt thì duyệt không có tác dụng
    }
}

struct ChoDuyet;
impl TrangThai for ChoDuyet {
    fn yeu_cau_duyet(self: Box<Self>) -> Box<dyn TrangThai> {
        self // đã ở trạng thái chờ duyệt, yêu cầu lại không đổi gì
    }
    fn duyet(self: Box<Self>) -> Box<dyn TrangThai> {
        Box::new(DaXuatBan)
    }
}

struct DaXuatBan;
impl TrangThai for DaXuatBan {
    fn yeu_cau_duyet(self: Box<Self>) -> Box<dyn TrangThai> {
        self
    }
    fn duyet(self: Box<Self>) -> Box<dyn TrangThai> {
        self
    }
    fn noi_dung<'a>(&self, bai_viet: &'a BaiViet) -> &'a str {
        &bai_viet.noi_dung
    }
}

struct BaiViet {
    trang_thai: Option<Box<dyn TrangThai>>,
    noi_dung: String,
}

impl BaiViet {
    fn moi() -> BaiViet {
        BaiViet {
            trang_thai: Some(Box::new(Nhap)),
            noi_dung: String::new(),
        }
    }

    fn them_noi_dung(&mut self, text: &str) {
        self.noi_dung.push_str(text);
    }

    fn noi_dung(&self) -> &str {
        self.trang_thai.as_ref().unwrap().noi_dung(self)
    }

    fn yeu_cau_duyet(&mut self) {
        if let Some(s) = self.trang_thai.take() {
            self.trang_thai = Some(s.yeu_cau_duyet());
        }
    }

    fn duyet(&mut self) {
        if let Some(s) = self.trang_thai.take() {
            self.trang_thai = Some(s.duyet());
        }
    }
}

fn main() {
    let mut bai_viet = BaiViet::moi();
    bai_viet.them_noi_dung("Hôm nay trời đẹp");
    assert_eq!("", bai_viet.noi_dung());

    bai_viet.yeu_cau_duyet();
    assert_eq!("", bai_viet.noi_dung());

    bai_viet.duyet();
    assert_eq!("Hôm nay trời đẹp", bai_viet.noi_dung());
    println!("{}", bai_viet.noi_dung());
}
```

Vài điểm đáng chú ý trong cách hiện thực:

- Method `yeu_cau_duyet`/`duyet` nhận `self: Box<Self>` — **tiêu thụ** hẳn trạng thái cũ, trả về
  trạng thái mới, thay vì sửa tại chỗ. Chuyển trạng thái ở đây là **thay hẳn một giá trị bằng giá
  trị khác**, không phải chỉnh sửa cờ nội bộ của một giá trị duy nhất.
- `trang_thai` trong `BaiViet` là `Option<Box<dyn TrangThai>>` chứ không phải `Box<dyn TrangThai>`
  trần — vì method tiêu thụ `self: Box<Self>`, phải tạm thời lấy nó ra khỏi `BaiViet` (`take()`) mới
  chuyển được vào lời gọi, rồi gán trạng thái mới trả về ngược lại. `Option` chỉ đóng vai trò một
  chỗ chứa tạm để làm việc đó hợp lệ với borrow checker, không mang ý nghĩa "bài viết có thể không
  có trạng thái".
- Toàn bộ logic "trạng thái nào cho phép chuyển sang trạng thái nào" nằm gọn trong từng impl của
  từng trạng thái — thêm một trạng thái mới không đụng gì tới code của `BaiViet` hay các trạng thái
  khác, chỉ cần viết đúng trait `TrangThai` cho trạng thái mới đó.

Cách tổ chức này đánh đổi một chút phức tạp cú pháp (`Box<Self>`, `Option` tạm) để đổi lấy: không có
`match`/`if` kiểm tra "đang ở trạng thái gì" rải rác khắp `BaiViet` — mỗi nơi chỉ cần gọi method,
hành vi đúng tự động tới từ trạng thái cụ thể đang được trỏ tới.
