# 🧾 Nhập vật tư – Tài Liệu Business Analyst (BA)

## 1. 🎯 Mục đích
Xây dựng chức năng **Nhập vật tư** trong hệ thống **quản lý kho** sử dụng Power Apps + Dataverse. Mục tiêu:

- Cho phép nhập nhiều vật tư cùng lúc.
- Ghi nhận đầy đủ thông tin: số lượng, đơn giá, người nhập, ngày nhập,...
- Phân tách rõ giữa dữ liệu tạm và xác nhận chính thức.

---

## 2. 📊 Data & Collection

### 🔹 Dataverse Tables
- `KhoVatTu`: Danh sách vật tư trong kho
- `KhoVatTu_NhapKho`: Lịch sử nhập kho

### 🔹 Collections trong app
- `colKhoVatTu`: Load danh sách vật tư ban đầu (OnVisible)
- `colVatTuNhap`: Lưu danh sách vật tư được chọn để nhập (tạm thời)

---

## 3. 🖼 UI Flow – Màn hình `scrNhapKho`

### 3.1 Bên trái: Danh sách vật tư

- **Control**: `galVatTu`
- **Checkbox**: `chkChonVatTuNhap`
- **Sự kiện**: `OnCheck`
```powerapps
Collect(
    colVatTuNhap,
    {
        MaVatTu: ThisItem.MaVatTu,
        TenVatTu: ThisItem.TenVatTu,
        DonViTinh: ThisItem.DonViTinh,
        SoLuongNhap: 0,
        DonGia: 0,
        NgayNhap: Today(),
        NguoiNhap: "",
        NhaCungCap: "",
        GhiChu: "",
        LoaiVatTu: ""
    }
)
```

---

### 3.2 Bên phải: Danh sách vật tư đã chọn

- **Control**: `galNhapKho`, `Items = colVatTuNhap`

#### Các trường nhập trong gallery:

| Trường        | Control           | PlaceholderText               | Sự kiện Patch |
|---------------|-------------------|-------------------------------|----------------|
| Số lượng      | `txtSoLuongNhap`  | `ThisItem.SoLuongNhap`        | `Patch(colVatTuNhap, ThisItem, {SoLuongNhap: Value(Self.Text)})` |
| Đơn giá       | `numDonGia`       | `ThisItem.DonGia`             | `Patch(colVatTuNhap, ThisItem, {DonGia: Value(Self.Text)})` |
| Ngày nhập     | `dpNgayNhap`      | `ThisItem.NgayNhap`           | `Patch(colVatTuNhap, ThisItem, {NgayNhap: dpNgayNhap.SelectedDate})` |
| Người nhập    | `txtNguoiNhap`    | `ThisItem.NguoiNhap`          | `Patch(colVatTuNhap, ThisItem, {NguoiNhap: txtNguoiNhap.Text})` |
| Nhà cung cấp  | `txtNhaCungCap`   | `ThisItem.NhaCungCap`         | `Patch(colVatTuNhap, ThisItem, {NhaCungCap: txtNhaCungCap.Text})` |
| Ghi chú       | `txtGhiChu`       | `ThisItem.GhiChu`             | `Patch(colVatTuNhap, ThisItem, {GhiChu: txtGhiChu.Text})` |
| Loại vật tư   | `drpLoaiVatTu`    | `ThisItem.LoaiVatTu`          | `Patch(colVatTuNhap, ThisItem, {LoaiVatTu: drpLoaiVatTu.Selected.Value})` |

---

## 4. 📥 Nút "Nhập" – Ghi nhận dữ liệu vào Dataverse

### `btnNhap.OnSelect`
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
            NgayNhap: NgayNhap,
            NguoiNhap: NguoiNhap,
            NhaCungCap: NhaCungCap,
            GhiChu: GhiChu,
            LoaiVatTu: LoaiVatTu
        }
    )
);
Clear(colVatTuNhap);
Notify("Đã ghi nhận nhập kho thành công!", NotificationType.Success)
```

---

## 5. 💡 Gợi ý UI/UX

- Mở rộng `galNhapKho` bằng cách set:
```powerapps
galNhapKho.X = galVatTu.X + galVatTu.Width + 10;
galNhapKho.Width = App.Width - galNhapKho.X - 20
```

- Định dạng `DonGia` đẹp hơn:
```powerapps
Text(ThisItem.DonGia, "[$-vi-VN]#,###")
```