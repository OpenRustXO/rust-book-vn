# Cú pháp pattern

## Khớp literal, nhiều pattern, khoảng giá trị

```rust
fn main() {
    let x = 5;

    match x {
        1 => println!("một"),
        2 | 3 => println!("hai hoặc ba"), // | nối nhiều pattern
        4..=6 => println!("từ bốn đến sáu"), // khoảng đóng cả hai đầu
        _ => println!("giá trị khác"),
    }
}
```

`..=` cũng dùng được với `char`:

```rust
fn main() {
    let c = 'f';

    match c {
        'a'..='j' => println!("chữ cái đầu bảng chữ cái"),
        'k'..='z' => println!("chữ cái cuối bảng chữ cái"),
        _ => println!("không phải chữ thường"),
    }
}
```

## Destructure struct và enum

```rust
struct Diem {
    x: i32,
    y: i32,
}

fn main() {
    let p = Diem { x: 0, y: 7 };

    let Diem { x, y } = p;
    println!("{x} {y}");

    match p {
        Diem { x: 0, y } => println!("Trên trục tung, y = {y}"),
        Diem { x, y: 0 } => println!("Trên trục hoành, x = {x}"),
        Diem { x, y } => println!("Không nằm trên trục nào: ({x}, {y})"),
    }
}
```

Destructure lồng nhau qua enum chứa struct:

```rust
enum Mau {
    Rgb(i32, i32, i32),
}

enum ThongDiep {
    DoiMau(Mau),
    ImLang,
}

fn main() {
    let td = ThongDiep::DoiMau(Mau::Rgb(0, 160, 255));

    match td {
        ThongDiep::DoiMau(Mau::Rgb(r, g, b)) => {
            println!("Đổi màu RGB: {r}, {g}, {b}");
        }
        ThongDiep::ImLang => println!("Im lặng"),
    }
}
```

## Bỏ qua giá trị

`_` bỏ qua toàn bộ một giá trị mà không bind vào biến nào — khác một biến bắt đầu bằng `_` (như
`_x`), vẫn bind giá trị vào biến đó (chỉ tắt cảnh báo "biến không dùng"), nên **vẫn giữ quyền sở hữu
giá trị cho tới hết scope**. `_` trần không bind gì cả, giá trị bị drop **ngay tại chỗ** nếu nó
không implement `Copy`:

```rust
fn main() {
    let s = Some(String::from("dữ liệu"));

    if let Some(_) = s {
        println!("có giá trị");
    }
    // s đã bị move vào nhánh match rồi drop ngay khi nhánh kết thúc,
    // vì `_` không giữ lại quyền sở hữu ở biến nào cả — nhưng vẫn không dùng lại được s ở đây
    // do giá trị bên trong Some đã bị lấy ra để khớp.
}
```

`..` bỏ qua **phần còn lại** của một struct hoặc tuple, không cần liệt kê hết field không quan tâm:

```rust
struct Diem3D {
    x: i32,
    y: i32,
    z: i32,
}

fn main() {
    let p = Diem3D { x: 1, y: 2, z: 3 };

    let Diem3D { x, .. } = p;
    println!("chỉ cần x = {x}");

    let so = (1, 2, 3, 4, 5);
    match so {
        (dau, .., cuoi) => println!("đầu = {dau}, cuối = {cuoi}"),
    }
}
```

## Match guard

Thêm điều kiện `if` sau một pattern — chỉ khớp nhánh đó nếu **cả** pattern khớp **lẫn** điều kiện
đúng. Match guard giải quyết được tình huống pattern tự nó không diễn tả nổi, đặc biệt khi cần so
sánh với một biến từ scope **ngoài** `match` (pattern thường tạo biến mới, che khuất biến ngoài cùng
tên — match guard vẫn thấy được biến ngoài vì nó là một expression bình thường, không phải cú pháp
pattern):

```rust
fn main() {
    let x = Some(5);
    let y = 10;

    match x {
        Some(gia_tri) if gia_tri == y => println!("bằng y"),
        Some(gia_tri) => println!("Some với giá trị {gia_tri}, y ở ngoài là {y}"),
        None => println!("không có gì"),
    }
}
```

## @ binding

`@` vừa bind giá trị vào một biến, vừa kiểm tra giá trị đó khớp một pattern khác — cần khi vừa muốn
**dùng** giá trị vừa muốn **ràng buộc phạm vi** nó trong cùng một nhánh:

```rust
fn main() {
    let id = 5;

    match id {
        id_hop_le @ 3..=7 => {
            println!("ID {id_hop_le} nằm trong khoảng hợp lệ")
        }
        10..=12 => println!("ID nằm trong khoảng đặc biệt, không lấy ra biến"),
        _ => println!("ID khác"),
    }
}
```

Không có `@`, nhánh `3..=7 => ...` vẫn kiểm tra được khoảng, nhưng không có cách nào lấy ra giá trị
cụ thể đã khớp (chỉ có thể dùng lại biến gốc `id` từ ngoài, không phải lúc nào cũng tiện hoặc rõ
ràng bằng cách đặt tên riêng ngay tại chỗ khớp).
