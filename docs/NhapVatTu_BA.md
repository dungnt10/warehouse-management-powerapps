# Nhập vật tư - Tài liệu Business Analyst (BA)

## 1. Mục đích
Xây dựng chức năng **Nhập vật tư** trong hệ thống **quản lý kho** sử dụng Power Apps + Dataverse. Tách biệt việc nhập dữ liệu tạm và phê duyệt chính thức.

## 2. Data & Collection
### Dataverse Tables
- `KhoVatTu`: Danh sách vật tư trong kho
- `KhoVatTu_NhapKho`: Lịch sử nhập kho

### Collections trong app
- `colKhoVatTu`: Load danh sách vật tư ban đầu (OnVisible)
- `colVatTuNhap`: Lưu danh sách vật tư được chọn nhập + số lượng

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
    SoLuongNhap: 0
  }
)
```

### 3.2 Bên Phải: Danh sách vật tư đã chọn
- Control: `galNhapKho`, Items = `colVatTuNhap`
- TextInput: `txtSoLuongNhap`
- PlaceholderText: `ThisItem.SoLuongNhap`
- Sự kiện `OnChange`:
```powerapps
Patch(
  colVatTuNhap,
  ThisItem,
  {
    SoLuongNhap: Value(txtSoLuongNhap.Text)
  }
)
```

### 3.3 Nút "Nhập"
- Button: `btnNhapKho`
- Code `OnSelect`:
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
      SoLuong: SoLuongNhap
    }
  )
);
Clear(colVatTuNhap);
Notify("Đã ghi nhận nhập kho thành công!", NotificationType.Success)
```

## 4. Lỗi thường gặp
- Patch không vào cột `SoLuong` vì sai tên schema (phải dùng đúng display name `SoLuong`)
- TextInput trong Gallery cần dùng PlaceholderText thay vì Default

## 5. Ghi chú
- Chưa có chức năng phê duyệt tên user/ghi ngày nhập
- Dự kiến tách bước phê duyệt sau: Patch user + datetime

