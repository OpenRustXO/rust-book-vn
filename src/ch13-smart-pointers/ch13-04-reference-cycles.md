# Chu trình tham chiếu

`Rc<T>` tự dọn dẹp dữ liệu khi bộ đếm về 0 — nhưng nếu hai giá trị `Rc` trỏ vòng lại chính nhau, bộ
đếm của cả hai **không bao giờ** về 0, dữ liệu rò rỉ vĩnh viễn dù không còn ai truy cập được tới nó
nữa. Đây không phải lỗi vi phạm an toàn bộ nhớ (không có use-after-free hay double-free) nên borrow
checker **không** ngăn được — chỉ là một lỗi logic, và Rust không hứa hẹn ngăn mọi rò rỉ bộ nhớ, chỉ
ngăn các lỗi bộ nhớ không an toàn.

```rust
use std::cell::RefCell;
use std::rc::Rc;

#[derive(Debug)]
struct Node {
    tiep_theo: RefCell<Option<Rc<Node>>>,
}

fn main() {
    let a = Rc::new(Node {
        tiep_theo: RefCell::new(None),
    });

    let b = Rc::new(Node {
        tiep_theo: RefCell::new(Some(Rc::clone(&a))),
    });

    // Tạo chu trình: a trỏ tới b, b đã trỏ tới a từ trước
    *a.tiep_theo.borrow_mut() = Some(Rc::clone(&b));

    println!(
        "đếm a = {}, đếm b = {}",
        Rc::strong_count(&a),
        Rc::strong_count(&b)
    );
}
```

Sau đoạn trên, cả `a` và `b` đều có `strong_count` là 2 (một từ biến cục bộ, một từ tham chiếu vòng
của phía kia) — không bao giờ giảm về 0 dù `a`, `b` đã ra khỏi scope, vì mỗi bên vẫn giữ một `Rc`
trỏ tới bên còn lại.

## Weak\<T\>: tham chiếu không sở hữu

`Rc::downgrade(&a)` tạo một `Weak<T>` — tham chiếu **không** tính vào `strong_count`, nên không ngăn
dữ liệu bị drop. Vì dữ liệu có thể đã bị drop mất bất cứ lúc nào, `Weak<T>` không cho truy cập trực
tiếp — phải gọi `.upgrade()`, trả về `Option<Rc<T>>`: `Some` nếu dữ liệu vẫn còn, `None` nếu đã bị
drop.

Ứng dụng điển hình: quan hệ cha-con, nơi cha sở hữu con nhưng con chỉ cần **tham chiếu ngược** tới
cha, không sở hữu:

```rust
use std::cell::RefCell;
use std::rc::{Rc, Weak};

struct Node {
    gia_tri: i32,
    cha: RefCell<Weak<Node>>,
    con: RefCell<Vec<Rc<Node>>>,
}

fn main() {
    let la = Rc::new(Node {
        gia_tri: 3,
        cha: RefCell::new(Weak::new()),
        con: RefCell::new(vec![]),
    });

    let goc = Rc::new(Node {
        gia_tri: 5,
        cha: RefCell::new(Weak::new()),
        con: RefCell::new(vec![Rc::clone(&la)]),
    });

    *la.cha.borrow_mut() = Rc::downgrade(&goc);

    match la.cha.borrow().upgrade() {
        Some(cha) => println!("Giá trị của cha: {}", cha.gia_tri),
        None => println!("Không còn cha"),
    }
}
```

`goc` sở hữu `la` qua `Rc` trong `con` (quan hệ sở hữu thật: cha giữ con sống); `la` chỉ giữ `Weak`
tới `goc` trong `cha` — không tạo thành chu trình sở hữu, vì `Weak` không góp phần giữ `goc` sống.
Chọn hướng nào là `Rc` (sở hữu thật), hướng nào là `Weak` (chỉ tham chiếu) tuỳ ngữ nghĩa dữ liệu:
hướng "cha sở hữu con" hợp lý về mặt vòng đời (con không nên tồn tại nếu cha đã mất), hướng ngược
lại chỉ cần biết "cha là ai" mà không ảnh hưởng gì tới việc cha có tồn tại hay không.
