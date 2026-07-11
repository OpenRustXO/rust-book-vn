# Bắt đầu

Trước khi viết dòng code Rust đầu tiên, cần có ba thứ trên máy: trình biên dịch Rust, Cargo — công
cụ build kiêm quản lý dependency đi kèm, và cargo-binstall — tiện ích cài nhanh các công cụ dòng
lệnh viết bằng Rust mà không phải build từ mã nguồn. Phần này hướng dẫn cài đặt cả ba trên macOS,
dùng phiên bản Rust ổn định (stable) mới nhất.

## Cài đặt Rust với rustup

`rustup` là công cụ chính thức để cài đặt và quản lý phiên bản Rust trên máy: tải toolchain, chuyển
đổi giữa các kênh phát hành (stable/beta/nightly), thêm target để biên dịch chéo, và khoá phiên bản
riêng cho từng dự án. Cài bằng script chính thức:

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

Script sẽ hỏi vài tuỳ chọn cài đặt, chọn `1) Proceed with installation (default)` là đủ cho hầu hết
trường hợp. Sau khi cài xong, mở lại terminal (hoặc chạy `source "$HOME/.cargo/env"`) để `rustc` và
`cargo` có trong `PATH`.

Kiểm tra đã cài đúng:

```bash
rustc --version
```

Lệnh trên in ra phiên bản stable hiện có trên máy, ví dụ `rustc 1.82.0 (...)`. Khi có bản mới hơn,
cập nhật bằng:

```bash
rustup update
```

Ngoài bản mặc định, có thể chủ động cài thêm toolchain khác — một phiên bản cụ thể, hoặc kênh
`beta`/`nightly` — song song mà không ảnh hưởng tới bản đang dùng:

```bash
rustup toolchain install 1.90.0
rustup toolchain install beta
rustup toolchain install nightly
```

Toolchain vừa cài không tự động trở thành mặc định. Muốn chạy thử một lệnh với toolchain cụ thể mà
không đổi mặc định, thêm tiền tố `+<toolchain>`:

```bash
cargo +1.90.0 build
```

Muốn đổi hẳn sang toolchain đó làm mặc định cho toàn máy:

```bash
rustup default 1.90.0
```

Xem toàn bộ toolchain đã cài trên máy — có thể có nhiều hơn một nếu từng cài thêm bản beta/nightly
hoặc một phiên bản cụ thể để thử:

```bash
rustup toolchain list
```

Gỡ một toolchain không dùng tới nữa:

```bash
rustup toolchain uninstall nightly
```

### Target biên dịch

Một target (target triple) mô tả nền tảng sẽ chạy binary sinh ra — kiến trúc CPU, hệ điều hành, và
ABI, ví dụ `aarch64-apple-darwin` cho Mac Apple Silicon hay `wasm32-unknown-unknown` cho
WebAssembly. Khi cài rustup, chỉ target khớp với máy đang dùng được cài sẵn — đủ để build và chạy
ngay trên máy, nhưng chưa đủ nếu muốn biên dịch chéo (cross-compile) sang nền tảng khác.

Xem target đã cài và toàn bộ target khả dụng:

```bash
rustup target list --installed
rustup target list
```

Thêm một target mới:

```bash
rustup target add wasm32-unknown-unknown
```

Gỡ một target không cần nữa:

```bash
rustup target remove wasm32-unknown-unknown
```

### Cấu hình phiên bản Rust cho từng dự án

Mặc định rustup dùng một toolchain chung cho toàn máy. Một dự án cụ thể có thể cần khoá cứng ở một
phiên bản hoặc kênh khác — khai báo trong file `rust-toolchain.toml` ở thư mục gốc dự án:

```toml
[toolchain]
channel = "stable"
```

Mỗi khi chạy `rustc`/`cargo` trong thư mục có file này (hoặc thư mục con của nó), rustup tự nhận ra
và dùng đúng phiên bản khai báo, tự tải về nếu máy chưa có — không cần đổi gì thủ công. Muốn khoá
một bản cụ thể thay vì luôn theo `stable` mới nhất, thay giá trị `channel` bằng số hiệu, ví dụ
`channel = "1.82.0"`. File này nằm trong dự án nên commit vào git, để ai clone về cũng dùng đúng
phiên bản.

Nếu chỉ cần đổi phiên bản tạm thời trên máy cá nhân, không muốn ghi vào file chia sẻ qua git, dùng
override theo thư mục thay thế:

```bash
rustup override set 1.82.0
```

Gỡ override để quay lại dùng toolchain mặc định của máy:

```bash
rustup override unset
```

Xem toàn cảnh trạng thái hiện tại — toolchain nào đang active, active vì lý do gì (default hay
override), và những target nào đã cài cho toolchain đó:

```bash
rustup show
```

## Cargo

Cargo được cài kèm tự động cùng rustup, không cần cài riêng. Đây là build tool kiêm trình quản lý
dependency của Rust: biên dịch, chạy, test, và tải các crate (thư viện) mà dự án phụ thuộc vào. Toàn
bộ ví dụ trong sách chạy qua `cargo` thay vì gọi `rustc` trực tiếp.

Kiểm tra đã có Cargo:

```bash
cargo --version
```

## cargo-binstall

`cargo install` cài một binary bằng cách biên dịch từ mã nguồn: tải crate, tải toàn bộ dependency,
rồi compile — với những crate lớn có thể mất vài phút và cần đủ toolchain để build.
[`cargo-binstall`](https://github.com/cargo-bins/cargo-binstall) là một subcommand thay thế: tìm bản
binary đã build sẵn của đúng crate đó đính kèm trong GitHub Releases, khớp với target của máy đang
dùng, tải về, kiểm tra tính toàn vẹn rồi cài thẳng — bỏ qua hoàn toàn bước compile, thường chỉ mất
vài giây thay vì vài phút.

Nếu crate không có sẵn bản build cho target đang dùng, cargo-binstall tự động rơi về biên dịch từ mã
nguồn giống hệt `cargo install`. Nhờ vậy `cargo binstall <tên-crate>` dùng thay được cho
`cargo install <tên-crate>` trong mọi trường hợp mà không mất khả năng cài đặt — chỉ được lợi thêm
tốc độ khi có sẵn bản build. Sách này dùng `cargo binstall` bất cứ khi nào cần cài một công cụ dòng
lệnh viết bằng Rust.

Cài bằng script chính thức — tải sẵn binary, không cần cài thêm công cụ nào khác ngoài `curl` (đã có
sẵn trên macOS):

```bash
curl -L --proto '=https' --tlsv1.2 -sSf https://raw.githubusercontent.com/cargo-bins/cargo-binstall/main/install-from-binstall-release.sh | bash
```

Kiểm tra (lưu ý dùng `-V` chứ không phải `--version` — cờ `--version` của cargo-binstall dùng để chỉ
định phiên bản của package cần cài, không phải để in phiên bản của chính công cụ):

```bash
cargo binstall -V
```
