# Trait object và hướng đối tượng

## Vấn đề: một collection chứa nhiều kiểu cụ thể khác nhau

`Vec<T>` chỉ chứa được một `T`. Generic (chương Generic, Trait, Lifetime) cũng chỉ chọn **một** kiểu
cụ thể tại mỗi lần dùng — không giúp được khi cần một danh sách các thành phần giao diện, mỗi phần
tử là một kiểu cụ thể **khác nhau**, nhưng tất cả cùng cần vẽ được:

```rust
pub trait Ve {
    fn ve(&self);
}

pub struct Nut {
    pub nhan: String,
}

impl Ve for Nut {
    fn ve(&self) {
        println!("Vẽ nút bấm: {}", self.nhan);
    }
}

pub struct O {
    pub da_chon: bool,
}

impl Ve for O {
    fn ve(&self) {
        println!("Vẽ ô chọn, đã chọn: {}", self.da_chon);
    }
}

fn main() {
    let cac_thanh_phan: Vec<Box<dyn Ve>> = vec![
        Box::new(Nut {
            nhan: String::from("Đồng ý"),
        }),
        Box::new(O { da_chon: true }),
    ];

    for tp in cac_thanh_phan.iter() {
        tp.ve();
    }
}
```

`Box<dyn Ve>` là một **trait object**: một con trỏ tới dữ liệu (`Box`) cộng một con trỏ tới bảng
phương thức (vtable) của trait `Ve` ứng với đúng kiểu cụ thể đang nằm trong `Box` đó. Kiểu
`Vec<Box<dyn Ve>>` chứa được `Nut` lẫn `O` cùng lúc — điều `Vec<T>` với `T` cố định không bao giờ
làm được.

## Static dispatch và dynamic dispatch

Generic (từ chương trước) dùng **static dispatch**: trình biên dịch biết chính xác kiểu cụ thể ngay
lúc biên dịch (nhờ monomorphization), gọi thẳng đúng hàm — không có gì phải tra cứu lúc chạy, nhưng
đổi lại một collection generic chỉ chứa được một kiểu cụ thể.

Trait object dùng **dynamic dispatch**: kiểu cụ thể đằng sau `dyn Ve` chỉ biết được lúc chạy, nên
gọi `tp.ve()` phải tra vtable để tìm đúng hàm `ve` của kiểu cụ thể đang nằm ở đó — thêm một lượt tra
cứu gián tiếp so với static dispatch, và trình biên dịch cũng mất khả năng inline lời gọi đó (vì
không biết trước sẽ gọi hàm nào). Đổi lại, có được thứ static dispatch không thể: một collection
chứa nhiều kiểu cụ thể khác nhau, miễn cùng implement một trait.

Đây là đánh đổi tường minh giữa hai công cụ giải quyết hai vấn đề khác nhau: cần hiệu năng tối đa và
chỉ một kiểu cụ thể tại một chỗ — dùng generic; cần chứa nhiều kiểu cụ thể không biết trước hết ngay
lúc viết code — dùng trait object.

## Không phải trait nào cũng thành trait object được

Một trait chỉ **an toàn cho object** (object-safe) nếu mọi method của nó không trả về `Self` và
không có tham số kiểu generic — cả hai điều đó đòi hỏi biết kiểu cụ thể ngay lúc biên dịch, mâu
thuẫn với bản chất "kiểu cụ thể chỉ biết lúc chạy" của trait object. Cố tạo `dyn` từ một trait không
thoả điều kiện này là lỗi biên dịch, kèm gợi ý rõ ràng trait đó "không thể trở thành object".
