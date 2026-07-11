# Toán tử và ký hiệu

## Toán tử

Phần lớn toán tử có thể nạp chồng (overload) thông qua trait tương ứng trong `std::ops` — implement
trait đó cho kiểu của mình để toán tử hoạt động được trên kiểu đó.

| Toán tử                  | Ý nghĩa                            | Trait tương ứng (nếu overload được) |
| ------------------------ | ---------------------------------- | ----------------------------------- |
| `+`                      | cộng                               | `Add`                               |
| `-`                      | trừ; phủ định một ngôi             | `Sub`, `Neg`                        |
| `*`                      | nhân; giải tham chiếu              | `Mul`, `Deref`                      |
| `/`                      | chia                               | `Div`                               |
| `%`                      | chia lấy dư                        | `Rem`                               |
| `!`                      | phủ định logic; kiểu never         | `Not`                               |
| `&`                      | tham chiếu bất biến; AND theo bit  | `BitAnd`                            |
| `&&`                     | AND logic (ngắn mạch)              |                                     |
| `&mut`                   | tham chiếu khả biến                |                                     |
| `\|`                     | OR theo bit                        | `BitOr`                             |
| `\|\|`                   | OR logic (ngắn mạch)               |                                     |
| `^`                      | XOR theo bit                       | `BitXor`                            |
| `<<`                     | dịch trái theo bit                 | `Shl`                               |
| `>>`                     | dịch phải theo bit                 | `Shr`                               |
| `==`                     | so sánh bằng                       | `PartialEq`                         |
| `!=`                     | so sánh khác                       | `PartialEq`                         |
| `<` `>` `<=` `>=`        | so sánh thứ tự                     | `PartialOrd`                        |
| `=`                      | gán giá trị                        |                                     |
| `+=` `-=` `*=` `/=` `%=` | gán kèm phép toán                  | `AddAssign`, `SubAssign`, ...       |
| `?`                      | lan truyền lỗi/`None` sớm khỏi hàm |                                     |
| `..` `..=`               | tạo một range (nửa mở/đóng)        |                                     |
| `@`                      | bind giá trị trong pattern         |                                     |

## Ký hiệu khác

| Ký hiệu       | Ý nghĩa                                                                |
| ------------- | ---------------------------------------------------------------------- |
| `::`          | đường dẫn tới item trong một module/kiểu (`std::collections::HashMap`) |
| `->`          | kiểu trả về của hàm/closure                                            |
| `=>`          | phân tách pattern và code trong một nhánh `match`                      |
| `'a`          | tham số lifetime (bắt đầu bằng dấu `'`)                                |
| `#[...]`      | attribute gắn lên item ngay bên dưới (`#[derive(Debug)]`, `#[test]`)   |
| `//` `/* */`  | comment một dòng / nhiều dòng                                          |
| `///` `//!`   | doc comment cho item bên dưới / cho item chứa nó                       |
| `_`           | pattern bỏ qua giá trị; ký tự phân cách trong số (`1_000_000`)         |
| `!` (sau tên) | gọi một macro (`println!`, `vec!`)                                     |
| `dyn Trait`   | trait object, dynamic dispatch                                         |
| `impl Trait`  | một kiểu cụ thể nào đó implement `Trait`, không tiết lộ kiểu chính xác |
