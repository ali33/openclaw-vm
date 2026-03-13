# Nhật ký thay đổi

Mọi thay đổi đáng chú ý của dự án được ghi trong file này.

Định dạng dựa trên [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
và dự án tuân theo [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Chưa phát hành]

### Thêm mới

- Thiết lập dự án ban đầu cho VM sandbox tác nhân AI OpenClaw
- Ubuntu 22.04 LTS với môi trường desktop XFCE
- Cấu hình VirtualBox với GUI, ảo hóa lồng nhau
- Cài đặt tự động OpenClaw qua npm (Node.js 22)
- Ứng dụng desktop phổ biến: Google Chrome, VLC, LibreOffice
- Trình quản lý gói snapd
- Công cụ phát triển: Git, vim, nano, htop, curl, wget
- Phím tắt desktop cho thiết lập OpenClaw, bảng điều khiển và TUI
- Chuyển tiếp cổng cho bảng điều khiển gateway OpenClaw (18789)
- Tài liệu đầy đủ (README, CONTRIBUTING, troubleshooting, customization)
- GitHub Actions CI cho kiểm tra Vagrantfile và lint markdown
- Mẫu issue GitHub
- Giấy phép MIT

### Thay đổi

- Giữ base box Ubuntu 22.04 LTS (Canonical đã ngừng box Vagrant chính thức 24.04)
- Thay phụ thuộc build game bằng stack tác nhân AI OpenClaw (Node.js, npm, pnpm)
- Viết lại provisioning để cài ứng dụng desktop phổ biến và OpenClaw

### Loại bỏ

- Công cụ build C/C++ (build-essential, cmake, SDL2, v.v.)
- Tham chiếu tài sản game Captain Claw

---

## Cách cập nhật Changelog này

Khi có thay đổi:

1. Thêm mục vào phần `[Chưa phát hành]`
2. Dùng phân mục phù hợp (Thêm mới, Thay đổi, Sửa lỗi, v.v.)
3. Viết thay đổi theo góc nhìn người dùng
4. Ghi số issue/PR khi có

Khi phát hành phiên bản, chuyển mục từ `[Chưa phát hành]` sang phần phiên bản mới:

```markdown
## [1.0.0] - 2026-01-31

### Thêm mới
- Phát hành đầu tiên
```
