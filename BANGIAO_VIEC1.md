# BÀN GIAO — Việc 1 (Đăng nhập số điện thoại + họ tên + chọn cửa hàng) — 31/10/2026

## Bối cảnh (phiên sau không có memory)
Dự án: app KẾT CA NHÂN VIÊN của 9 Sạch (Google Apps Script `Code.gs` + `index.html` chạy Vercel, cầu nối `google.script.run` → fetch POST JSON tới Web App). Liên kết web VẬN HÀNH (idvh21-*.html + Code.gs vận hành, spreadsheet 1Uldgf0QcU3ZT3pKxoh-L8H7qeMASr7ENLt0ykv4046M). Phân vai: NHÂN VIÊN lập → QUẢN LÝ CỬA HÀNG duyệt → quản lý tổng xem lại. Prompt gốc 9 việc (Việc 0–9) do Gp giao; làm TỪNG VIỆC, xong dừng + bàn giao.

## Đã làm
- Việc 0 (rà soát): đã được Gp DUYỆT. Kết luận quan trọng:
  - Sheet vận hành: `Data_BanhSale` (bánh sale), `Data_XuatKhac` (Loai: Huy, MatBanh, TraTiktok, BSN, KhongBill; cột DaGhiNhanChiPhi). App NV HIỆN KHÔNG ghi vào 2 sheet này → khoảng trống số 1 (xử lý ở Việc 5/7).
  - `pushTienMat_` đang đẩy cả khoản chờ duyệt (chỉ bỏ TuChoi) → phải chỉ đẩy khoản đã DUYỆT (Việc 4/7). `nvQlDuyetCa` đang chặn duyệt ca khi còn khoản chưa duyệt → bỏ chặn (Việc 4). Dòng CHI/HOAN của khoản ứng chưa được đẩy (Việc 3).
- Việc 1 (làm xong, đã kiểm thử): file trong `ketca-nv/`.

## Việc 1 — thay đổi
Code.gs (APP_BUILD = 20261031a-v1):
- SCHEMAS.Users_V2 thêm cột cuối `HoTen` (tự thêm khi chạy; không đổi cột cũ).
- `nvLogin`: Username = SỐ ĐIỆN THOẠI. Chuẩn hóa `phoneNorm_` (bỏ khoảng trắng/dấu, +84/84→0, bù số 0 đầu bị Sheets làm mất). `sameLogin_`: số điện thoại so theo chuẩn hóa; QL/Admin dùng tên chữ thì so đúng chữ. Mật khẩu giữ nguyên cách lưu/so sánh cũ (chuỗi thường). Khóa 10 lần sai/10 phút giữ nguyên.
- Nhiều cửa hàng: cùng SĐT có thể là nhiều dòng Users_V2 (mỗi dòng 1 cửa hàng) HOẶC 1 dòng Stores cách nhau dấu phẩy; chỉ gộp các dòng đúng mật khẩu. Trả về `hoTen`, `myStores`, `needStore`.
- Hàm mới `nvChonCuaHang(token, store)` (đã thêm vào API_FNS_): chọn/đổi cửa hàng, chỉ trong danh sách được gán, ghi vào token (`saveAccount_`, giữ hạn phiên).
- `ctx_` (NhanVien): dùng `allowedStores_` + store đã chọn; chưa chọn → lỗi code `NEEDSTORE`.
- `nvBootstrap`: NV nhiều cửa hàng chưa chọn → trả `needStore:true`; luôn trả `hoTen`, `myStores`.
- Người thao tác: mọi chỗ ghi `c.account.username` (NguoiKet, DuyetBoi, TdDuyetBoi, UploadedBy, NguoiSua, who đẩy vận hành) nay ghi HỌ TÊN (`actor_`); nhật ký `AuditLog` ghi "Họ tên (SĐT)" (`actorFull_`). Chưa có HoTen thì dùng tên đăng nhập.
- `themNhanVienMau`: mẫu mới SĐT | mật khẩu | vai trò | cửa hàng | họ tên.

