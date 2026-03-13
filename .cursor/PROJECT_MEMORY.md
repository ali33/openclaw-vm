# Memory dự án OpenClaw VM

File này lưu ngữ cảnh dự án để lần sau tiếp tục làm việc.

## Tổng quan

- **Dự án:** openclaw-vm — Máy ảo Vagrant cô lập để chạy/kiểm thử tác nhân AI [OpenClaw](https://github.com/openclaw/openclaw).
- **Đường dẫn:** `D:\META\Projects\OpenClaw\openclaw-vm`
- **Git:** branch `main`, remote `origin` → https://github.com/ali33/openclaw-vm.git

## Cấu trúc chính

- **Vagrantfile** — Cấu hình VM (Ubuntu 22.04, XFCE, OpenClaw, provisioning 8 bước).
- **docs/** — Tài liệu tiếng Anh: `customization.md`, `troubleshooting.md`.
- **vi/** — Bản dịch tiếng Việt toàn bộ file .md:
  - `vi/README.md`, `vi/CONTRIBUTING.md`, `vi/CHANGELOG.md`, `vi/CLAUDE.md`
  - `vi/docs/troubleshooting.md`, `vi/docs/customization.md`
  - `vi/.github/ISSUE_TEMPLATE/bug_report.md`, `vi/.github/ISSUE_TEMPLATE/feature_request.md`

## Lưu ý khi sửa tiếp

1. **Cập nhật song ngôn ngữ:** Khi sửa nội dung trong file .md gốc (README, CONTRIBUTING, docs/...), nhớ cập nhật luôn file tương ứng trong `vi/`.
2. **Commit/push:** Đã từng dùng message dạng `docs(vi): ...` cho thay đổi bản dịch.
3. **Các file .bat** (ssh-to-vm, vagrant-up, vagrant-halt, vagrant-destroy) có thể vẫn untracked — thêm vào git nếu muốn đẩy lên.

## Cập nhật lần cuối

- **Ngày:** 2026-03-13  
- **Việc đã làm:** Dịch toàn bộ file .md sang tiếng Việt vào thư mục `vi/`, commit và push lên `origin/main`.
