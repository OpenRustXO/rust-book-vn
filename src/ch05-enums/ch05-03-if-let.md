# if let và let else

`match` đầy đủ nhưng dài dòng khi chỉ thật sự quan tâm **một** pattern và muốn bỏ qua mọi trường hợp
khác. `if let` viết gọn đúng tình huống đó:

```rust
fn main() {
    let so_toi_da: Option<u8> = Some(3);

    if let Some(max) = so_toi_da {
        println!("Số tối đa: {max}");
    }
}
```

Đoạn trên tương đương một `match` với nhánh `Some(max) => println!(...)` và nhánh `_ => ()` bị bỏ
qua — nhưng đổi lại việc rút gọn này, `if let` **mất đi kiểm tra đầy đủ** mà `match` đảm bảo: thêm
một variant mới vào enum sẽ không khiến `if let` báo lỗi biên dịch, vì bản chất nó chưa từng buộc
phải xử lý hết mọi khả năng. Chọn `if let` là đánh đổi có ý thức giữa ngắn gọn và đầy đủ, không phải
lựa chọn mặc định.

Kèm `else` để xử lý trường hợp không khớp, tương đương nhánh `_` của `match`:

```rust
enum DongTien {
    HaiNamXu(String), // tên bang được in trên đồng xu
    Khac,
}

fn main() {
    let xu = DongTien::HaiNamXu(String::from("Texas"));
    let mut dem = 0;

    if let DongTien::HaiNamXu(bang) = xu {
        println!("Đồng 25 xu từ bang {bang}!");
    } else {
        dem += 1;
    }

    println!("{dem}");
}
```

## let-else

Một tình huống rất thường gặp: muốn lấy giá trị ra khỏi một pattern, và nếu không khớp thì **thoát
sớm** khỏi hàm hiện tại (`return`, `break`, hoặc `panic!`) — không phải xử lý gì thêm trong nhánh
khớp mà tiếp tục dùng biến đó ở phần code phía sau, cùng cấp với `if let` chứ không bị lồng vào
trong:

```rust
fn mo_ta(so: Option<u8>) -> String {
    let Some(gia_tri) = so else {
        return String::from("không có giá trị");
    };

    format!("giá trị là {gia_tri}")
}

fn main() {
    println!("{}", mo_ta(Some(5)));
    println!("{}", mo_ta(None));
}
```

Nếu viết bằng `if let` thông thường, phần code dùng `gia_tri` sẽ phải nằm lồng trong khối `if let`
(vì `gia_tri` chỉ tồn tại trong scope của nhánh khớp), khiến hàm dài thêm một cấp thụt lề dù logic
chính chỉ có một luồng. `let-else` đảo ngược cấu trúc: nhánh **không khớp** (`else`) mới là nhánh
bắt buộc phải rẽ nhánh khỏi luồng chính (`return`/`break`/`panic!`/`continue` — bất kỳ thứ gì khiến
hàm không "rơi xuống" tiếp), còn biến bind được ở nhánh khớp (`gia_tri`) tồn tại ngay trong scope
bao quanh, dùng tiếp bình thường mà không cần lồng thêm khối nào.
