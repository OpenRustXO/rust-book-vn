# Đường dẫn, use, và tách file

## Đường dẫn tuyệt đối và tương đối

Gọi một item qua cây module cần một đường dẫn, theo một trong hai dạng: **tuyệt đối**, bắt đầu từ
gốc crate bằng từ khoá `crate`, hoặc **tương đối**, bắt đầu từ module hiện tại (`self`, hoặc thẳng
tên item nếu cùng module) — dùng `super` để đi lên module cha, tương tự `..` trong đường dẫn thư
mục:

```rust
mod nha_bep {
    pub fn nau_an() {
        sua_soan_nguyen_lieu();
    }

    fn sua_soan_nguyen_lieu() {
        println!("Đang sơ chế");
    }

    pub mod trang_tri {
        pub fn xep_dia() {
            super::sua_soan_nguyen_lieu(); // super trỏ về nha_bep, module cha
        }
    }
}

fn phuc_vu() {
    crate::nha_bep::nau_an(); // đường dẫn tuyệt đối, từ gốc crate
    nha_bep::nau_an(); // đường dẫn tương đối, vì phuc_vu ở cùng cấp với nha_bep
    nha_bep::trang_tri::xep_dia();
}

fn main() {
    phuc_vu();
}
```

Chọn tuyệt đối hay tương đối phần lớn là vấn đề sở thích — tương đối tiện khi hay di chuyển cả một
khối module đi nơi khác trong cây (các đường dẫn `super`/tên module con vẫn đúng), tuyệt đối rõ ràng
hơn khi đường dẫn dài hoặc code hay bị cắt dán sang chỗ khác.

## use: đưa đường dẫn vào scope

Gõ lại đường dẫn đầy đủ mỗi lần dùng một item rất dài dòng. `use` đưa một đường dẫn vào scope hiện
tại để dùng thẳng tên ngắn:

```rust
mod nha_hang {
    pub mod nha_bep {
        pub fn nau_an() {
            println!("Đang nấu");
        }
    }
}

use crate::nha_hang::nha_bep;

fn main() {
    nha_bep::nau_an();
}
```

Quy ước thành ngữ: với **hàm**, `use` chỉ nên đưa **module chứa hàm** vào scope (như ví dụ trên) rồi
gọi qua `module::ham()` — nhìn vào lời gọi là biết ngay hàm không phải định nghĩa cục bộ. Với
**struct, enum, trait**, `use` nên đưa **thẳng đường dẫn đầy đủ** vào scope, vì phần lớn thời gian
sẽ nhắc tới tên kiểu đó nhiều lần và không có nhập nhằng nào cần làm rõ:

```rust
use std::collections::HashMap;

fn main() {
    let mut diem: HashMap<String, i32> = HashMap::new();
    diem.insert(String::from("An"), 10);
}
```

Khi hai kiểu trùng tên tới từ hai module khác nhau, đưa thẳng cả hai vào scope sẽ xung đột — hoặc
chỉ đưa module cha vào scope của từng cái (theo quy ước dùng cho hàm), hoặc dùng `as` để đặt bí
danh:

```rust
use std::fmt;
use std::io;

fn viet(_ket_qua: fmt::Result) {}
fn doc(_ket_qua: io::Result<()>) {}
```

```rust
use std::fmt::Result as FmtResult;
use std::io::Result as IoResult;

fn viet(_ket_qua: FmtResult) {}
fn doc(_ket_qua: IoResult<()>) {}
```

## pub use: re-export

`use` thường thì chỉ đưa tên vào scope nội bộ; thêm `pub` phía trước để code ở **ngoài** crate cũng
gọi được qua đường dẫn mới này, dù item gốc nằm sâu trong một module riêng tư của việc tổ chức nội
bộ:

```rust
mod nha_bep {
    pub fn nau_an() {
        println!("Đang nấu");
    }
}

pub use crate::nha_bep::nau_an;

fn main() {
    nau_an(); // gọi thẳng nhờ re-export, không cần đường dẫn qua nha_bep
}
```

`pub use` tách rời **cấu trúc thư mục/module nội bộ** khỏi **API nhìn thấy được từ bên ngoài** — có
thể tổ chức code nội bộ theo cách hợp lý với người viết, đồng thời export ra một cấu trúc phẳng, dễ
dùng hơn cho người gọi crate từ bên ngoài.

## Gộp nhiều use

Nhiều `use` cùng chung tiền tố gộp được bằng cú pháp lồng `{}`, hoặc dùng `*` để đưa **mọi** item
công khai của một module vào scope (thường chỉ hợp lý trong code test, ít dùng ở code thường vì làm
mất khả năng biết một tên tới từ đâu):

```rust
use std::{cmp::Ordering, io};

fn main() {
    let ket_qua = 1.cmp(&2);
    match ket_qua {
        Ordering::Less => println!("nhỏ hơn"),
        Ordering::Equal => println!("bằng"),
        Ordering::Greater => println!("lớn hơn"),
    }
    let _ = io::stdin();
}
```

## Tách module ra file riêng

Khi một module đủ lớn, tách nó khỏi file cha bằng cách khai báo `mod ten_module;` — dấu `;` thay cho
khối `{}` báo cho trình biên dịch: nội dung module nằm ở một file khác, tự đi tìm theo tên:

```rust,ignore
// src/lib.rs
mod nha_bep;
```

Trình biên dịch tìm nội dung ở `src/nha_bep.rs`. Nếu `nha_bep` còn chứa module con cũng cần tách
riêng (ví dụ `trang_tri`), chúng nằm trong một **thư mục** cùng tên module cha:

```text
src/
├── lib.rs
├── nha_bep.rs
└── nha_bep/
    └── trang_tri.rs
```

Trong `src/nha_bep.rs`, khai báo `pub mod trang_tri;` tương tự — trình biên dịch tiếp tục đi tìm ở
`src/nha_bep/trang_tri.rs`. Cây module (đường dẫn, quy tắc riêng tư) không hề thay đổi khi tách file
— tách file chỉ là chuyện tổ chức mã nguồn vật lý trên đĩa, hoàn toàn độc lập với cấu trúc module
logic đã khai báo bằng `mod`.
