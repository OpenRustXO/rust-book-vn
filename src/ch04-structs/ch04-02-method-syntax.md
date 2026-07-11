# Method

**Method** là một hàm gắn với một kiểu cụ thể, khai báo bên trong khối `impl` của kiểu đó, tham số
đầu tiên luôn là một dạng của `self` — chỉ chính instance đang gọi method.

```rust
#[derive(Debug)]
struct HinhChuNhat {
    rong: u32,
    cao: u32,
}

impl HinhChuNhat {
    fn dien_tich(&self) -> u32 {
        self.rong * self.cao
    }
}

fn main() {
    let hcn = HinhChuNhat { rong: 30, cao: 50 };
    println!("Diện tích: {}", hcn.dien_tich());
}
```

`&self` là viết tắt của `self: &Self` (`Self` là bí danh cho chính kiểu `impl` đang khai báo) — mượn
bất biến instance, đúng nhu cầu của `dien_tich` vì nó chỉ đọc, không sửa. Method cần sửa dữ liệu
nhận `&mut self`; method cần lấy quyền sở hữu instance (hiếm gặp, thường chỉ khi method biến
instance thành một kiểu khác rồi trả về) nhận `self` không tham chiếu.

## Tự động tham chiếu / giải tham chiếu

Gọi `hcn.dien_tich()` ở trên hoạt động dù `dien_tich` khai báo nhận `&self` chứ không phải nhận
thẳng `HinhChuNhat` — Rust tự thêm `&`, `&mut`, hoặc `*` khi cần để khớp chữ ký method, không cần
người gọi tự viết `(&hcn).dien_tich()`. Đây gọi là **tự động tham chiếu** (automatic referencing):
khi thấy `obj.method()`, Rust nhìn chữ ký `method` rồi tự quyết định `obj`, `&obj`, hay `&mut obj`
là thứ cần truyền, dựa vào đó mà không cần cú pháp con trỏ tường minh như một số ngôn ngữ hệ thống
khác (ví dụ C, nơi phải tự viết rõ `->` hay `.` tuỳ đang thao tác trên con trỏ hay giá trị).

## Nhiều tham số

Method nhận thêm tham số bình thường sau `self`:

```rust
struct HinhChuNhat {
    rong: u32,
    cao: u32,
}

impl HinhChuNhat {
    fn dien_tich(&self) -> u32 {
        self.rong * self.cao
    }

    fn co_the_chua(&self, cai_khac: &HinhChuNhat) -> bool {
        self.rong > cai_khac.rong && self.cao > cai_khac.cao
    }
}

fn main() {
    let hcn1 = HinhChuNhat { rong: 30, cao: 50 };
    let hcn2 = HinhChuNhat { rong: 10, cao: 40 };

    println!("hcn1 chứa được hcn2: {}", hcn1.co_the_chua(&hcn2));
}
```

## Hàm liên kết (không nhận self)

Một hàm khai báo trong `impl` nhưng không nhận tham số `self` nào gọi là **hàm liên kết**
(associated function) — không gắn với một instance cụ thể, gọi qua cú pháp `Kiểu::ten_ham` thay vì
`instance.ten_ham()`. `String::from` chính là một hàm liên kết. Dùng phổ biến nhất: viết một hàm
dựng sẵn đóng vai trò khởi tạo, tương tự "constructor" ở một số ngôn ngữ khác nhưng thực chất chỉ là
một hàm liên kết bình thường, không phải cú pháp đặc biệt riêng cho việc khởi tạo:

```rust
struct HinhChuNhat {
    rong: u32,
    cao: u32,
}

impl HinhChuNhat {
    fn hinh_vuong(canh: u32) -> Self {
        Self {
            rong: canh,
            cao: canh,
        }
    }
}

fn main() {
    let hv = HinhChuNhat::hinh_vuong(20);
    println!("{} {}", hv.rong, hv.cao);
}
```

Một kiểu có thể có nhiều khối `impl` riêng biệt — hợp lệ về cú pháp dù không có lý do đặc biệt để
tách trong ví dụ đơn giản này, nhưng hữu ích khi tổ chức code theo nhóm (ví dụ tách riêng phần
implement một trait, sẽ gặp ở chương sau) khỏi phần method thường của kiểu.
