# Hướng dẫn tùy biến

Cách tùy biến môi trường VM OpenClaw theo nhu cầu của bạn.

## Mục lục

- [Tài nguyên VM](#tài-nguyên-vm)
- [Môi trường desktop](#môi-trường-desktop)
- [Phần mềm bổ sung](#phần-mềm-bổ-sung)
- [Thư mục dùng chung](#thư-mục-dùng-chung)
- [Cấu hình mạng](#cấu-hình-mạng)
- [Cấu hình âm thanh](#cấu-hình-âm-thanh)
- [Quản lý snapshot](#quản-lý-snapshot)
- [Nhiều VM](#nhiều-vm)
- [Biến môi trường](#biến-môi-trường)

---

## Tài nguyên VM

### Bộ nhớ và CPU

Sửa `Vagrantfile` để điều chỉnh tài nguyên cấp:

```ruby
config.vm.provider "virtualbox" do |vb|
  vb.memory = "8192"  # 8GB RAM
  vb.cpus = 4         # 4 nhân CPU
end
```

**Gợi ý**:
- Tối thiểu: 2GB RAM, 2 CPU
- Khuyến nghị: 4GB RAM, 2–4 CPU
- Cho tải tác nhân nặng: 8GB+ RAM, 4+ CPU

### Bộ nhớ video

```ruby
vb.customize ["modifyvm", :id, "--vram", "256"]  # 256MB
```

### Kích thước đĩa

Đĩa mặc định thường 64GB. Để đổi kích thước:

```bash
VBoxManage modifyhd "đường/dẫn/disk.vdi" --resize 102400  # 100GB
```

---

## Môi trường desktop

### Chuyển sang DE khác

Thay XFCE bằng môi trường desktop khác bằng cách sửa script provisioning:

#### GNOME (nặng hơn, nhiều tính năng)

```bash
apt-get install -y ubuntu-desktop gnome-shell
```

#### LXDE (nhẹ hơn XFCE)

```bash
apt-get install -y lxde lxdm
```

### Tắt tự đăng nhập

```bash
sudo rm /etc/lightdm/lightdm.conf.d/50-autologin.conf
sudo systemctl restart lightdm
```

---

## Phần mềm bổ sung

### Cài qua Snap

Sau provisioning, bạn có thể cài gói snap trong VM:

```bash
sudo snap install code --classic       # VS Code
sudo snap install firefox              # Firefox
sudo snap install telegram-desktop     # Telegram
sudo snap install discord              # Discord
sudo snap install slack                # Slack
```

### Cài VS Code qua provisioning

Thêm vào script provisioning trong Vagrantfile:

```bash
wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > packages.microsoft.gpg
install -D -o root -g root -m 644 packages.microsoft.gpg /etc/apt/keyrings/packages.microsoft.gpg
echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/packages.microsoft.gpg] https://packages.microsoft.com/repos/code stable main" > /etc/apt/sources.list.d/vscode.list
apt-get update
apt-get install -y code
```

### Cài Docker (cho chế độ sandbox OpenClaw)

OpenClaw hỗ trợ chạy phiên không-main trong sandbox Docker. Thêm vào provisioning:

```bash
curl -fsSL https://get.docker.com | sh
usermod -aG docker vagrant
```

Sau đó cấu hình `agents.defaults.sandbox.mode: "non-main"` trong cài đặt OpenClaw.

---

## Thư mục dùng chung

### Bật thư mục dùng chung

Chia sẻ thư mục giữa máy chủ và VM:

```ruby
config.vm.synced_folder "./shared", "/home/vagrant/shared"
```

Tạo thư mục trên máy chủ:

```bash
mkdir shared
```

### Tắt thư mục dùng chung mặc định

```ruby
config.vm.synced_folder ".", "/vagrant", disabled: true
```

### Thư mục dùng chung NFS (nhanh hơn, máy chủ Linux/macOS)

```ruby
config.vm.synced_folder "./shared", "/home/vagrant/shared",
  type: "nfs",
  nfs_udp: false
```

---

## Cấu hình mạng

### Chuyển tiếp cổng

Chuyển tiếp thêm cổng từ guest sang host:

```ruby
# Chuyển tiếp web server
config.vm.network "forwarded_port", guest: 3000, host: 3000

# Chuyển tiếp cổng SSH tùy chỉnh
config.vm.network "forwarded_port", guest: 22, host: 2222, id: "ssh"
```

### Mạng bridged

Kết nối VM trực tiếp vào mạng của bạn:

```ruby
config.vm.network "public_network"
```

### Mạng riêng

Tạo mạng chỉ máy chủ:

```ruby
config.vm.network "private_network", ip: "192.168.33.10"
```

---

## Cấu hình âm thanh

```ruby
config.vm.provider "virtualbox" do |vb|
  vb.customize ["modifyvm", :id, "--audio", "pulse"]        # Linux
  # vb.customize ["modifyvm", :id, "--audio", "coreaudio"]  # macOS
  # vb.customize ["modifyvm", :id, "--audio", "dsound"]     # Windows

  vb.customize ["modifyvm", :id, "--audioout", "on"]
  vb.customize ["modifyvm", :id, "--audioin", "on"]
end
```

---

## Quản lý snapshot

Snapshot cho phép lưu và khôi phục trạng thái VM:

```bash
# Lưu snapshot sau thiết lập ban đầu
vagrant snapshot save clean-state

# Liệt kê snapshot
vagrant snapshot list

# Khôi phục về snapshot (hoàn tác mọi thay đổi)
vagrant snapshot restore clean-state

# Xóa snapshot
vagrant snapshot delete clean-state
```

Hữu ích cho kiểm thử OpenClaw — chụp snapshot trước khi để tác nhân thực hiện tác vụ, sau đó khôi phục nếu có sự cố.

### Snapshot tự động khi khởi động

```ruby
config.trigger.after :up do |trigger|
  trigger.info = "Đang tạo snapshot sau khi khởi động..."
  trigger.run = {inline: "vagrant snapshot save post-boot"}
end
```

---

## Nhiều VM

Định nghĩa VM riêng cho mục đích khác nhau:

```ruby
Vagrant.configure("2") do |config|
  # VM desktop đầy đủ cho dùng OpenClaw tương tác
  config.vm.define "desktop" do |desktop|
    desktop.vm.box = "ubuntu/jammy64"
    desktop.vm.hostname = "openclaw-desktop"
    desktop.vm.provider "virtualbox" do |vb|
      vb.memory = "4096"
      vb.gui = true
    end
  end

  # VM headless cho kiểm thử chỉ gateway
  config.vm.define "headless" do |headless|
    headless.vm.box = "ubuntu/jammy64"
    headless.vm.hostname = "openclaw-headless"
    headless.vm.network "forwarded_port", guest: 18789, host: 18790
    headless.vm.provider "virtualbox" do |vb|
      vb.memory = "2048"
      vb.gui = false
    end
  end
end
```

Cách dùng:

```bash
vagrant up desktop
vagrant up headless
vagrant ssh headless
```

---

## Biến môi trường

Truyền biến môi trường vào provisioning:

```ruby
config.vm.provision "shell" do |s|
  s.env = {
    "OPENCLAW_VERSION" => "latest",
    "NODE_VERSION" => "22"
  }
  s.inline = "npm install -g openclaw@$OPENCLAW_VERSION"
end
```

---

## Mẹo

1. **Chụp snapshot** trước khi để OpenClaw thực hiện tác vụ phá hủy
2. **Kiểm thử từng bước** — dùng `vagrant provision` để chạy lại provisioning
3. **Dùng thư mục dùng chung** để đưa API key hoặc file cấu hình vào VM
4. **Tắt mạng** nếu muốn kiểm thử OpenClaw trong môi trường không kết nối
5. **Giữ Vagrantfile gọn** — chuyển provisioning phức tạp sang script riêng trong thư mục `scripts/`

---

Thêm thông tin:
- [Tài liệu Vagrant](https://www.vagrantup.com/docs)
- [Sổ tay VirtualBox](https://www.virtualbox.org/manual/)
- [Tài liệu OpenClaw](https://docs.openclaw.ai/)
