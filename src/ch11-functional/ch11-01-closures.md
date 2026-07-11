# Closure và Iterator

Hai công cụ trung tâm của phong cách lập trình hàm trong Rust: **closure** — hàm ẩn danh có thể giữ
lại (capture) biến từ ngữ cảnh xung quanh nơi nó được tạo, và **iterator** — cách xử lý một chuỗi
giá trị theo từng bước, tính lười (lazy), không tạo collection trung gian nếu không cần. Cả hai đều
biên dịch xuống hiệu năng ngang code viết tay bằng vòng lặp thông thường — trừu tượng hoá không có
nghĩa là đánh đổi tốc độ.

## Closure

```rust
fn main() {
    let cong_them = |x: i32, y: i32| x + y;
    println!("{}", cong_them(2, 3));
}
```

Cú pháp `|tham_so| bieu_thuc` định nghĩa một closure. Khác `fn`, kiểu tham số/trả về thường **không
cần** viết tường minh — trình biên dịch suy luận từ lần gọi đầu tiên. Vì closure thường dùng ngay
tại chỗ, trong một phạm vi hẹp, phần lớn lợi ích của việc ghi kiểu tường minh cho `fn` (chữ ký là
tài liệu công khai, đã nói ở chương Hàm) không áp dụng ở đây.

Suy luận chỉ diễn ra **một lần**, khoá cứng vào kiểu cụ thể đầu tiên gặp phải — không giống generic:

```rust,compile_fail
fn main() {
    let identity = |x| x;

    let n = identity(5);
    let s = identity(String::from("chào")); // lỗi: kiểu đã khoá thành i32 từ lần gọi trước
}
```

## Bắt giữ môi trường xung quanh

Điều closure làm được mà một `fn` thường không làm được: **capture** biến từ scope bao quanh nơi nó
định nghĩa.

```rust
fn main() {
    let ten_cong_ty = String::from("Rust Corp");
    let gioi_thieu = || println!("Chào mừng tới {ten_cong_ty}");
    gioi_thieu();
}
```

`gioi_thieu` "nhìn thấy" `ten_cong_ty` dù đó không phải tham số — closure tự động mượn các biến nó
dùng tới từ scope cha. Có ba cách capture, tương ứng ba trait mà mọi closure implement tuỳ theo cách
nó thật sự dùng biến capture được:

- **`FnOnce`**: mọi closure ít nhất implement trait này — có thể gọi được, ít nhất một lần. Closure
  _chỉ_ implement `FnOnce` (không implement thêm hai trait dưới) là closure **move** giá trị capture
  được ra khỏi thân nó, nên chỉ gọi được đúng một lần (gọi lần hai sẽ dùng một giá trị đã bị move).
- **`FnMut`**: capture bằng tham chiếu khả biến, có thể gọi nhiều lần, mỗi lần có thể sửa giá trị đã
  capture.
- **`Fn`**: capture bằng tham chiếu bất biến (hoặc không capture gì cả), gọi được nhiều lần, không
  bao giờ sửa gì.

```rust
fn main() {
    let mut dem = 0;
    let mut tang_dem = || {
        dem += 1;
        println!("Đếm: {dem}");
    };

    tang_dem();
    tang_dem();
}
```

`tang_dem` sửa `dem` mỗi lần gọi — closure này capture `dem` bằng tham chiếu khả biến, nên biến
`tang_dem` phải khai báo `mut` và implement `FnMut` (không implement `Fn`, vì `Fn` không được phép
sửa gì).

## Từ khoá move

Thêm `move` trước closure ép nó **lấy quyền sở hữu** mọi biến capture được, thay vì chỉ mượn — cần
thiết khi closure phải sống lâu hơn scope hiện tại, ví dụ chuyển cho một luồng khác chạy (nói kỹ ở
[chương Concurrency](../ch14-concurrency/ch14-01-threads.md)):

```rust
fn main() {
    let danh_sach = vec![1, 2, 3];

    let in_ra = move || println!("{danh_sach:?}");

    in_ra();
    // danh_sach đã bị move vào closure, không dùng lại được ở đây
}
```

## Nhận closure làm tham số

Một hàm nhận closure qua ràng buộc trait `Fn`/`FnMut`/`FnOnce` — chọn trait nào tuỳ hàm cần gọi
closure bao nhiêu lần và có sửa gì bên trong hay không:

```rust
fn ap_dung_hai_lan<F>(f: F) -> i32
where
    F: Fn(i32) -> i32,
{
    f(1) + f(2)
}

fn main() {
    let nhan_ba = |x| x * 3;
    println!("{}", ap_dung_hai_lan(nhan_ba));
}
```

Mỗi closure có một kiểu vô danh riêng biệt do trình biên dịch tự sinh — không có cách viết ra kiểu
cụ thể đó, nên tham số nhận closure luôn phải là generic (`F: Fn(...)...`) hoặc `impl Fn(...)...`,
không bao giờ viết được một kiểu cụ thể cho tham số đó.
