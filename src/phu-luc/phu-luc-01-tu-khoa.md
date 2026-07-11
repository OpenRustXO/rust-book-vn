# Từ khoá

Danh sách từ khoá của Rust — không dùng được làm tên biến, hàm, kiểu, module, v.v. trừ trường hợp
raw identifier nói ở cuối phụ lục này.

## Đang dùng

| Từ khoá    | Ý nghĩa                                                                          |
| ---------- | -------------------------------------------------------------------------------- |
| `as`       | ép kiểu tường minh; đổi tên item khi `use`                                       |
| `async`    | đánh dấu một hàm/khối trả về `Future` thay vì chạy ngay                          |
| `await`    | tạm dừng chờ một `Future` tới khi `Ready`                                        |
| `break`    | thoát khỏi vòng lặp ngay lập tức                                                 |
| `const`    | khai báo hằng số, hoặc một tham số/hàm hằng                                      |
| `continue` | bỏ qua phần còn lại của lượt lặp hiện tại, sang lượt kế                          |
| `crate`    | tham chiếu tới module gốc của crate hiện tại                                     |
| `dyn`      | đánh dấu dynamic dispatch qua trait object (`dyn Trait`)                         |
| `else`     | nhánh thay thế cho `if`, `if let`, `let-else`                                    |
| `enum`     | định nghĩa một enum                                                              |
| `extern`   | khai báo liên kết với hàm/biến từ ngoài crate (FFI)                              |
| `false`    | giá trị `bool` sai                                                               |
| `fn`       | định nghĩa một hàm                                                               |
| `for`      | lặp qua các phần tử của một iterator; cũng dùng trong cú pháp trait cho lifetime |
| `if`       | rẽ nhánh dựa trên điều kiện `bool`                                               |
| `impl`     | implement chức năng cho một kiểu, hoặc một trait cho kiểu đó                     |
| `in`       | một phần cú pháp của `for`                                                       |
| `let`      | gán một giá trị cho biến                                                         |
| `loop`     | lặp vô hạn cho tới khi `break`                                                   |
| `match`    | so khớp một giá trị với các pattern                                              |
| `mod`      | định nghĩa một module                                                            |
| `move`     | ép closure lấy quyền sở hữu mọi giá trị nó capture                               |
| `mut`      | đánh dấu khả biến cho tham chiếu, con trỏ, biến                                  |
| `pub`      | đánh dấu công khai cho struct, method, module                                    |
| `ref`      | bind theo tham chiếu thay vì theo giá trị trong một pattern                      |
| `return`   | trả về giá trị, thoát khỏi hàm ngay                                              |
| `Self`     | bí danh cho kiểu đang định nghĩa/implement                                       |
| `self`     | chính đối tượng hoặc module hiện tại                                             |
| `static`   | biến toàn cục, hoặc lifetime kéo dài suốt chương trình                           |
| `struct`   | định nghĩa một struct                                                            |
| `super`    | module cha của module hiện tại                                                   |
| `trait`    | định nghĩa một trait                                                             |
| `true`     | giá trị `bool` đúng                                                              |
| `type`     | định nghĩa type alias, hoặc associated type                                      |
| `union`    | định nghĩa một union (chỉ có ý nghĩa trong ngữ cảnh `unsafe`)                    |
| `unsafe`   | đánh dấu code/hàm/trait bỏ qua một số kiểm tra an toàn của Rust                  |
| `use`      | đưa một đường dẫn (module, hàm, kiểu) vào scope                                  |
| `where`    | tách ràng buộc kiểu ra khỏi chữ ký, dễ đọc hơn khi ràng buộc dài                 |
| `while`    | lặp khi một điều kiện còn đúng                                                   |

## Dự trữ cho tương lai

Không dùng được dù hiện tại chưa có chức năng: `abstract`, `become`, `box`, `do`, `final`, `macro`,
`override`, `priv`, `try`, `typeof`, `unsized`, `virtual`, `yield`, `gen`.

## Raw identifier

Đặt tiền tố `r#` trước một từ vốn là từ khoá để vẫn dùng được nó làm tên định danh — chủ yếu cần khi
gọi code từ một edition Rust khác đã đưa thêm từ khoá mới (ví dụ tên biến `match` viết bằng
`r#match`), không phải cú pháp dùng thường xuyên trong code mới.
