# Trait có thể derive

Attribute `#[derive(...)]` tự sinh implementation cho một số trait chuẩn, dựa trên cấu trúc field
của struct/enum — chỉ áp dụng được khi **mọi** field cũng implement đúng trait đó (derive `Clone`
cho một struct chứa field không phải `Clone` sẽ báo lỗi biên dịch).

| Trait        | Tự sinh gì                                                                                                         |
| ------------ | ------------------------------------------------------------------------------------------------------------------ |
| `Debug`      | định dạng in ra phục vụ debug qua `{:?}`/`{:#?}` — gần như luôn nên derive cho mọi kiểu tự định nghĩa              |
| `PartialEq`  | so sánh bằng nhau (`==`, `!=`) — so từng field, tất cả field phải bằng nhau thì hai instance mới bằng nhau         |
| `Eq`         | khẳng định `PartialEq` của kiểu này luôn phản xạ (`x == x` luôn đúng) — không cho `f32`/`f64` vì `NaN != NaN`      |
| `PartialOrd` | so sánh thứ tự (`<`, `>`, ...) — so field theo đúng thứ tự khai báo, field nào khác trước quyết định kết quả       |
| `Ord`        | như `PartialOrd` nhưng khẳng định luôn so sánh được với mọi giá trị khác (không như `f32`/`f64` với `NaN`)         |
| `Clone`      | sinh method `.clone()` — deep clone từng field                                                                     |
| `Copy`       | cho phép gán bằng cách copy bit thay vì move — chỉ hợp lệ nếu mọi field đều `Copy` (không có `String`, `Vec`, ...) |
| `Hash`       | sinh cách băm giá trị — cần để dùng làm khoá trong `HashMap`/`HashSet`                                             |
| `Default`    | sinh hàm `::default()` trả về một instance với giá trị mặc định cho từng field                                     |

Thứ tự thường đi kèm nhau: một kiểu dùng làm khoá `HashMap` cần cả `Eq` lẫn `Hash`; một kiểu cần sắp
xếp (`Vec::sort`) cần `Ord`.

```rust
#[derive(Debug, Clone, PartialEq, Eq, Hash, Default)]
struct NguoiDung {
    id: u32,
    ten: String,
}

fn main() {
    let a = NguoiDung {
        id: 1,
        ten: String::from("An"),
    };
    let b = a.clone();

    println!("{a:?}");
    assert_eq!(a, b);

    let mac_dinh = NguoiDung::default();
    println!("{mac_dinh:?}");
}
```
