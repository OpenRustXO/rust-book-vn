---
name: terminology-check
description: Check and maintain consistent Vietnamese renderings of Rust technical terms (ownership, borrowing, trait, lifetime, ...) across src/**/*.md in this book, backed by glossary.md. Use when adding a technical term for the first time, after writing/editing a chapter, or when asked to "check terminology"/"kiểm tra thuật ngữ"/"rà soát thuật ngữ".
---

# Rà soát thuật ngữ

Sách kỹ thuật viết trong nhiều phiên làm việc rất dễ bị dịch một thuật ngữ theo hai cách khác nhau ở
hai chương — skill này giữ cho `glossary.md` là nguồn sự thật duy nhất và toàn bộ `src/` khớp với
nó.

## 1. Đọc glossary hiện tại

`.claude/skills/terminology-check/glossary.md` liệt kê quyết định đã chốt cho từng thuật ngữ, cùng
quy ước chung (khi nào giữ nguyên tiếng Anh, khi nào dịch nghĩa).

## 2. Với thuật ngữ mới chưa có trong glossary

1. Xem quy ước chung ở đầu `glossary.md` để đoán hướng xử lý phù hợp (tên kiểu/API giữ nguyên, khái
   niệm cốt lõi thường dịch nghĩa kèm tiếng Anh trong ngoặc lần đầu xuất hiện, v.v.).
2. Nếu thuật ngữ chỉ là biến thể của một thuật ngữ đã có (`borrow` khi đã có `borrowing`), suy ra
   cách dùng nhất quán chứ không tự bịa cách mới.
3. Nếu là thuật ngữ cốt lõi hoàn toàn mới và quy ước chung không đủ rõ để suy ra — hỏi tác giả cách
   họ muốn dịch/giữ, đừng tự quyết định một mình rồi lan ra nhiều chương.
4. Thêm một dòng mới vào bảng trong `glossary.md` ngay khi có quyết định.

## 3. Rà soát toàn bộ sách tìm chỗ lệch

```bash
# Liệt kê các thuật ngữ tiếng Anh xuất hiện trong nội dung (loại trừ khối code) để đối chiếu
# với glossary — chạy trong thư mục gốc repo, thay THUẬT_NGỮ bằng từ cần kiểm tra:
grep -rn --include='*.md' -i 'THUẬT_NGỮ' src/
```

Với mỗi thuật ngữ đã có trong glossary, tìm mọi chỗ nó xuất hiện trong `src/**/*.md` (bỏ qua nội
dung bên trong khối `` ```rust ``, vì đó là code chứ không phải văn xuôi) và so với cách dùng đã
chốt. Báo cáo danh sách file:dòng bị lệch — **không tự động sửa hàng loạt** nếu chưa rõ tác giả muốn
giữ cách dùng nào làm chuẩn, vì phần văn xuôi quanh đó có thể cần viết lại theo hướng khác nhau tuỳ
từng cách dịch.

## 4. Cập nhật glossary

Sau khi có quyết định (thuật ngữ mới hoặc thống nhất lại một thuật ngữ cũ), sửa `glossary.md` ngay —
đây là bước dễ quên nhất và là lý do glossary hay lệch khỏi thực tế bài viết.
