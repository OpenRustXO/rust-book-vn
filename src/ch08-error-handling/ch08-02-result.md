# Result

Lỗi có thể phục hồi được biểu diễn bằng enum có sẵn trong thư viện chuẩn:

```rust,ignore
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

`T` là kiểu giá trị khi thành công, `E` là kiểu lỗi khi thất bại — cả hai đều là generic, mỗi hàm
trả `Result` tự quyết định `T`/`E` cụ thể của mình. `File::open` là ví dụ quen thuộc: trả về
`Result<File, std::io::Error>`.

```rust,no_run
use std::fs::File;

fn main() {
    let ket_qua = File::open("hello.txt");

    let _file = match ket_qua {
        Ok(file) => file,
        Err(loi) => panic!("Không mở được file: {loi:?}"),
    };
}
```

## Xử lý khác nhau theo loại lỗi

`match` lồng nhau để phân biệt loại lỗi cụ thể — ví dụ tự tạo file nếu lỗi là "không tìm thấy",
nhưng vẫn panic với mọi lỗi khác (không đủ quyền, đĩa hỏng, ...):

```rust,no_run
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let ket_qua = File::open("hello.txt");

    let _file = match ket_qua {
        Ok(file) => file,
        Err(loi) => match loi.kind() {
            ErrorKind::NotFound => match File::create("hello.txt") {
                Ok(fc) => fc,
                Err(loi_tao) => panic!("Không tạo được file: {loi_tao:?}"),
            },
            khac => panic!("Không mở được file: {khac:?}"),
        },
    };
}
```

## unwrap và expect: rút gọn cho trường hợp không cần phân biệt lỗi

`unwrap()` trả thẳng giá trị `T` nếu `Ok`, panic với thông báo mặc định nếu `Err`. `expect(msg)` làm
y hệt nhưng cho tự viết thông báo panic — nên dùng `expect` thay `unwrap` trong code thật, vì thông
báo tự viết giúp định vị đúng chỗ gây lỗi nhanh hơn nhiều so với thông báo mặc định chung chung:

```rust,should_panic
use std::fs::File;

fn main() {
    let _file =
        File::open("chac_chan_khong_ton_tai.txt").expect("file cấu hình phải tồn tại sẵn");
}
```

## Lan truyền lỗi ra ngoài

Một hàm gặp lỗi không nhất thiết phải tự xử lý ngay — trả `Err` ra ngoài để hàm gọi nó quyết định là
lựa chọn hợp lý, đặc biệt khi hàm gọi có nhiều ngữ cảnh hơn để biết nên làm gì tiếp:

```rust
use std::fs::File;
use std::io::{self, Read};

fn doc_ten_tu_file() -> Result<String, io::Error> {
    let ket_qua_mo = File::open("ten.txt");

    let mut file = match ket_qua_mo {
        Ok(file) => file,
        Err(loi) => return Err(loi),
    };

    let mut ten = String::new();

    match file.read_to_string(&mut ten) {
        Ok(_) => Ok(ten),
        Err(loi) => Err(loi),
    }
}

fn main() {
    match doc_ten_tu_file() {
        Ok(ten) => println!("Tên: {ten}"),
        Err(loi) => println!("Lỗi: {loi}"),
    }
}
```

### Toán tử ?

Mẫu "match rồi return Err sớm nếu lỗi, dùng tiếp giá trị nếu Ok" lặp lại đủ nhiều để có hẳn một toán
tử riêng. `?` đặt sau một expression kiểu `Result`: nếu `Ok(gia_tri)`, biểu thức trả về `gia_tri` để
dùng tiếp; nếu `Err(loi)`, **return ngay** `Err(loi)` khỏi hàm hiện tại — viết lại hàm trên:

```rust
use std::fs::File;
use std::io::{self, Read};

fn doc_ten_tu_file() -> Result<String, io::Error> {
    let mut file = File::open("ten.txt")?;
    let mut ten = String::new();
    file.read_to_string(&mut ten)?;
    Ok(ten)
}

fn main() {
    match doc_ten_tu_file() {
        Ok(ten) => println!("Tên: {ten}"),
        Err(loi) => println!("Lỗi: {loi}"),
    }
}
```

`?` chỉ dùng được trong hàm có kiểu trả về tương thích (`Result`, `Option`, hoặc kiểu khác implement
`FromResidual`) — không thể rải `?` ở giữa một hàm trả `()` chỉ vì tiện. Khi kiểu lỗi của expression
khác kiểu lỗi hàm đang trả về, `?` còn tự gọi `From::from` để chuyển đổi — đây là lý do rất nhiều
hàm trong code Rust thực tế trả về `Result<T, Box<dyn std::error::Error>>`: `Box<dyn Error>` nhận
được chuyển đổi từ hầu hết mọi kiểu lỗi cụ thể, cho phép dùng `?` với nhiều nguồn lỗi khác nhau
trong cùng một hàm mà không cần tự viết converter cho từng loại.

`?` cũng dùng được với `Option<T>`: `None` khiến hàm return `None` ngay, `Some(gia_tri)` cho ra
`gia_tri` để dùng tiếp — hữu ích khi cần lan truyền "không tìm thấy" qua nhiều bước tra cứu lồng
nhau.
