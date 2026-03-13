# CLAUDE.md

## Dự án: openclaw-vm

VM dựa trên Vagrant để chạy và kiểm thử **OpenClaw** an toàn (tác nhân AI tự trị mã nguồn mở, trước đây là Moltbot/Clawdbot của Peter Steinberger). Đây KHÔNG phải bản triển khai lại game Captain Claw — ban đầu được tạo khung theo hướng đó nhưng đã được viết lại hoàn toàn.

## OpenClaw là gì

- Tác nhân AI tự lưu trữ mã nguồn mở: https://github.com/openclaw/openclaw
- Cài qua `npm install -g openclaw@latest` (cần Node >= 22)
- Daemon gateway chạy trên cổng 18789 (bảng điều khiển web)
- Trình hướng dẫn onboarding: `openclaw onboard --install-daemon`
- Tài liệu: https://docs.openclaw.ai/
- Có quyền truy cập toàn hệ thống (shell, file, trình duyệt, nhắn tin) — do đó cần cô lập bằng VM

## Cấu trúc dự án

```
Vagrantfile              # Cấu hình VM: Ubuntu 22.04 + XFCE + OpenClaw + ứng dụng desktop
README.md                # Tài liệu hướng người dùng
CONTRIBUTING.md          # Hướng dẫn đóng góp
CHANGELOG.md             # Theo Keep a Changelog
LICENSE                  # MIT
docs/customization.md    # Tùy biến VM nâng cao (tài nguyên, Docker, snapshot, đa VM)
docs/troubleshooting.md  # Sự cố thường gặp (provisioning, GUI, OpenClaw, VirtualBox, mạng)
.github/workflows/vagrant-validate.yml  # CI: vagrant validate + cú pháp ruby + lint markdown
.github/ISSUE_TEMPLATE/  # Mẫu báo lỗi + đề xuất tính năng
```

## VM provisioning (Vagrantfile)

Provisioning 8 giai đoạn:
1. Cập nhật hệ thống
2. Desktop XFCE + LightDM (tự đăng nhập với vagrant)
3. VirtualBox Guest Additions
4. Ứng dụng phổ biến: git, curl, wget, vim, nano, htop, LibreOffice, VLC, plugin Thunar, Evince, Ristretto, gnome-system-monitor
5. Google Chrome (tải .deb)
6. snapd
7. Node.js 22 (NodeSource) + pnpm + OpenClaw
8. Cấu hình desktop: 3 phím tắt (OpenClaw Setup, Dashboard, TUI) + README.txt

Chuyển tiếp cổng: guest 18789 -> host 18789 (bảng điều khiển gateway OpenClaw)
Ảo hóa lồng nhau được bật.

## Trạng thái hiện tại

- Commit đầu có khung hướng game cũ
- Mọi file đã được viết lại cho tác nhân AI OpenClaw nhưng thay đổi chưa commit
- Chưa cấu hình remote repository
- Vagrantfile pass `ruby -c` và `vagrant validate`
- Đã kiểm thử thành công với `vagrant up` — cả 8 giai đoạn provisioning đều pass
- Đã xác nhận trong VM: Node.js 22.22.0, OpenClaw 2026.1.29, Chrome, VLC, LibreOffice đều có

## Vấn đề / lưu ý đã biết

- `ubuntu/noble64` (24.04) KHÔNG tồn tại trên Vagrant Cloud — Canonical đã ngừng box chính thức do HashiCorp BSL
- `virtualbox-guest-dkms` không có trong repo jammy — dùng `virtualbox-guest-utils virtualbox-guest-x11` với `|| true`
- KVM và VirtualBox xung đột: nếu máy chủ đã load `kvm_amd`/`kvm_intel`, VirtualBox sẽ lỗi `VERR_SVM_IN_USE`. Sửa: `sudo modprobe -r kvm_amd kvm`
- Cảnh báo lệch phiên bản Guest Additions (6.0.0 vs 7.0) không ảnh hưởng — clipboard/kéo-thả vẫn hoạt động
- LightDM `systemctl enable` hiện cảnh báo về unit file không có cấu hình install — chỉ là giao diện, tự đăng nhập vẫn ổn

## Quyết định chính

- Base box: `ubuntu/jammy64` (22.04 LTS) — Canonical đã ngừng box Vagrant chính thức 24.04 do HashiCorp BSL
- Desktop: XFCE (nhẹ)
- OpenClaw cài toàn cục qua npm (không Docker, không từ source)
- Người dùng chạy trình hướng dẫn onboarding thủ công sau lần khởi động đầu (cần API key)
- Chrome cài qua tải .deb trực tiếp (không snap) để ổn định

## Lệnh

```bash
ruby -c Vagrantfile      # Kiểm tra cú pháp Vagrantfile
vagrant validate         # Kiểm tra Vagrant đầy đủ (cần cài Vagrant)
vagrant up               # Build và khởi động VM
vagrant destroy -f       # Xóa VM
vagrant provision        # Chạy lại provisioning trên VM hiện có
vagrant ssh              # SSH vào VM đang chạy
```
