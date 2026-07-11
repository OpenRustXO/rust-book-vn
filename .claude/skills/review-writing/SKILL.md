---
name: review-writing
description: Review an existing chapter/section of the Vietnamese Rust book for voice, structure, conciseness, terminology consistency, and whether its Rust code actually compiles — before it's considered done or ready to publish. Reports findings, does not rewrite wholesale. Use when asked to "review chapter X", "rà soát chương...", "proofread", "chuẩn bị publish".
---

# Rà soát một chương đã có

Dùng khi một chương/mục đã có nội dung và cần kiểm tra chất lượng trước khi coi là xong — khác với
`new-chapter` (viết mới) và `terminology-check` (chỉ tập trung thuật ngữ, quét toàn sách). Báo cáo
góp ý cụ thể theo file:dòng; không viết lại toàn bộ nội dung trừ khi tác giả yêu cầu.

## 1. Đọc lại bối cảnh

Đọc `README.md` (triết lý: học hiểu bản chất, rõ ràng có cấu trúc, đào sâu nhưng không lan man) và
`.claude/skills/terminology-check/glossary.md` trước khi đọc chương cần review.

## 2. Kiểm tra cấu trúc và văn phong

Với mỗi mục trong chương, xét:

- Có mở đầu ngắn nêu vấn đề/động lực trước khi vào cú pháp không, hay nhảy thẳng vào liệt kê?
- Giải thích có đi từ bản chất/cơ chế rồi mới tới ví dụ, hay chỉ liệt kê cú pháp?
- Có đoạn nào lan man, lặp ý đã nói, hoặc mượn khái niệm của chương sau chưa giới thiệu không?
- Ví dụ code có tối thiểu và đúng trọng tâm của mục, hay ôm đồm nhiều khái niệm cùng lúc?

## 3. Kiểm tra thuật ngữ

Đối chiếu thuật ngữ kỹ thuật dùng trong chương với `.claude/skills/terminology-check/glossary.md`.
Nếu gặp thuật ngữ chưa có trong glossary hoặc dùng khác với chương khác, dùng skill
`terminology-check` để xử lý thay vì tự sửa tại chỗ.

## 4. Kiểm tra kỹ thuật và định dạng

````bash
mdbook test    # mọi khối ```rust phải biên dịch được, trừ khi cố ý đánh dấu ignore/does_not_compile
mdbook build   # bắt lỗi link nội bộ hỏng
dprint check   # xác nhận đã format đúng dprint.json, không tự sửa
````

## 5. Báo cáo

Trả về danh sách góp ý, mỗi ý gắn `file:dòng`, xếp theo mức độ quan trọng (lỗi kỹ thuật/code không
chạy trước, rồi tới cấu trúc/văn phong, rồi tới góp ý nhỏ về câu chữ). Không tự ý sửa hàng loạt câu
chữ chủ quan — chỉ sửa khi tác giả xác nhận hoặc khi là lỗi rõ ràng (code không compile, link hỏng,
thuật ngữ lệch khỏi glossary đã chốt).
