# OpenClaw VM

Máy ảo Vagrant được cô lập để chạy và kiểm thử [OpenClaw](https://github.com/openclaw/openclaw) một cách an toàn — tác nhân AI tự trị mã nguồn mở (trước đây là Moltbot / Clawdbot).

![OpenClaw VM Desktop](screenshot.png)

## Tại sao dùng VM?

OpenClaw là tác nhân AI mạnh với quyền truy cập toàn hệ thống — shell, file, trình duyệt, nền tảng nhắn tin. Chạy trực tiếp trên máy cá nhân [gây lo ngại bảo mật](https://blogs.cisco.com/ai/personal-ai-agents-like-openclaw-are-a-security-nightmare). Dự án này cung cấp VM được cấu hình sẵn, cô lập để bạn có thể:

- **Kiểm thử an toàn** — OpenClaw chạy trong VM có thể hủy, không chạy trên máy chủ của bạn
- **Thử nghiệm thoải mái** — dùng thử khả năng của tác nhân mà không rủi ro cho dữ liệu
- **Xóa và tạo lại** — `vagrant destroy && vagrant up` để có môi trường sạch
- **Truy cập từ máy chủ** — Bảng điều khiển Gateway chuyển tiếp tới `localhost:18789` (cần token)
- **Tự khởi động** — Daemon Gateway khởi động tự động khi đăng nhập sau khi thiết lập

## Nội dung đi kèm

| Danh mục | Phần mềm |
|----------|----------|
| **HĐH** | Ubuntu 22.04 LTS với màn hình XFCE |
| **Tác nhân AI** | OpenClaw (mới nhất), Node.js 22, pnpm |
| **Trình duyệt** | Google Chrome |
| **Văn phòng** | LibreOffice |
| **Đa phương tiện** | VLC |
| **Công cụ phát triển** | Git, vim, nano, htop, curl, wget |
| **Quản lý gói** | snapd, apt |
| **Bổ sung** | Evince (PDF), Ristretto (hình ảnh), p7zip, system monitor |

## Yêu cầu

- [VirtualBox](https://www.virtualbox.org/wiki/Downloads) (7.0 trở lên)
- [Vagrant](https://www.vagrantup.com/downloads) (2.4 trở lên)

<details>
<summary>Windows</summary>

1. Tải và cài VirtualBox từ https://www.virtualbox.org/wiki/Downloads
2. Tải và cài Vagrant từ https://www.vagrantup.com/downloads
3. Khởi động lại hệ thống sau khi cài

</details>

<details>
<summary>macOS</summary>

```bash
brew install --cask virtualbox
brew install --cask vagrant
```

</details>

<details>
<summary>Linux</summary>

```bash
# Ubuntu/Debian
sudo apt-get update
sudo apt-get install virtualbox vagrant

# Fedora
sudo dnf install virtualbox vagrant

# Arch Linux
sudo pacman -S virtualbox vagrant
```

</details>

## Bắt đầu nhanh

```bash
# Clone repo này
git clone https://github.com/YOUR_USERNAME/openclaw-vm.git
cd openclaw-vm

# Khởi động VM (lần đầu sẽ tải box nền và cấu hình)
vagrant up

# Giao diện desktop XFCE mở tự động
# Đăng nhập: vagrant / vagrant
```

## Thiết lập OpenClaw

Sau khi desktop load xong:

1. **Chạy trình hướng dẫn onboarding** — double-click "OpenClaw Setup" trên desktop, hoặc mở terminal và chạy:

   ```bash
   openclaw onboard --install-daemon
   ```

   Quá trình này sẽ hướng dẫn:
   - Chọn nhà cung cấp AI (Anthropic, OpenAI, v.v.) và nhập API key
   - Cấu hình kênh nhắn tin (Telegram, Discord, WhatsApp, v.v.)
   - Cài daemon gateway dưới dạng systemd user service

2. **Lấy gateway token** — bảng điều khiển cần token để xác thực. Tìm token:

   ```bash
   cat ~/.openclaw/openclaw.json | grep -A 2 '"auth"'
   ```

   Tìm giá trị `"token"` (chuỗi hex dài).

3. **Truy cập bảng điều khiển** — mở Chrome và truy cập:

   ```
   http://127.0.0.1:18789/?token=YOUR_TOKEN_HERE
   ```

   Thay `YOUR_TOKEN_HERE` bằng token từ bước 2.

   Bảng điều khiển cũng truy cập được từ **máy chủ** của bạn tại:
   ```
   http://localhost:18789/?token=YOUR_TOKEN_HERE
   ```

4. **Dùng TUI** — double-click "OpenClaw TUI" hoặc chạy `openclaw` trong terminal.

### Gateway tự khởi động

Sau onboarding, daemon gateway tự khởi động mỗi lần đăng nhập qua systemd user service. Kiểm tra trạng thái:

```bash
systemctl --user status openclaw-gateway
```

Điều khiển thủ công:
```bash
systemctl --user start openclaw-gateway    # Khởi động
systemctl --user stop openclaw-gateway     # Dừng
systemctl --user restart openclaw-gateway  # Khởi động lại
```

## Quản lý VM

```bash
vagrant up        # Khởi động VM
vagrant halt      # Tắt VM
vagrant reload    # Khởi động lại VM
vagrant ssh       # SSH vào VM (headless)
vagrant destroy   # Xóa VM hoàn toàn
vagrant provision # Chạy lại provisioning trên VM hiện có
```

## Cấu hình

Thông số VM mặc định (sửa `Vagrantfile` để thay đổi):

| Thiết lập | Mặc định |
|---------|---------|
| RAM | 4 GB |
| CPU | 2 nhân |
| Video RAM | 128 MB |
| Tăng tốc 3D | Bật |
| Clipboard | Hai chiều |
| Kéo thả | Hai chiều |
| Ảo hóa lồng nhau | Bật |

## Xử lý sự cố

Xem [docs/troubleshooting.md](docs/troubleshooting.md) để biết cách xử lý chi tiết. Sửa nhanh:

<details>
<summary>Cửa sổ GUI không hiện</summary>

Đảm bảo `vb.gui = true` trong Vagrantfile. Kiểm tra đã cài VirtualBox Extension Pack.

</details>

<details>
<summary>Bảng điều khiển báo "unauthorized: gateway token missing"</summary>

Bảng điều khiển cần token để xác thực. Tìm token:

```bash
vagrant ssh
cat ~/.openclaw/openclaw.json | grep '"token"'
```

Sau đó truy cập bảng điều khiển với token trong URL:
```
http://localhost:18789/?token=YOUR_TOKEN_HERE
```

</details>

<details>
<summary>Biểu tượng OpenClaw TUI mở rồi đóng ngay</summary>

Launcher trên desktop đã được cập nhật để giữ terminal mở. Nếu bạn dùng bản VM cũ, sửa thủ công:

```bash
cat > ~/Desktop/openclaw-tui.desktop <<'EOF'
[Desktop Entry]
Version=1.0
Type=Application
Name=OpenClaw TUI
Comment=Khởi chạy OpenClaw Terminal Interface
Exec=xfce4-terminal -e "bash -c 'openclaw || bash'"
Icon=utilities-terminal
Terminal=false
Categories=Development;
EOF

chmod +x ~/Desktop/openclaw-tui.desktop
```

</details>

<details>
<summary>Không tìm thấy lệnh OpenClaw</summary>

SSH vào VM và cài lại:
```bash
vagrant ssh
sudo npm install -g openclaw@latest
```

</details>

<details>
<summary>Cổng 18789 đã được dùng trên máy chủ</summary>

Vagrant tự điều chỉnh xung đột cổng. Chạy `vagrant port` để xem ánh xạ thực tế, hoặc dừng dịch vụ đang dùng cổng 18789 trên máy chủ.

</details>

<details>
<summary>Gateway không chạy sau khi khởi động lại</summary>

Gateway chạy dưới user service và khởi động khi đăng nhập. Kiểm tra trạng thái:
```bash
vagrant ssh
systemctl --user status openclaw-gateway
```

Nếu không chạy, có thể cần hoàn thành onboarding trước (`openclaw onboard --install-daemon`).

</details>

<details>
<summary>VM chạy chậm</summary>

Tăng tài nguyên trong Vagrantfile:
```ruby
vb.memory = "8192"  # 8GB RAM
vb.cpus = 4         # 4 nhân CPU
```

</details>

## Đóng góp

Mọi đóng góp đều được chào đón. Xem [CONTRIBUTING.md](CONTRIBUTING.md) để biết hướng dẫn.

## Giấy phép

MIT License — xem [LICENSE](LICENSE) để biết chi tiết.

## Liên kết

- [OpenClaw](https://github.com/openclaw/openclaw) — Repo OpenClaw chính
- [OpenClaw Docs](https://docs.openclaw.ai/) — Tài liệu chính thức
- [openclaw.ai](https://openclaw.ai/) — Trang web dự án
- [Vagrant](https://www.vagrantup.com/) — Tự động hóa VM
- [VirtualBox](https://www.virtualbox.org/) — Nền tảng ảo hóa
