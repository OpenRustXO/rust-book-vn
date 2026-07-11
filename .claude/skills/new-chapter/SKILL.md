---
name: new-chapter
description: Scaffold and draft a new chapter or section for the Vietnamese Rust book (rust-book-vn) — creates the file under the correct src/chNN-slug/ directory, registers it in src/SUMMARY.md with correct nesting, drafts content in the book's own voice (not a translation), checks Rust code blocks with mdbook test, and formats with dprint. Use when asked to write/add a new chapter or section — "viết chương mới", "thêm mục về...", "write chapter X".
---

# Tạo chương/mục mới

Dùng khi cần thêm một chương lớn hoặc một mục con mới vào sách. Đọc `CLAUDE.md` ở gốc repo trước nếu
chưa nắm quy ước cấu trúc/định dạng của dự án.

## 1. Xác định vị trí trong mục lục

Đọc `src/SUMMARY.md` để biết mục mới là:

- **Chương lớn mới** → tạo thư mục `src/chNN-slug/` (NN = số thứ tự 2 chữ số tiếp theo, slug ngắn
  gọn không dấu, ví dụ `ch02-guessing-game`), file đầu tiên bên trong là `chNN-01-slug.md`.
- **Mục con của chương đã có** → thêm file `chNN-MM-slug.md` vào đúng thư mục `chNN-slug/` đã tồn
  tại, MM là số thứ tự tiếp theo trong chương đó.

Lưu ý: project này **không dùng file chNN-00 tổng quan chương** như sách gốc tiếng Anh — mục con đầu
tiên đã là `chNN-01`. Theo đúng ví dụ hiện có (`ch01-install/ch01-01-getting-started.md`), tên thư
mục dùng slug mô tả ngắn (`ch01-install`), không nhất thiết trùng slug với file bên trong.

## 2. Cập nhật `src/SUMMARY.md`

Thêm dòng link đúng cấp lồng — chương lớn ở cấp 0 (`- [Tên chương](chNN-slug/chNN-01-slug.md)`), mục
con thụt vào 4 space bên dưới chương cha:

```markdown
- [Tên chương](chNN-slug/chNN-01-slug.md)
  - [Tên mục con](chNN-slug/chNN-02-slug.md)
```

Tên hiển thị trong `SUMMARY.md` là tiếng Việt — đây là tiêu đề người đọc thấy trong mục lục, phải
khớp với heading H1 trong file. mdBook chỉ build những file có mặt ở đây; quên khai báo là file sẽ
không xuất hiện trong sách dù đã tạo.

## 3. Kiểm tra glossary trước khi viết

Nếu chương sắp viết dùng thuật ngữ Rust chưa từng xuất hiện trong sách (ownership, borrowing, trait,
lifetime, generic, closure, v.v.), đọc `.claude/skills/terminology-check/glossary.md` trước. Nếu
thuật ngữ chưa có quyết định, dùng skill `terminology-check` hoặc hỏi thẳng tác giả cách dịch/giữ
nguyên trước khi viết cả chương với một lựa chọn tuỳ tiện — sửa lại sau khi đã viết nhiều thường tốn
công hơn quyết định trước.

## 4. Soạn nội dung

**Không dịch nguyên văn** [rust-lang/book](https://github.com/rust-lang/book) — đối chiếu để đảm bảo
đúng kỹ thuật, nhưng viết lại bằng giọng văn và ví dụ riêng, đúng tinh thần "học hiểu bản chất, đào
sâu nhưng không lan man" trong `README.md`. Cấu trúc gợi ý cho một mục:

1. Mở đầu ngắn: vấn đề/động lực khiến tính năng này tồn tại, trước khi đi vào cú pháp.
2. Giải thích bản chất/cơ chế trước, ví dụ minh hoạ sau — không liệt kê cú pháp suông.
3. Ví dụ code Rust tối thiểu, đúng trọng tâm của mục (không mượn khái niệm của chương sau).
4. Không cần phần "Tóm tắt" nếu mục đã ngắn — chỉ thêm khi thực sự giúp chốt lại ý.

**Không nhắc tên hay so sánh với bất kỳ ngôn ngữ lập trình nào khác, kể cả để giải thích khái niệm**
— không viết kiểu "giống X trong Python/JS/Go". Ngoại lệ duy nhất là C/C++ (mốc so sánh hiệu
năng/quản lý bộ nhớ đã dùng trong `title-page.md`). Đây là tài liệu Rust thuần — nêu thẳng khái niệm
bằng ngữ cảnh của chính Rust.

**Khi hướng dẫn cài đặt một công cụ**, ưu tiên cách không bắt người đọc cài thêm gì khác trước đó
(script cài đặt chính thức qua `curl`/`sh`, hoặc thứ đã có sẵn từ bước cài trước trong sách) hơn là
cách đòi hỏi một trình quản lý gói trung gian như Homebrew. Không nhắc tới Homebrew.

Đây là bản nháp đầu — nói rõ với tác giả đây là draft cần họ đọc lại, không phải bản chốt.

## 5. Xác minh trước khi coi là xong

````bash
mdbook test    # mọi khối ```rust phải biên dịch được, trừ khi cố ý đánh dấu ignore/compile_fail
mdbook build   # bắt lỗi link nội bộ hỏng (mục lục trỏ sai đường dẫn)
dprint fmt     # format lại theo dprint.json (bọc dòng ở 100 ký tự)
````

Nếu `mdbook test` báo lỗi ở một khối code không nhằm minh hoạ lỗi biên dịch, sửa code chứ đừng thêm
`ignore` để né lỗi.

Nếu chương có ví dụ đụng tới filesystem thật (`File::open`, `File::create`, ...): chạy `mdbook test`
**ít nhất hai lần liên tiếp** trước khi coi là xong. `mdbook test` chia sẻ một working directory bền
giữa các khối code và giữa các lần chạy khác nhau — một ví dụ tạo file ở khối này có thể âm thầm đổi
kết quả của một ví dụ khác mong đợi file đó không tồn tại (đã xảy ra thật khi viết chương Xử lý
lỗi). Dùng `no_run` cho ví dụ chỉ cần minh hoạ cú pháp, hoặc đặt tên file duy nhất không trùng ví dụ
nào khác trong sách khi cần chạy thật.
