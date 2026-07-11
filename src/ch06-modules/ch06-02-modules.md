# Module và phạm vi truy cập

`mod` gom các item liên quan (hàm, struct, enum, module con...) vào một khối có tên, tạo thành một
**cây module** — module gốc của crate root nằm ở đỉnh cây, tên là `crate`.

```rust
mod thu_vien {
    pub mod muon_tra {
        pub fn muon_sach() {
            println!("Đã mượn sách");
        }

        pub fn tra_sach() {
            super::sap_xep_lai();
        }
    }

    fn sap_xep_lai() {
        println!("Sắp xếp lại kệ sách");
    }
}

fn main() {
    crate::thu_vien::muon_tra::muon_sach();
    crate::thu_vien::muon_tra::tra_sach();
}
```

## Quy tắc riêng tư

Mọi item — hàm, struct, module con — **mặc định riêng tư** (private), chỉ nhìn thấy được từ chính
module khai báo nó và các module **con cháu** của module đó. Muốn một module hay item bên ngoài nhìn
thấy được, phải khai báo `pub` tường minh. Đây là lựa chọn mặc định ngược với "mọi thứ đều công khai
trừ khi giấu đi" — buộc người viết code phải chủ động quyết định phần nào là API công khai, thay vì
lộ ra ngoài mọi chi tiết cài đặt bên trong theo mặc định.

Quy tắc nhìn thấy đi theo một chiều duy nhất: module **con** nhìn thấy được item riêng tư của module
**cha** (và mọi tổ tiên của nó), nhưng module cha **không** tự nhìn thấy được item riêng tư của con
trừ khi con khai báo `pub`. Ở ví dụ trên, `sap_xep_lai` là hàm riêng tư của `thu_vien`, nhưng gọi
được từ `muon_tra` (module con của `thu_vien`) — ngược lại thì không:

```rust,compile_fail
mod thu_vien {
    pub mod muon_tra {
        fn ghi_log() {}
    }

    fn kiem_tra() {
        muon_tra::ghi_log(); // lỗi: `ghi_log` là hàm riêng tư của module `muon_tra`
    }
}

fn main() {}
```

Trực giác: một module con biết rõ ngữ cảnh nội bộ của cha (nó được định nghĩa bên trong cha), nên
được tin tưởng truy cập chi tiết cài đặt của cha; nhưng cha không nên phải biết chi tiết cài đặt bên
trong của từng con — đó chính là điều giúp con được tự do thay đổi cài đặt nội bộ mà không ảnh hưởng
tới cha, miễn giữ nguyên phần đã công khai.

## pub trên struct và enum

`pub` trên một `struct` chỉ công khai chính kiểu đó — **từng field vẫn riêng tư theo mặc định**,
phải đánh dấu `pub` cho từng field muốn lộ ra ngoài:

```rust
mod bep {
    pub struct BuaAn {
        pub mon_chinh: String,
        gia_tien: f64, // vẫn riêng tư dù struct đã pub
    }

    impl BuaAn {
        pub fn moi(mon_chinh: &str, gia_tien: f64) -> BuaAn {
            BuaAn {
                mon_chinh: String::from(mon_chinh),
                gia_tien,
            }
        }
    }
}

fn main() {
    let bua = bep::BuaAn::moi("Phở", 45.0);
    println!("{}", bua.mon_chinh); // được, vì field này pub
}
```

`enum` thì ngược lại: một `enum` đã `pub` thì **mọi variant của nó tự động công khai theo** — nếu
phải khai báo `pub` cho từng variant sẽ rất rườm rà, vì gần như luôn cần dùng được hết các variant
mới coi enum đó có ích.
