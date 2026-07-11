# Trait

**Trait** định nghĩa một tập hành vi mà nhiều kiểu khác nhau có thể cùng implement — giống một bản
cam kết "kiểu nào implement trait này thì chắc chắn có các method sau".

```rust
pub trait TomTat {
    fn tom_tat(&self) -> String;
}

pub struct BaiBao {
    pub tieu_de: String,
    pub tac_gia: String,
}

impl TomTat for BaiBao {
    fn tom_tat(&self) -> String {
        format!("{}, bởi {}", self.tieu_de, self.tac_gia)
    }
}

pub struct Tweet {
    pub ten_nguoi_dung: String,
    pub noi_dung: String,
}

impl TomTat for Tweet {
    fn tom_tat(&self) -> String {
        format!("@{}: {}", self.ten_nguoi_dung, self.noi_dung)
    }
}

fn main() {
    let bai_bao = BaiBao {
        tieu_de: String::from("Rust 2024"),
        tac_gia: String::from("Đội ngũ Rust"),
    };
    println!("{}", bai_bao.tom_tat());
}
```

`BaiBao` và `Tweet` không liên quan gì nhau về cấu trúc dữ liệu, nhưng cùng implement `TomTat` — bất
kỳ hàm nào chỉ cần "một thứ biết tự tóm tắt" đều nhận được cả hai, không cần quan tâm chúng thật sự
là gì.

Một ràng buộc gọi là **quy tắc mồ côi** (orphan rule): chỉ implement được một trait cho một kiểu nếu
**ít nhất một trong hai** — trait hoặc kiểu — được định nghĩa ngay trong crate hiện tại. Không thể
implement một trait của thư viện ngoài cho một kiểu cũng của thư viện ngoài khác. Ràng buộc này đảm
bảo tính nhất quán (coherence): không bao giờ có hai crate khác nhau cùng implement một trait cho
cùng một kiểu theo hai cách khác nhau, gây xung đột không thể giải quyết khi cả hai được dùng chung
trong một chương trình.

## Trait mặc định

Một method trong trait có thể có sẵn phần thân — kiểu implement trait được quyền dùng luôn bản mặc
định, hoặc ghi đè nếu cần:

```rust
pub trait TomTat {
    fn tom_tat(&self) -> String {
        String::from("(Đọc thêm...)")
    }
}

pub struct BaiBao {
    pub tieu_de: String,
}

impl TomTat for BaiBao {} // dùng nguyên bản mặc định, không ghi đè gì

fn main() {
    let bb = BaiBao {
        tieu_de: String::from("Rust 2024"),
    };
    println!("{}", bb.tom_tat());
}
```

## Trait làm tham số

Nhận một tham số "kiểu nào cũng được, miễn implement trait X" bằng `impl Trait`:

```rust
pub trait TomTat {
    fn tom_tat(&self) -> String;
}

pub fn thong_bao(item: &impl TomTat) {
    println!("Tin mới! {}", item.tom_tat());
}
```

`&impl TomTat` là cách viết gọn của cú pháp ràng buộc trait đầy đủ hơn:

```rust,ignore
pub fn thong_bao<T: TomTat>(item: &T) {
    println!("Tin mới! {}", item.tom_tat());
}
```

Hai cách viết tương đương khi chỉ có một tham số; dạng `<T: TomTat>` cần thiết khi muốn ép **hai**
tham số phải cùng một kiểu cụ thể (`fn ham<T: TomTat>(a: &T, b: &T)` — `a` và `b` buộc cùng kiểu;
viết `&impl TomTat` cho cả hai tham số cho phép `a`, `b` là hai kiểu khác nhau, miễn cả hai đều
implement `TomTat`).

Nhiều ràng buộc nối bằng `+`, hoặc tách ra `where` cho dễ đọc khi ràng buộc dài:

```rust,ignore
fn ham<T: TomTat + Clone>(item: &T) {}

fn ham_khac<T>(item: &T)
where
    T: TomTat + Clone,
{
}
```

## Trả về một kiểu implement trait

```rust
pub trait TomTat {
    fn tom_tat(&self) -> String;
}

pub struct Tweet {
    pub noi_dung: String,
}

impl TomTat for Tweet {
    fn tom_tat(&self) -> String {
        self.noi_dung.clone()
    }
}

fn tao_tweet() -> impl TomTat {
    Tweet {
        noi_dung: String::from("nội dung mới"),
    }
}

fn main() {
    println!("{}", tao_tweet().tom_tat());
}
```

`impl TomTat` ở vị trí trả về nghĩa là "trả về một kiểu cụ thể nào đó implement `TomTat`, không tiết
lộ chính xác là kiểu gì" — hữu ích khi kiểu trả về thật sự dài dòng để viết ra (ví dụ closure ở
chương sau) hoặc không muốn cam kết công khai đúng kiểu cụ thể. Giới hạn quan trọng: cách này chỉ
dùng được khi hàm luôn trả về **đúng một** kiểu cụ thể duy nhất ở mọi nhánh — không thể trả `Tweet`
ở nhánh này và `BaiBao` ở nhánh khác dù cả hai đều implement `TomTat`, vì trình biên dịch cần biết
kích thước chính xác của kiểu trả về ngay tại thời điểm biên dịch. Khi thật sự cần trả về nhiều kiểu
cụ thể khác nhau tuỳ điều kiện, cần tới trait object — nói ở
[chương Trait object và hướng đối tượng](../ch16-oop/ch16-01-trait-objects.md).
