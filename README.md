# Bài tập 2: Khởi tạo User thường và Thiết lập Đặc quyền Quản trị (Sudoers Configuration)

## Thông tin sinh viên
- **Họ và tên:** Phạm Thanh Phong
- **MSSV:** N24DTCN120
- **Môn học:** IT209 - Phát triển ứng dụng Web / Cloud Infrastructure
- **Session:** Session 02 - Bài tập 2

---

## Nội dung thực hiện
1. **Khởi tạo tài khoản thường & gán nhóm quản trị:**
   - Tạo user `devops`: `adduser devops`
   - Gán vào nhóm quản trị: `usermod -aG sudo devops`
2. **Cấu hình xác thực SSH Key:**
   - Sao chép và phân quyền: `rsync --archive --chown=devops:devops ~/.ssh /home/devops/`
   - Đảm bảo quyền thư mục: `chmod 700 /home/devops/.ssh` và `chmod 600 /home/devops/.ssh/authorized_keys`
3. **Kiểm tra đăng nhập & quyền sudo:**
   - Kết nối: `ssh -i ~/.ssh/id_digitalocean_ed25519 devops@<IP_ADDRESS>`
   - Kiểm tra quyền: `sudo whoami` -> Kết quả trả về `root`.

Chi tiết xem tại: [IT209_Session02_BT2_PhamThanhPhong_N24DTCN120.md](./IT209_Session02_BT2_PhamThanhPhong_N24DTCN120.md)
