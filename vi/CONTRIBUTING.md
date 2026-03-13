# Đóng góp cho OpenClaw VM

Cảm ơn bạn đã cân nhắc đóng góp cho OpenClaw VM. Dự án cung cấp môi trường Vagrant cô lập để chạy tác nhân AI OpenClaw một cách an toàn.

## Quy tắc ứng xử

Dự án và mọi người tham gia tuân theo cam kết tạo môi trường chào đón và hòa nhập. Khi tham gia, bạn cần duy trì chuẩn mực này.

## Tôi có thể đóng góp thế nào?

### Báo lỗi

Trước khi tạo báo cáo lỗi, hãy kiểm tra issue hiện có để tránh trùng lặp. Khi gửi báo cáo lỗi, cần có:

- Mô tả lỗi
- Các bước tái hiện
- Hành vi mong đợi
- HĐH máy chủ, phiên bản VirtualBox, phiên bản Vagrant
- Log lỗi (`vagrant up --debug > vagrant.log 2>&1`)

### Đề xuất cải tiến

Đề xuất cải tiến được theo dõi qua GitHub issues. Cần có:

- Tiêu đề rõ ràng, mô tả cụ thể
- Mô tả chi tiết đề xuất cải tiến
- Lý do cải tiến này hữu ích
- Các giải pháp thay thế bạn đã cân nhắc

### Pull Request

1. Fork repo và tạo nhánh từ `main`
2. Thực hiện thay đổi với commit message rõ ràng, mô tả
3. Kiểm thử thay đổi — đảm bảo VM provision đúng
4. Cập nhật tài liệu nếu thay đổi chức năng
5. Gửi pull request kèm mô tả rõ ràng

#### Checklist kiểm thử

Trước khi gửi PR, xác nhận:

- [ ] VM provision thành công (`vagrant up`)
- [ ] GUI hiển thị đúng sau khi khởi động lại
- [ ] OpenClaw cài đặt và có trên PATH
- [ ] Ứng dụng thường dùng (Chrome, VLC, LibreOffice) chạy được
- [ ] Chuyển tiếp cổng hoạt động (bảng điều khiển gateway tại host:18789)
- [ ] Phím tắt desktop hoạt động đúng
- [ ] Tài liệu đã được cập nhật
- [ ] Không commit file không cần thiết

#### Hướng dẫn commit message

- Dùng thì hiện tại ("Thêm tính năng" thay vì "Đã thêm tính năng")
- Dùng thể mệnh lệnh ("Di chuyển con trỏ tới..." thay vì "Di chuyển con trỏ tới...")
- Giới hạn dòng đầu 72 ký tự
- Tham chiếu issue và pull request khi liên quan

### Thiết lập phát triển

```bash
# Fork và clone repo
git clone https://github.com/YOUR_USERNAME/openclaw-vm.git
cd openclaw-vm

# Tạo nhánh tính năng
git checkout -b feature/ten-tinh-nang-cua-ban

# Kiểm thử thay đổi
vagrant up
# ... kiểm thử VM ...
vagrant destroy

# Commit và push
git add .
git commit -m "Thêm tính năng của bạn"
git push origin feature/ten-tinh-nang-cua-ban
```

### Cấu trúc dự án

```
openclaw-vm/
├── Vagrantfile              # Cấu hình VM và provisioning chính
├── README.md                # Tài liệu dự án
├── CONTRIBUTING.md          # File này
├── CHANGELOG.md             # Lịch sử phiên bản
├── LICENSE                  # Giấy phép MIT
├── .gitignore               # Quy tắc git ignore
├── .markdownlint.json       # Cấu hình lint markdown
├── .markdown-link-check.json # Cấu hình kiểm tra link
├── docs/
│   ├── customization.md     # Hướng dẫn tùy biến VM
│   └── troubleshooting.md   # Hướng dẫn xử lý sự cố
└── .github/
    ├── workflows/
    │   └── vagrant-validate.yml  # Pipeline CI
    └── ISSUE_TEMPLATE/
        ├── bug_report.md
        └── feature_request.md
```

## Hướng dẫn phong cách

### Phong cách Vagrantfile

- Dùng 2 khoảng trắng cho thụt lề
- Thêm comment cho cấu hình không hiển nhiên
- Tổ chức script provisioning theo các giai đoạn đánh số
- Dùng biến cho giá trị có thể cấu hình

### Phong cách tài liệu

- Dùng Markdown cho mọi tài liệu
- Dùng ngôn ngữ rõ ràng, súc tích
- Thêm ví dụ code khi hữu ích

## Tài liệu bổ sung

- [Tài liệu Vagrant](https://www.vagrantup.com/docs)
- [Sổ tay VirtualBox](https://www.virtualbox.org/manual/)
- [Dự án OpenClaw](https://github.com/openclaw/openclaw)
- [Tài liệu OpenClaw](https://docs.openclaw.ai/)

## Có câu hỏi?

Mở issue với nhãn `question`.
