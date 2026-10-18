# CR7 — File phát hiện gian lận thi cử trên máy tính

Một file `.bat` duy nhất (không cần cài gì thêm). Chạy xong nó quét máy và xuất **bảng chứng** kèm file Word, có mã SHA-256 niêm phong để không ai sửa được kết quả.

## Tải file

Mở trang: <https://robloxhihi.github.io/cr7-gian-lan/>

Trang yêu cầu **nhập key** mới tải được `cr7.bat`. Key không nằm trong repo này — ai cần thì hỏi trực tiếp.

File được mã hoá **AES-256-GCM**, key suy ra bằng **PBKDF2-SHA256 250.000 vòng**. Việc giải mã chạy ngay trong trình duyệt (WebCrypto), không gọi server nào cả. Sai key thì khối mã không giải ra gì, nên không có file để tải — không có kiểu "ẩn nút tải" qua mặt nhau được.

## Nó lấy được những gì

| Nhóm | Nguồn dữ liệu |
|---|---|
| Lệnh terminal đã gõ / đã chạy | UserAssist (có giờ), `ConsoleHost_history.txt` PSReadLine, Run MRU, event log PowerShell 4103/4104, Security 4688, Windows Terminal settings |
| USB & thiết bị | Registry `Enum\USB`, `USBSTOR`, `DeviceClasses`, event log DriverFrameworks-UserMode + Kernel-PnP, `Win32_LogicalDiskToStorage` |
| Wi-Fi & mạng | `netsh wlan show all`, event log WLAN-AutoConfig + NetworkProfile, registry NetworkList (SSID + giờ tạo / giờ kết nối cuối) |
| Web & DNS | Bản sao History/Favicons/Cookies của Chrome, Edge, Cốc Cốc + `ipconfig /displaydns` |
| Thay đổi tệp | Toàn bộ thư mục có `CreationTime` hoặc `LastWriteTime` rơi vào khung giờ khảo sát, mụcRecently mở trong `Recent` |
| Niêm phong | `Checksum_NiemPhong_*.csv` ghi SHA-256 từng file bằng chứng |

## Cách chạy

```
cr7.bat                       -> hôm nay, cả ngày
cr7.bat 18/10                 -> ngày 18/10, cả ngày
cr7.bat 18/10 0800-1700       -> chỉ 08:00 - 17:00
cr7.bat hom qua 8-17          -> hôm qua, 08:00 - 17:00
cr7.bat 18/10 full            -> quét cả C:\Windows (cham hon nhieu lan)
```

Nháy đúp không tham số thì nó **hỏi 2 câu**: ngày khảo sát, rồi khung giờ (Enter trắng = quét cả ngày). Khung giờ chấp nhận `0800-1700`, `08:00-17:00`, `8-17`, `8h-17h`, `8 gio - 17 gio`. Gõ `22-06` được hiểu là 22:00 hôm trước đến 06:00 hôm sau.

Kết quả lưu ở gốc ổ chứa file `.bat`, ví dụ `C:\BangChung_18_10_2026_0800h-1700h_<TEN_MAY>\`.

## Ghi chú thật thà

- Chạy với quyền **Administrator**; thiếu quyền thì một số event log và khoá registry sẽ trống.
- `ConsoleHost_history.txt` **không lưu giờ từng dòng lệnh**, chỉ lưu nội dung + giờ sửa file. Bằng chứng có giờ chính xác là UserAssist, event log PowerShell và Security 4688.
- Muốn Security 4688 ghi tên lệnh thì bật audit trước: `auditpol /set /subcategory:"Process Creation" /success:enable`.
- Chế độ nhanh bỏ qua `C:\Windows`, `WinSxS`, `WindowsApps`, junction — khoảng 40–100 giây cho một ngày.
- File chỉ đọc máy đang chạy và ghi kết quả ra máy đó, không gửi dữ liệu đi đâu.
