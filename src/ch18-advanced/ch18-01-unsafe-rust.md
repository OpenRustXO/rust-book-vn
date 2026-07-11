# Tính năng nâng cao

Bốn chủ đề còn lại, mỗi cái mở ra một góc khác của Rust ít gặp trong code hàng ngày nhưng cần biết
để đọc hiểu code hệ thống, thư viện nền tảng, hay code sinh tự động: `unsafe` (tắt một số kiểm tra
của trình biên dịch có kiểm soát), trait/type nâng cao, và macro (sinh code lúc biên dịch).

## Unsafe Rust

Mọi đảm bảo an toàn bộ nhớ nói từ đầu sách — borrow checker, không tham chiếu treo, không data race
— đều do trình biên dịch **tĩnh** kiểm tra được, nghĩa là nó phải từ chối cả một số đoạn code **thật
ra vẫn an toàn** nhưng trình biên dịch không đủ thông tin để chứng minh điều đó. `unsafe` là lối
thoát có kiểm soát cho đúng những trường hợp đó — mở khoá thêm 5 khả năng bị cấm ở Rust thường:

1. Giải tham chiếu một con trỏ thô (raw pointer).
2. Gọi một hàm hoặc method `unsafe`.
3. Truy cập/sửa một biến `static mut`.
4. Implement một trait `unsafe`.
5. Truy cập field của một `union`.

Quan trọng: `unsafe` **không tắt** borrow checker hay bất kỳ kiểm tra nào khác — nó chỉ mở thêm đúng
5 khả năng trên, còn mọi quy tắc khác vẫn áp dụng nguyên vẹn.

### Con trỏ thô

```rust
fn main() {
    let mut num = 5;

    let r1 = &raw const num;
    let r2 = &raw mut num;

    unsafe {
        println!("r1: {}", *r1);
        *r2 += 1;
        println!("r2: {}", *r2);
    }
}
```

`*const T`/`*mut T` khác tham chiếu thường ở chỗ: có thể null, không đảm bảo trỏ tới bộ nhớ hợp lệ,
và **được phép** có nhiều con trỏ thô khả biến cùng trỏ tới một chỗ cùng lúc — borrow checker không
theo dõi con trỏ thô. Tạo con trỏ thô (`&raw const`/`&raw mut`) là an toàn; chỉ **giải tham chiếu**
(`*r1`) mới cần khối `unsafe`, vì đó là lúc thật sự có rủi ro đọc/ghi vùng nhớ không hợp lệ.

### Dùng unsafe để viết một API an toàn

`unsafe` bên trong không có nghĩa là API công khai phải `unsafe` theo — cách dùng phổ biến nhất: bọc
một thao tác mà bản thân nó cần `unsafe` (vì borrow checker không đủ thông minh để tự chứng minh)
bên trong một hàm thường, nếu người viết tự đảm bảo được lời gọi đó luôn an toàn:

```rust
fn chia_doi_kha_bien(day: &mut [i32], vi_tri: usize) -> (&mut [i32], &mut [i32]) {
    let do_dai = day.len();
    let con_tro = day.as_mut_ptr();

    assert!(vi_tri <= do_dai);

    unsafe {
        (
            std::slice::from_raw_parts_mut(con_tro, vi_tri),
            std::slice::from_raw_parts_mut(con_tro.add(vi_tri), do_dai - vi_tri),
        )
    }
}

fn main() {
    let mut v = vec![1, 2, 3, 4, 5, 6];
    let (a, b) = chia_doi_kha_bien(&mut v, 3);
    println!("{a:?} {b:?}");
}
```

Trả về **hai** tham chiếu khả biến từ **cùng một** slice là điều borrow checker không tự chứng minh
được an toàn (nó không biết hai nửa đó không chồng lấn nhau) — nhưng người viết hàm này biết chắc,
vì `vi_tri <= do_dai` đã kiểm tra và hai slice ghép từ điểm cắt đó chưa bao giờ chồng lấn. `unsafe`
cho phép diễn đạt sự thật đó, `assert!` bên ngoài đảm bảo lời hứa đó luôn đúng trước khi vào phần
`unsafe`. Người gọi `chia_doi_kha_bien` không cần biết gì về `unsafe` cả — API nhìn từ ngoài hoàn
toàn an toàn.

### static mut

Một biến `static` khai báo `mut` là trạng thái toàn cục, khả biến — truy cập **hay** sửa nó đều cần
`unsafe`, kể cả chỉ đọc, vì nhiều luồng cùng truy cập một `static mut` không đồng bộ chính là một
data race tiềm ẩn mà trình biên dịch không kiểm soát được:

```rust
static mut BO_DEM: i32 = 0;

fn tang_bo_dem() {
    unsafe {
        *(&raw mut BO_DEM) += 1;
    }
}

fn main() {
    tang_bo_dem();
    tang_bo_dem();

    unsafe {
        println!("{}", *(&raw const BO_DEM));
    }
}
```

Trong code thực tế, gần như luôn nên dùng `std::sync::atomic` hoặc `Mutex<T>` (chương Concurrency)
thay vì `static mut` — cả hai cho trạng thái toàn cục khả biến **an toàn** giữa nhiều luồng mà không
cần `unsafe` ở nơi dùng. `static mut` chủ yếu còn xuất hiện trong code cấp rất thấp hoặc tương thích
ngược.
