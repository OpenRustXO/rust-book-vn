# Trait nâng cao

## Associated type

`Iterator` (chương Closure và Iterator) khai báo `type Item` thay vì một tham số generic
`Iterator<T>`. Khác biệt cốt lõi: với **associated type**, một kiểu chỉ implement `Iterator` được
**đúng một lần**, `Item` cụ thể do chính kiểu đó quyết định cố định; với generic trait (`Trait<T>`),
một kiểu implement được **nhiều lần** cho nhiều `T` khác nhau. `Iterator` chọn associated type vì
hợp lý về mặt ngữ nghĩa: một kiểu chỉ nên cho ra đúng một loại phần tử khi duyệt, không có lý do gì
để cùng một kiểu vừa là `Iterator<i32>` vừa là `Iterator<String>`.

## Tham số kiểu generic mặc định

`Add` (trait cho toán tử `+`) khai báo `trait Add<Rhs = Self>` — tham số `Rhs` (vế phải) mặc định
bằng chính `Self` nếu không chỉ định gì khác, cho phép vừa cộng hai giá trị cùng kiểu theo cách tự
nhiên, vừa ghi đè để cộng được với một kiểu khác hẳn khi cần:

```rust
use std::ops::Add;

#[derive(Debug, Clone, Copy)]
struct Diem {
    x: i32,
    y: i32,
}

impl Add for Diem {
    // dùng Rhs mặc định = Self
    type Output = Diem;
    fn add(self, khac: Diem) -> Diem {
        Diem {
            x: self.x + khac.x,
            y: self.y + khac.y,
        }
    }
}

struct Milimet(u32);
struct Met(u32);

impl Add<Met> for Milimet {
    // ghi đè Rhs thành Met
    type Output = Milimet;
    fn add(self, khac: Met) -> Milimet {
        Milimet(self.0 + khac.0 * 1000)
    }
}

fn main() {
    let p1 = Diem { x: 1, y: 2 };
    let p2 = Diem { x: 3, y: 4 };
    println!("{:?}", p1 + p2);

    let tong = Milimet(10) + Met(1);
    println!("{} mm", tong.0);
}
```

## Cú pháp đủ điều kiện để gỡ nhập nhằng

Hai trait khác nhau (hoặc một trait và một method riêng của kiểu) có thể cùng đặt tên method trùng
nhau. Gọi `.method()` thường sẽ ưu tiên method riêng của kiểu nếu có; muốn gọi đích danh method của
một trait cụ thể, cần cú pháp đủ điều kiện:

```rust
trait KeuGi {
    fn keu(&self) -> String {
        String::from("...")
    }
}

trait KeuGiKhac {
    fn keu(&self) -> String {
        String::from("???")
    }
}

struct Vit;
impl KeuGi for Vit {
    fn keu(&self) -> String {
        String::from("Cạc cạc")
    }
}
impl KeuGiKhac for Vit {
    fn keu(&self) -> String {
        String::from("Quạc")
    }
}

fn main() {
    let v = Vit;
    println!("{}", KeuGi::keu(&v));
    println!("{}", <Vit as KeuGiKhac>::keu(&v));
}
```

`KeuGi::keu(&v)` là dạng gọn (đủ dùng khi trình biên dịch tự suy ra được `Self` từ tham số); dạng
đầy đủ `<Vit as KeuGiKhac>::keu(&v)` cần thiết khi cả tên trait lẫn kiểu đều phải nói rõ mới hết
nhập nhằng.

## Supertrait

Một trait có thể yêu cầu kiểu implement nó phải **đồng thời** implement một trait khác — gọi là
supertrait — cho phép dùng method của trait kia ngay trong phần mặc định của trait này:

```rust
use std::fmt;

trait HienThiDep: fmt::Display {
    fn in_dep(&self) {
        println!("*** {self} ***");
    }
}

struct Diem {
    x: i32,
    y: i32,
}

impl fmt::Display for Diem {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "({}, {})", self.x, self.y)
    }
}

impl HienThiDep for Diem {}

fn main() {
    let p = Diem { x: 1, y: 2 };
    p.in_dep();
}
```

`{self}` bên trong `in_dep` chỉ hợp lệ vì `HienThiDep: fmt::Display` đảm bảo mọi kiểu implement
`HienThiDep` cũng implement `Display` — thiếu ràng buộc supertrait đó, trình biên dịch không có gì
đảm bảo `self` in ra được.

## Newtype pattern để lách quy tắc mồ côi

Quy tắc mồ côi (chương Trait) cấm implement một trait ngoài crate cho một kiểu cũng ngoài crate. Bọc
kiểu ngoài trong một tuple struct của riêng mình (newtype) tạo ra một kiểu **nội bộ** crate, hợp lệ
để implement bất kỳ trait ngoài nào lên nó:

```rust
use std::fmt;

struct Bao(Vec<String>);

impl fmt::Display for Bao {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "[{}]", self.0.join(", "))
    }
}

fn main() {
    let b = Bao(vec![String::from("hello"), String::from("world")]);
    println!("{b}");
}
```

`Vec<String>` (kiểu ngoài) và `Display` (trait ngoài) không thể implement trực tiếp cho nhau trong
crate của mình — `Bao` là một kiểu mới, dù chỉ bọc đúng một `Vec<String>` bên trong, đã đủ điều kiện
để implement `Display` hợp lệ theo quy tắc mồ côi. Đánh đổi: `Bao` không tự động có sẵn các method
của `Vec<String>` — phải truy cập qua `self.0`, hoặc implement `Deref` (chương Con trỏ thông minh)
nếu muốn dùng xuyên qua như thể chính là `Vec<String>`.
