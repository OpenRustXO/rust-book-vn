# CLAUDE.md

Hướng dẫn cho Claude Code khi làm việc trong repo này.

## Dự án là gì

**rust-book-vn** (OpenRustXO) — tài liệu học ngôn ngữ lập trình Rust bằng tiếng Việt, build bằng
[mdBook](https://rust-lang.github.io/mdBook/). Đây **không phải bản dịch máy móc** từ sách gốc, mà
là giáo trình/ghi chú cá nhân của tác giả. Triết lý viết nêu rõ trong `README.md`: học hiểu bản
chất, nội dung rõ ràng có cấu trúc, đào sâu nhưng không lan man.

## Cấu trúc thư mục

- `book.toml` — cấu hình mdBook: `language = "vi"`, `rust.edition = "2024"`, mục lục có gấp
  (`fold`), tìm kiếm AND theo mặc định.
- `src/SUMMARY.md` — mục lục, **là nguồn sự thật duy nhất** về cấu trúc sách. mdBook chỉ build các
  file được liệt kê ở đây (thêm file mà quên khai báo ở đây thì file đó không xuất hiện trong sách).
- `src/chNN-slug/chNN-MM-slug.md` — mỗi chương lớn là một thư mục `chNN-slug/` (slug mô tả ngắn,
  không dấu), các mục con là file `chNN-MM-slug.md` bên trong. Khác với sách gốc tiếng Anh, project
  này **không có file mục lục chNN-00** — các mục con bắt đầu thẳng từ `chNN-01`. Theo đúng ví dụ
  hiện có: thư mục `ch01-install/`, file `ch01-01-getting-started.md`.
- `book/` — output build tĩnh của mdBook, đã có trong `.gitignore`, không commit.
- `dprint.json` — format Markdown: bọc dòng cứng (`textWrap: "always"`) ở 100 ký tự.

## Lệnh hay dùng

````bash
mdbook serve        # xem trước tại localhost, tự rebuild khi sửa file trong src/
mdbook build        # build tĩnh ra book/
mdbook test         # chạy mọi khối ```rust trong sách như doctest — bắt buộc trước khi
                     # coi một chương có code là xong
dprint fmt          # format lại Markdown theo dprint.json — chạy trước khi commit
dprint check        # kiểm tra format mà không sửa
````

## Quy ước viết nội dung

- Không dịch nguyên văn từ [rust-lang/book](https://github.com/rust-lang/book). Dùng sách gốc để đối
  chiếu kỹ thuật cho đúng, nhưng diễn giải lại bằng giọng văn và ví dụ riêng.
- Thuật ngữ kỹ thuật phải nhất quán xuyên suốt sách. Trước khi dùng một thuật ngữ Rust lần đầu
  (ownership, borrowing, trait, lifetime, v.v.), kiểm tra
  `.claude/skills/terminology-check/glossary.md` xem đã có cách dịch/giữ nguyên chưa; nếu chưa, bàn
  với tác giả rồi thêm vào glossary.
- Mọi khối `` ```rust `` phải biên dịch được — xác nhận bằng `mdbook test`. Nếu chủ đích minh hoạ
  lỗi biên dịch, đánh dấu bằng thuộc tính mdBook tương ứng (`ignore`, `does_not_compile`, ...) thay
  vì để mdBook cố chạy nó như một ví dụ hợp lệ.
- Mỗi mục nên ngắn gọn, đúng trọng tâm của tiêu đề; không nhồi kiến thức của chương sau vào chương
  hiện tại.

## Skill trong dự án

- `new-chapter` — tạo khung một chương/mục mới đúng quy ước (`SUMMARY.md`, đường dẫn file) và soạn
  nội dung nháp theo giọng văn của sách.
- `terminology-check` — rà soát tính nhất quán thuật ngữ trong toàn bộ `src/`, cập nhật glossary.
- `review-writing` — rà soát văn phong/cấu trúc/thuật ngữ/code của một chương đã có trước khi coi là
  hoàn thiện.
