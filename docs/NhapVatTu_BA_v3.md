
# 🧾 Nhập vật tư – Tài Liệu Business Analyst (BA)
**Version 3 – 2025-04-06**

## 1. 🎯 Mục đích
Xây dựng chức năng **Nhập vật tư** trong hệ thống **quản lý kho** sử dụng Power Apps + Dataverse. Mục tiêu:
- Cho phép nhập nhiều vật tư cùng lúc.
- Ghi nhận đầy đủ thông tin: số lượng, đơn giá, người nhập, ngày nhập,...
- Cập nhật số lượng tồn kho và giá bình quân (Giá trung bình gia quyền).
- Giao tiếp qua Flow `NhapVatTuFlow`.

---

## 2. 📊 Dữ liệu

### 🔹 Tables (Dataverse)
- `KhoVatTu`: Danh sách vật tư chính đang tồn kho.
- `KhoVatTu_NhapKho`: Lưu lịch sử nhập kho.

### 🔹 Collections trong Power Apps
- `colKhoVatTu`: Load từ `KhoVatTu`
- `colVatTuNhap`: Lưu danh sách vật tư được chọn để nhập, tạm thời.

---

## 3. 🖼 Giao diện: `scrNhapKho`

### 3.1 Bên trái – Danh sách vật tư

| Control | Tên | Ghi chú |
|--------|-----|---------|
| Gallery | `galVatTu` | Dữ liệu từ `colKhoVatTu` |
| Checkbox | `chkChonVatTuNhap` | Sự kiện chọn để thêm vào collection |

#### ✅ Sự kiện `OnCheck`:
```powerapps
Collect(
    colVatTuNhap,
    {
        MaVatTu: ThisItem.MaVatTu,
        TenVatTu: ThisItem.TenVatTu,
        DonViTinh: ThisItem.DonViTinh,
        LoaiVatTu: ThisItem.LoaiVatTu,
        SoLuongNhap: 0,
        DonGia: 0,
        NgayNhap: Blank(),
        NguoiNhap: "",
        NhaCungCap: "",
        GhiChu: ""
    }
)
```

---

### 3.2 Bên phải – Danh sách vật tư đã chọn (`galNhapKho`)

`Items = colVatTuNhap`

| Field         | Control        | PlaceholderText        | Patch khi thay đổi |
|---------------|----------------|------------------------|---------------------|
| Số lượng nhập | `txtSoLuong`   | `ThisItem.SoLuongNhap` | `Patch(colVatTuNhap, ThisItem, {SoLuongNhap: Value(Self.Text)})` |
| Đơn giá       | `txtDonGia`    | `ThisItem.DonGia`      | `Patch(colVatTuNhap, ThisItem, {DonGia: Value(Self.Text)})` |
| Ngày nhập     | `dpNgayNhap`   | `ThisItem.NgayNhap`    | `Patch(colVatTuNhap, ThisItem, {NgayNhap: dpNgayNhap.SelectedDate})` |
| Người nhập    | `txtNguoiNhap` | `ThisItem.NguoiNhap`   | `Patch(colVatTuNhap, ThisItem, {NguoiNhap: txtNguoiNhap.Text})` |
| Nhà cung cấp  | `txtNhaCC`     | `ThisItem.NhaCungCap`  | `Patch(colVatTuNhap, ThisItem, {NhaCungCap: txtNhaCC.Text})` |
| Ghi chú       | `txtGhiChu`    | `ThisItem.GhiChu`      | `Patch(colVatTuNhap, ThisItem, {GhiChu: txtGhiChu.Text})` |

📌 Không còn sử dụng `drpLoaiVatTu`, dữ liệu `LoaiVatTu` lấy từ `KhoVatTu`.

---

## 4. 📥 Button `btnNhap` – Ghi nhận và gọi Flow

### ✅ Công thức `OnSelect`
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
            SoLuongNhap: SoLuongNhap,
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

## 5. 🔁 Flow `NhapVatTuFlow`

- Flow được gọi sau khi nhập dữ liệu tạm vào `KhoVatTu_NhapKho`.
- Mục tiêu: cập nhật `KhoVatTu` với số lượng tồn mới và giá bình quân mới.

### 🧮 Cấu hình:

#### 5.1. `List rows` (lọc vật tư cần cập nhật)
```plaintext
concat('cr095_itemcode eq ''', triggerBody()?['text'], '''')
```

#### 5.2. `Update a row` – tính lại số lượng và giá bình quân

- **SoLuong**:
```plaintext
add(items('Apply_to_each')?['cr095_quantity'], int(triggerBody()['number']))
```

- **GiaBinhQuan (Average Price)**:
```plaintext
div(
    add(
        mul(
            coalesce(items('Apply_to_each')?['cr095_quantity'], 0),
            coalesce(items('Apply_to_each')?['cr095_averageprice'], 0)
        ),
        mul(
            int(triggerBody()?['number_1']),
            int(triggerBody()?['number'])
        )
    ),
    add(
        coalesce(items('Apply_to_each')?['cr095_quantity'], 0),
        int(triggerBody()?['number'])
    )
)
```

---

## 6. ✅ Kết luận

- Người dùng có thể nhập nhiều vật tư cùng lúc với thao tác dễ dàng.
- Hệ thống phân tách rõ phần giao diện và xử lý cập nhật qua Flow.
- Giá trị tồn kho luôn được cập nhật chính xác với giá bình quân gia quyền.
