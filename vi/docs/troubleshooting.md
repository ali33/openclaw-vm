# Hướng dẫn xử lý sự cố

Các sự cố thường gặp và cách xử lý cho môi trường VM OpenClaw.

## Mục lục

- [Sự cố provisioning](#sự-cố-provisioning)
- [Sự cố GUI](#sự-cố-gui)
- [Sự cố OpenClaw](#sự-cố-openclaw)
- [Sự cố hiệu năng](#sự-cố-hiệu-năng)
- [Sự cố VirtualBox](#sự-cố-virtualbox)
- [Sự cố mạng](#sự-cố-mạng)

---

## Sự cố provisioning

### VM provisioning thất bại

**Triệu chứng**: `vagrant up` thất bại trong lúc provisioning

**Cách xử lý**:

1. Kiểm tra kết nối internet — provisioning cần tải gói
2. Thử provisioning lại từ đầu:
   ```bash
   vagrant destroy -f
   vagrant up
   ```
3. Kiểm tra log VirtualBox:
   ```bash
   VBoxManage list vms
   VBoxManage showvminfo "OpenClaw VM"
   ```

### Cài đặt gói thất bại

**Triệu chứng**: Lỗi APT khi provisioning

**Cách xử lý**:

1. SSH vào VM và cập nhật thủ công:
   ```bash
   vagrant ssh
   sudo apt-get update
   sudo apt-get upgrade
   ```
2. Kiểm tra trạng thái mirror Ubuntu và thử mirror khác

### Cài đặt Node.js thất bại

**Triệu chứng**: Lỗi script thiết lập NodeSource

**Cách xử lý**:

1. SSH vào và cài thủ công:
   ```bash
   vagrant ssh
   curl -fsSL https://deb.nodesource.com/setup_22.x | sudo bash -
   sudo apt-get install -y nodejs
   ```
2. Kiểm tra phiên bản Node: `node --version` (cần >= 22)

---

## Sự cố GUI

### Cửa sổ GUI không hiện

**Cách xử lý**:

1. Kiểm tra GUI đã bật trong Vagrantfile:
   ```ruby
   vb.gui = true
   ```
2. Kiểm tra VM có đang chạy:
   ```bash
   vagrant status
   VBoxManage list runningvms
   ```
3. Khởi động lại display manager:
   ```bash
   vagrant ssh
   sudo systemctl restart lightdm
   ```

### Màn hình đen khi khởi động

**Cách xử lý**:

1. Tắt tăng tốc 3D trong Vagrantfile:
   ```ruby
   # Comment dòng này:
   # vb.customize ["modifyvm", :id, "--accelerate3d", "on"]
   ```
2. Đổi bộ điều khiển đồ họa:
   ```ruby
   vb.customize ["modifyvm", :id, "--graphicscontroller", "vboxvga"]
   ```
3. Giảm bộ nhớ video:
   ```ruby
   vb.customize ["modifyvm", :id, "--vram", "64"]
   ```

### Clipboard không hoạt động

**Cách xử lý**:

1. Đảm bảo đã cài Guest Additions:
   ```bash
   vagrant ssh
   lsmod | grep vbox
   ```
2. Khởi động lại dịch vụ VirtualBox:
   ```bash
   sudo systemctl restart vboxadd-service
   ```

---

## Sự cố OpenClaw

### Không tìm thấy lệnh `openclaw`

**Cách xử lý**:

1. Cài lại toàn cục:
   ```bash
   sudo npm install -g openclaw@latest
   ```
2. Kiểm tra npm global bin có trong PATH:
   ```bash
   npm config get prefix
   echo $PATH
   ```

### OpenClaw gateway không khởi động

**Cách xử lý**:

1. Kiểm tra daemon đã cài:
   ```bash
   systemctl --user status openclaw-gateway
   ```
2. Chạy lại onboarding:
   ```bash
   openclaw onboard --install-daemon
   ```
3. Khởi động thủ công:
   ```bash
   openclaw gateway
   ```

### Không truy cập được Dashboard từ máy chủ

**Triệu chứng**: `http://localhost:18789/` không load trên máy chủ

**Cách xử lý**:

1. Kiểm tra chuyển tiếp cổng đang bật:
   ```bash
   vagrant port
   ```
2. Xác nhận gateway đang chạy trong VM:
   ```bash
   vagrant ssh
   curl http://127.0.0.1:18789/
   ```
3. Kiểm tra cổng có bị tự động đổi do xung đột — `vagrant port` hiển thị ánh xạ thực tế

### OpenClaw onboarding thất bại

**Cách xử lý**:

1. Kiểm tra phiên bản Node.js:
   ```bash
   node --version  # Phải >= 22
   ```
2. Kiểm tra kết nối mạng (cần cho xác thực API key):
   ```bash
   curl -s https://api.openai.com > /dev/null && echo "OK" || echo "FAIL"
   ```
3. Thử chạy với output chi tiết:
   ```bash
   DEBUG=* openclaw onboard --install-daemon
   ```

---

## Sự cố hiệu năng

### VM chậm/giật

**Cách xử lý**:

1. Tăng tài nguyên cấp trong Vagrantfile:
   ```ruby
   vb.memory = "8192"  # 8GB
   vb.cpus = 4
   ```
2. Đóng ứng dụng khác trên máy chủ
3. Tắt dịch vụ không cần trong VM:
   ```bash
   vagrant ssh
   sudo systemctl disable bluetooth
   sudo systemctl disable cups
   ```

### CPU sử dụng cao

**Cách xử lý**:

1. Tắt tăng tốc 3D nếu không cần
2. Kiểm tra tiến trình chiếm tài nguyên:
   ```bash
   vagrant ssh
   htop
   ```
3. Gateway OpenClaw có thể tốn CPU khi có tác vụ — điều này là bình thường

### I/O đĩa chậm

**Cách xử lý**:

1. Dùng SSD trên máy chủ nếu có thể
2. Bật host I/O cache:
   ```ruby
   vb.customize ["storagectl", :id, "--name", "SCSI", "--hostiocache", "on"]
   ```

---

## Sự cố VirtualBox

### VT-x/AMD-V không khả dụng

**Triệu chứng**: VM không khởi động với lỗi ảo hóa

**Cách xử lý**:

1. Bật ảo hóa trong BIOS/UEFI:
   - Khởi động lại máy
   - Vào BIOS (thường F2, F10 hoặc DEL)
   - Bật VT-x (Intel) hoặc AMD-V (AMD)
   - Lưu và thoát

2. Trên Windows, tắt Hyper-V:
   ```powershell
   # Chạy với quyền Administrator
   bcdedit /set hypervisorlaunchtype off
   ```
   Cần khởi động lại.

### Chưa cài kernel driver

**Triệu chứng**: VirtualBox không khởi động được VM

**Cách xử lý**:

1. Trên Linux, cài lại module kernel VirtualBox:
   ```bash
   sudo /sbin/vboxconfig
   ```
2. Trên macOS, cho phép kernel extension trong Cài đặt Bảo mật & Quyền riêng tư

---

## Sự cố mạng

### VM không truy cập được internet

**Cách xử lý**:

1. Kiểm tra mạng NAT đã bật (mặc định)
2. Khởi động lại mạng:
   ```bash
   vagrant ssh
   sudo systemctl restart NetworkManager
   ```
3. Thử chế độ mạng khác trong Vagrantfile:
   ```ruby
   config.vm.network "public_network"
   ```

---

## Cần thêm trợ giúp

Nếu các cách trên không giải quyết được:

1. Xem [GitHub Issues](https://github.com/YOUR_USERNAME/openclaw-vm/issues) hiện có
2. Tạo issue mới với:
   - Mô tả chi tiết
   - Các bước tái hiện
   - Log lỗi
   - Thông tin hệ thống
3. Xem [tài liệu OpenClaw](https://docs.openclaw.ai/) cho sự cố liên quan tác nhân

### Thu thập thông tin debug

```bash
# Thông tin hệ thống
vagrant --version
VBoxManage --version
uname -a

# Trạng thái Vagrant
vagrant status

# Log chi tiết
vagrant up --debug > vagrant-debug.log 2>&1

# Trong VM
node --version
openclaw --version
systemctl --user status openclaw-gateway
```
