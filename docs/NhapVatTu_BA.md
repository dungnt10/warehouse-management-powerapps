
# Nhập vật tư – Tài Liệu Business Analyst (BA)

## 1. Mục đích
Xây dựng chức năng **Nhập vật tư** trong hệ thống **quản lý kho** sử dụng Power Apps + Dataverse. Tách biệt việc nhập dữ liệu tạm và phê duyệt chính thức.

## 2. Data & Collection

### Dataverse Tables
- `KhoVatTu`: Danh sách vật tư trong kho
- `KhoVatTu_NhapKho`: Lịch sử nhập kho

### Collections trong app
- `colKhoVatTu`: Load danh sách vật tư ban đầu (OnVisible)
- `colVatTuNhap`: Lưu danh sách vật tư được chọn nhập + số lượng + thông tin bổ sung

## 3. UI Flow

### 3.1 Bên Trái: Danh sách vật tư
- Control: `galVatTu`
- Checkbox: `chkChonVatTuNhap`
- Sự kiện: `OnCheck`

```powerapps
Collect(
    colVatTuNhap,
    {
        MaVatTu: ThisItem.MaVatTu,
        TenVatTu: ThisItem.TenVatTu,
        DonViTinh: ThisItem.DonViTinh,
        SoLuongNhap: 0,
        DonGia: 0,
        NgayNhap: Blank(),
        NguoiNhap: "",
        NhaCungCap: "",
        GhiChu: "",
        LoaiVatTu: ThisItem.LoaiVatTu
    }
)
```

### 3.2 Bên Phải: Danh sách vật tư đã chọn
- Control: `galNhapKho`, Items = `colVatTuNhap`

#### Các trường nhập liệu trong gallery:
- **TextInput: `txtSoLuongNhap`**
  - `PlaceholderText`: `ThisItem.SoLuongNhap`
  - `OnChange`:
    ```powerapps
    Patch(colVatTuNhap, ThisItem, { SoLuongNhap: Value(txtSoLuongNhap.Text) })
    ```

- **TextInput: `txtDonGia`**
  - `PlaceholderText`: `ThisItem.DonGia`
  - `OnChange`:
    ```powerapps
    Patch(colVatTuNhap, ThisItem, { DonGia: Value(txtDonGia.Text) })
    ```

- **DatePicker: `dpNgayNhap`**
  - `DefaultDate`: `Today()`

- **TextInput: `txtNguoiNhap`**
  - Nhập tên người thực hiện

- **TextInput: `txtNhaCungCap`**
  - Nhập tên nhà cung cấp

- **TextInput (Multiline): `txtGhiChuNhap`**
  - Nhập ghi chú nhập hàng

### 3.3 Nút "Nhập"
- `OnSelect`:
```powerapps
ForAll(
    colVatTuNhap,
    Patch(
        KhoVatTu_NhapKho,
        Defaults(KhoVatTu_NhapKho),
        {
            MaVatTu: MaVatTu,
            TenVatTu: TenVatTu,
            DonViTinh: DonViTinh,
            SoLuong: SoLuongNhap,
            DonGia: DonGia,
            NgayNhap: dpNgayNhap.SelectedDate,
            NguoiNhap: txtNguoiNhap.Text,
            NhaCungCap: txtNhaCungCap.Text,
            GhiChu: txtGhiChuNhap.Text,
            LoaiVatTu: LoaiVatTu
        }
    )
);
Clear(colVatTuNhap);
Notify("Đã ghi nhận nhập kho thành công!", NotificationType.Success)
```

---

**Cập nhật ngày:** 04/04/2025
