# AppSheet Print System v2
### Bộ công cụ in báo cáo từ AppSheet — không cần server, hoàn toàn miễn phí

---

## Cấu trúc (3 file)

```
Code.gs           → Google Apps Script (API backend)
admin-config.html → Admin tạo & quản lý mẫu báo cáo
print.html        → Trang in cho người dùng cuối
```

---

## BƯỚC 1 — Apps Script

1. Mở Google Sheet → **Extensions → Apps Script**
2. Xoá code cũ → dán toàn bộ `Code.gs`
3. Sửa **1 dòng duy nhất**:
```javascript
SPREADSHEET_ID: "ID_CỦA_GOOGLE_SHEET"
// Lấy từ URL: https://docs.google.com/spreadsheets/d/[ID_Ở_ĐÂY]/edit
```
4. **Deploy → New deployment**
   - Type: Web app
   - Execute as: **Me**
   - Who has access: **Anyone**
5. Copy URL → dán vào `admin-config.html` và `print.html`

> Sau mỗi lần sửa Code.gs → phải Deploy lại (New deployment hoặc Edit version)

---

## BƯỚC 2 — Host 2 file HTML (chọn 1)

| Cách | Tốc độ setup | URL |
|------|-------------|-----|
| **GitHub Pages** | 5 phút | `username.github.io/repo/print.html` |
| **Cloudflare Pages** | 3 phút | `project.pages.dev/print.html` |
| **Google Sites** | 2 phút | Embed trực tiếp vào trang Sites |

---

## BƯỚC 3 — Khai báo URL (2 dòng trong admin, 1 dòng trong print)

**admin-config.html** — tìm và sửa:
```javascript
const API_URL       = "https://script.google.com/macros/s/xxxx/exec";
const PRINT_APP_URL = "https://username.github.io/repo/print.html";
```

**print.html** — tìm và sửa:
```javascript
const API_URL = "https://script.google.com/macros/s/xxxx/exec";
```

---

## BƯỚC 4 — Tạo mẫu báo cáo (Admin)

Mở `admin-config.html` → điền form:

| Trường | Ví dụ |
|--------|-------|
| Tên mẫu | Hóa đơn bán hàng |
| ID mẫu | `hoa-don` |
| Sheet | `DonHang` (chọn từ danh sách) |
| Cột | Tick + kéo thả sắp thứ tự |
| Loại | Hóa đơn / Bảng / Phiếu kho / Tổng hợp |

Nhấn **Lưu** → config ghi vào sheet `_PrintConfig` trong Google Sheet.

---

## BƯỚC 5 — Action trong AppSheet

1. AppSheet → chọn Table → **Actions → Add**
2. Cấu hình:
```
Action name : In hóa đơn
For a record: [table]
Do this     : Open a link
URL         : [copy từ ô "URL AppSheet Action" trong admin]
```
Ví dụ URL:
```
https://username.github.io/repo/print.html?id=[_UNIQUEID]&tpl=hoa-don&sheet=DonHang
```

---

## Tham số URL đầy đủ

| Param | Bắt buộc | Mô tả |
|-------|----------|-------|
| `id` | ✓ | `[_UNIQUEID]` — AppSheet tự điền |
| `sheet` | ✓ | Tên sheet Google Sheets |
| `tpl` | Nên có | ID template → tự load config |
| `cols` | Tùy | Ghi đè danh sách cột |
| `preview=1` | — | Dùng data demo, không cần ID thật |
| `autoprint=1` | — | Tự mở hộp thoại in khi tải xong |

---

## Báo cáo tổng hợp nhiều sheet (summary)

Trong admin, chọn loại **"Tổng hợp nhiều sheet"** → thêm "Sheet dữ liệu phụ":

```
Sheet phụ: KhachHang  |  Join theo: MaKhachHang  |  Alias: customer
Sheet phụ: SanPham    |  Join theo: MaDonHang     |  Alias: items
```

Print App sẽ:
1. Lấy dòng chính từ sheet DonHang
2. Join tự động với KhachHang theo MaKhachHang
3. Lấy tất cả dòng SanPham có MaDonHang khớp
4. Render thành báo cáo gộp nhiều section

---

## Cột tính toán (Custom Fields)

Trong admin → "Cột tính toán":
```
Tên cột : Thành tiền
Công thức: {SoLuong} * {DonGia}

Tên cột : VAT 10%
Công thức: {ThanhTien} * 0.1
```
Kết quả tự tính khi render, không cần cột này có trong Google Sheet.

---

## Nhiều ứng dụng AppSheet dùng chung

Mỗi app chỉ cần:
- Cùng `API_URL` (1 Apps Script)
- Cùng `print.html`
- Tạo template riêng với `tplId` khác nhau

**Không cần deploy lại, không cần code mới.**

---

## Lưu trữ config

| Nơi | Khi nào |
|-----|---------|
| **localStorage** | Tải tức thì khi mở admin (offline-capable) |
| **Google Sheets `_PrintConfig`** | Master — đồng bộ mỗi khi mở admin |

Nhấn **"Đồng bộ từ Sheets"** để pull bản mới nhất về.

---

## Troubleshooting

| Lỗi | Nguyên nhân | Cách sửa |
|-----|------------|----------|
| "CORS" hoặc không kết nối | Apps Script chưa deploy đúng | Deploy lại, chọn Anyone |
| Không thấy sheet | SPREADSHEET_ID sai | Kiểm tra lại ID |
| Cột không hiện | Hàng 1 không phải header | Đảm bảo row 1 là tên cột |
| Trang trắng khi in | Lỗi JS | Mở `?preview=1` → F12 → Console |
| "Không tìm thấy dòng" | ID không khớp | Kiểm tra keyCol hoặc để trống |
