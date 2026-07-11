# Bảng thuật ngữ (nội bộ)

File này **không phải nội dung sách** — là nguồn tham chiếu để giữ cách dịch/giữ nguyên thuật ngữ kỹ
thuật nhất quán xuyên suốt `src/`. Dùng bởi skill `terminology-check`, cập nhật ngay khi có thuật
ngữ mới hoặc quyết định lại một thuật ngữ cũ. Xem cách dùng ở
`.claude/skills/terminology-check/SKILL.md`.

## Quy ước chung

- **Giữ nguyên tiếng Anh**: tên kiểu và API của thư viện chuẩn (`String`, `Vec`, `Option`, `Result`,
  `HashMap`, `Future`, ...), từ khoá ngôn ngữ (`struct`, `enum`, `trait`, `impl`, `match`, `mod`,
  `crate`, `async`/`await`, `unsafe`, ...) — đây là identifier thật trong code, dịch ra sẽ gây khó
  tra cứu. Áp dụng tương tự cho các thuật ngữ đặc trưng của Rust không có bản dịch tự nhiên
  (`generic`, `closure`, `iterator`, `slice`, `macro`, `shadowing`) — giữ nguyên tốt hơn là ép dịch
  gượng gạo.
- **Thuật ngữ cốt lõi** (ownership, borrowing, lifetime, ...): dịch nghĩa hay giữ nguyên tuỳ từng
  từ, quyết định theo từng dòng trong bảng bên dưới. Khi một thuật ngữ cốt lõi được dùng lần đầu
  trong sách, chú thích tiếng Anh trong ngoặc ngay lần xuất hiện đầu tiên của chương đó, ví dụ:
  "quyền sở hữu (ownership)".
- Các quyết định đánh dấu **"chốt theo /goal"** là do tự quyết định khi viết toàn bộ sách theo chỉ
  đạo tự chọn chủ đề/thuật ngữ của tác giả (không dừng lại hỏi từng từ) — tác giả nên rà lại một
  lượt sau khi có bản thảo, đổi bằng cách sửa dòng tương ứng rồi chạy skill `terminology-check` để
  lan sửa đổi ra toàn sách.

## Bảng thuật ngữ

| Thuật ngữ (EN) | Cách dùng trong sách   | Ghi chú                                                                                                                                                                                        |
| -------------- | ---------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| variable       | biến                   |                                                                                                                                                                                                |
| function       | hàm                    |                                                                                                                                                                                                |
| data type      | kiểu dữ liệu           |                                                                                                                                                                                                |
| compiler       | trình biên dịch        |                                                                                                                                                                                                |
| compile        | biên dịch              |                                                                                                                                                                                                |
| scope          | phạm vi                | chốt theo /goal                                                                                                                                                                                |
| mutable        | khả biến               | chốt theo /goal — đối lập với bất biến                                                                                                                                                         |
| immutable      | bất biến               | chốt theo /goal                                                                                                                                                                                |
| shadowing      | giữ nguyên `shadowing` | chốt theo /goal — hành vi đặc trưng của Rust, không ép dịch                                                                                                                                    |
| stack (bộ nhớ) | giữ nguyên `stack`     | chốt theo /goal — tránh nhầm với cấu trúc dữ liệu "ngăn xếp"                                                                                                                                   |
| heap (bộ nhớ)  | giữ nguyên `heap`      | chốt theo /goal — tránh nhầm với cấu trúc dữ liệu "đống"                                                                                                                                       |
| crate          | giữ nguyên `crate`     | theo quy ước chung                                                                                                                                                                             |
| package        | giữ nguyên `package`   | sửa lại sau rà soát — bản nháp ban đầu chốt "gói" nhưng khi viết thực tế (ch01, ch06, ch12) đã dùng nhất quán "package" (khớp tên section `[package]` trong Cargo.toml), giữ nguyên hợp lý hơn |
| module         | giữ nguyên `module`    | sửa lại sau rà soát — bản nháp ban đầu chốt "mô-đun" nhưng khi viết thực tế (ch06 trở đi) đã dùng nhất quán "module", khớp quy ước chung hơn (giống crate/generic/closure)                     |
| ownership      | quyền sở hữu           | chốt theo /goal — thuật ngữ cốt lõi, dùng xuyên suốt chương 3                                                                                                                                  |
| borrow(ing)    | mượn / việc mượn       | chốt theo /goal — đi cùng ownership                                                                                                                                                            |
| reference      | tham chiếu             | chốt theo /goal                                                                                                                                                                                |
| dangling       | treo                   | chốt theo /goal — "tham chiếu treo", "con trỏ treo"                                                                                                                                            |
| lifetime       | thời hạn sống          | chốt theo /goal — tham số `'a` gọi là "tham số thời hạn sống"                                                                                                                                  |
| slice          | giữ nguyên `slice`     | chốt theo /goal — coi như tên kiểu, giống `&str`/`&[T]`                                                                                                                                        |
| generic        | giữ nguyên `generic`   | chốt theo /goal                                                                                                                                                                                |
| trait bound    | ràng buộc trait        | chốt theo /goal                                                                                                                                                                                |
| closure        | giữ nguyên `closure`   | chốt theo /goal                                                                                                                                                                                |
| iterator       | giữ nguyên `iterator`  | chốt theo /goal — trait `Iterator` trong thư viện chuẩn                                                                                                                                        |
| pattern        | mẫu                    | chốt theo /goal — "pattern matching" = "khớp mẫu"                                                                                                                                              |
| smart pointer  | con trỏ thông minh     | chốt theo /goal                                                                                                                                                                                |
| concurrency    | đồng thời              | chốt theo /goal — "lập trình đồng thời"                                                                                                                                                        |
| thread         | luồng                  | chốt theo /goal                                                                                                                                                                                |
| macro          | giữ nguyên `macro`     | chốt theo /goal                                                                                                                                                                                |
| trait          | giữ nguyên `trait`     | theo quy ước giữ nguyên từ khoá ngôn ngữ                                                                                                                                                       |
| struct         | giữ nguyên `struct`    | theo quy ước giữ nguyên từ khoá ngôn ngữ                                                                                                                                                       |
| enum           | giữ nguyên `enum`      | theo quy ước giữ nguyên từ khoá ngôn ngữ                                                                                                                                                       |