index.html:
- Ô đăng nhập "Số điện thoại" (bàn phím số); nút nhỏ "Tài khoản quản lý (không phải số điện thoại)" đổi sang bàn phím chữ.
- Sau đăng nhập hiện "Xin chào, <Họ tên>"; header: cửa hàng / họ tên.
- Màn chọn cửa hàng `#pick` (nút lớn) khi nhiều cửa hàng; tab Tài khoản có "Đổi cửa hàng" (chặn khi còn ca/nháp đang đồng bộ). Bản nháp lưu theo khóa có tên cửa hàng nên đổi cửa hàng không lẫn nháp.
- Bản lưu nhanh trên máy (BOOT_KEY) KHÔNG dùng cho tài khoản nhiều cửa hàng (tránh hiện nhầm cửa hàng).
- AUTH_KEY lưu thêm hoTen.

## Kiểm thử đã chạy
- Mô phỏng server (13 kịch bản đạt): đăng nhập +84/khoảng trắng; nhiều cửa hàng cần chọn; chọn sai cửa hàng bị từ chối; chọn đúng; mật khẩu sai/ dòng khác; QL tên chữ; bootstrap needStore; token hết hạn; khóa sau 10 lần sai.
- Giao diện (Playwright, dữ liệu mẫu): login → màn chọn → vào cửa hàng → tab Tài khoản → tải lại trang vẫn giữ đăng nhập.
- CHƯA chạy trên Apps Script/Sheet thật.

## Hướng dẫn kiểm thử trên dữ liệu thật
1. Dán `ketca-nv/Code.gs` vào project Apps Script KẾT CA NV, Deploy phiên bản mới (Web App). Dán `index.html` lên Vercel (chỗ đang chạy app NV).
2. Mở Users_V2, thêm cột HoTen (hoặc để tự thêm sau lần đăng nhập đầu). Nhập: Username=SĐT (định dạng văn bản), Password, Role=NhanVien, Stores=tên cửa hàng ĐÚNG như Store_Settings, HoTen.
3. Tạo thêm 1 dòng cùng SĐT + cùng mật khẩu, Stores = cửa hàng khác → đăng nhập phải thấy màn "Hôm nay bạn làm ở cửa hàng nào?".
4. Kết ca thử → Data_NV_KetCa.NguoiKet phải là HỌ TÊN; Data_AuditLog hiện "Họ tên (SĐT)".
5. Đăng nhập sai 10 lần phải bị khóa 10 phút.

## Rủi ro / lưu ý mở
- Tài khoản nhân viên cũ (Username chữ như `nv_demo`) vẫn đăng nhập được (so đúng chữ) nhưng không có HoTen cho tới khi điền cột HoTen.
- Hai dòng cùng SĐT khác mật khẩu: chỉ dòng đúng mật khẩu được tính.
- Giả định đã áp dụng: QL/Admin có thể vẫn dùng tên chữ; mật khẩu giữ nguyên dạng lưu hiện tại.

## Việc tiếp theo (theo thứ tự)
Việc 2 (Nộp tiền) → 3 (Ứng/Chi ứng/Hoàn ứng + sửa web vận hành loại HOAN) → 4 (Quản lý duyệt) → 5 (Bán/Tồn/Xuất khác) → 6 (Thu chi thẻ tình huống) → 7 (Đẩy dữ liệu) → 8 (Kết ca cuối ngày) → 9 (Nháp/Hoàn tác/Zalo/Tốc độ).
Phiên sau cần đính kèm: prompt gốc (attachment.txt), `ketca-nv/Code.gs`, `ketca-nv/index.html` (bản mới), `idvh21-*.html`, `Code.gs` vận hành (`codegsvh-*.md`), ảnh màn "Tiền mặt còn lại cần nộp" nếu có.
