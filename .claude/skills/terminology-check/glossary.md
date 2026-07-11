# Bảng thuật ngữ (nội bộ)

File này **không phải nội dung sách** — là nguồn tham chiếu để giữ cách dịch/giữ nguyên thuật ngữ kỹ
thuật nhất quán xuyên suốt `src/`. Dùng bởi skill `terminology-check`, cập nhật ngay khi có thuật
ngữ mới hoặc quyết định lại một thuật ngữ cũ. Xem cách dùng ở
`.claude/skills/terminology-check/SKILL.md`.

## Quy ước chung

- **Giữ nguyên tiếng Anh**: tên kiểu và API của thư viện chuẩn (`String`, `Vec`, `Option`, `Result`,
  `HashMap`, ...), từ khoá ngôn ngữ (`struct`, `enum`, `trait`, `impl`, `match`, `mod`, `crate`,
  `async`/`await`, ...) — đây là identifier thật trong code, dịch ra sẽ gây khó tra cứu.
- **Thuật ngữ cốt lõi** (ownership, borrowing, lifetime, generic, closure, iterator, ...): dịch
  nghĩa hay giữ nguyên tuỳ từng từ, quyết định theo từng dòng trong bảng bên dưới. Khi một thuật ngữ
  cốt lõi được dùng lần đầu trong sách, chú thích tiếng Anh trong ngoặc ngay lần xuất hiện đầu tiên
  của chương đó, ví dụ: "quyền sở hữu (ownership)".
- Thuật ngữ đánh dấu **"chưa quyết định"** nghĩa là chưa chốt — bàn với tác giả trước khi dùng lần
  đầu trong một chương, đừng tự chọn rồi để nó lan ra nhiều nơi.

## Bảng thuật ngữ

| Thuật ngữ (EN) | Cách dùng trong sách | Ghi chú                                                                        |
| -------------- | -------------------- | ------------------------------------------------------------------------------ |
| variable       | biến                 |                                                                                |
| function       | hàm                  |                                                                                |
| data type      | kiểu dữ liệu         |                                                                                |
| compiler       | trình biên dịch      |                                                                                |
| compile        | biên dịch            |                                                                                |
| crate          | giữ nguyên `crate`   | theo quy ước chung — dùng ngay ở ch01-01 khi giới thiệu Cargo/cargo-binstall   |
| package        | _chưa quyết định_    | phân biệt với crate — chưa cần dùng tới, quyết định khi thực sự phải phân biệt |
| ownership      | _chưa quyết định_    | thuật ngữ cốt lõi, dùng xuyên suốt chương Ownership                            |
| borrowing      | _chưa quyết định_    | liên quan chặt tới ownership, quyết định cùng lúc                              |
| lifetime       | _chưa quyết định_    |                                                                                |
| trait          | giữ nguyên `trait`   | theo quy ước giữ nguyên từ khoá ngôn ngữ                                       |
| struct         | giữ nguyên `struct`  | theo quy ước giữ nguyên từ khoá ngôn ngữ                                       |
| enum           | giữ nguyên `enum`    | theo quy ước giữ nguyên từ khoá ngôn ngữ                                       |
