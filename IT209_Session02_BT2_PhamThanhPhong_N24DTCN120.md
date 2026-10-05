# Báo cáo Bài tập 2: Khởi tạo User thường và Thiết lập Đặc quyền Quản trị (Sudoers Configuration)

**Thông tin sinh viên:**
- **Họ và tên:** Phạm Thanh Phong
- **Mã sinh viên / Lớp:** N24DTCN120
- **Môn học:** IT209 - Phát triển ứng dụng Web / Cloud Infrastructure
- **Session:** Session 02 - Bài tập 2

---

## 1. Mục tiêu & Bối cảnh bài toán

### 1.1. Mục tiêu
- Vận hành máy chủ Linux/Ubuntu theo nguyên tắc **Đặc quyền tối thiểu (Principle of Least Privilege)** nhằm hạn chế tối đa các rủi ro thao tác sai sót trực tiếp bằng tài khoản `root`.
- Khởi tạo tài khoản người dùng thường (`devops`) và cấp quyền quản trị gián tiếp thông qua nhóm `sudo`.
- Cấu hình xác thực SSH Keypair cho user `devops` và thiết lập phân quyền tệp/thư mục nghiêm ngặt (`chmod 700` / `chmod 600`) để OpenSSH Server chấp thuận kết nối.

### 1.2. Ràng buộc
- Tài khoản người dùng mới: `devops`.
- Gia nhập nhóm: `sudo` (`wheel` trên RHEL/CentOS, `sudo` trên Debian/Ubuntu).
- Cơ chế SSH: Kế thừa Public Key từ `root` sang thư mục home `/home/devops/.ssh/authorized_keys`.
- Xác thực đặc quyền: Kiểm tra `sudo whoami` thành công trả về `root`.

---

## 2. Quy trình thực hiện chi tiết

### Bước 1: Đăng nhập vào máy chủ bằng tài khoản root
Từ máy cá nhân, đăng nhập vào Droplet qua SSH:
```bash
ssh -i ~/.ssh/id_digitalocean_ed25519 root@159.223.45.188
```

---

### Bước 2: Tạo User thường và gán vào nhóm sudo

1. **Khởi tạo tài khoản `devops`:**
   ```bash
   adduser devops
   ```
   *Nhập mật khẩu an toàn và các thông tin định danh (Full Name, Room Number,... hoặc Enter để mặc định).*

2. **Thêm tài khoản `devops` vào nhóm `sudo`:**
   ```bash
   usermod -aG sudo devops
   ```
   *Kiểm tra danh sách nhóm của user `devops`:*
   ```bash
   groups devops
   # Output: devops : devops sudo
   ```

---

### Bước 3: Cấu hình và phân quyền SSH Key cho User `devops`

Để user `devops` có thể xác thực bằng cùng SSH Key với `root`, tiến hành sao chép thư mục `.ssh` và cập nhật quyền sở hữu (owner):

**Cách 1: Sử dụng lệnh `rsync` (Khuyến nghị - nhanh và an toàn):**
```bash
rsync --archive --chown=devops:devops ~/.ssh /home/devops/
```

**Cách 2: Thực hiện thủ công và phân quyền chuẩn POSIX:**
```bash
# Tạo thư mục .ssh trong home của devops
mkdir -p /home/devops/.ssh

# Copy file authorized_keys từ root sang devops
cp /root/.ssh/authorized_keys /home/devops/.ssh/

# Thay đổi quyền sở hữu sang devops:devops
chown -R devops:devops /home/devops/.ssh

# Phân quyền nghiêm ngặt theo tiêu chuẩn OpenSSH
chmod 700 /home/devops/.ssh
chmod 600 /home/devops/.ssh/authorized_keys
```

> **Giải thích phân quyền:**
> - `chmod 700 /home/devops/.ssh` (`drwx------`): Chỉ chủ sở hữu `devops` mới có quyền đọc, ghi và truy cập thư mục.
> - `chmod 600 /home/devops/.ssh/authorized_keys` (`-rw-------`): Chỉ chủ sở hữu mới có quyền đọc/ghi file chứa public key. Nếu quyền bị lỏng lẻo (ví dụ: `777` hoặc `664`), OpenSSH Daemon sẽ từ chối xác thực vì vi phạm chính sách bảo mật StrictModes.

---

## 3. Kiểm tra & Nhật ký Terminal (Verification Logs)

### 3.1. Đăng nhập SSH từ máy cá nhân bằng user `devops`
Mở Terminal mới trên máy local:
```bash
PS C:\Users\ACER> ssh -i ~/.ssh/id_digitalocean_ed25519 devops@159.223.45.188
Welcome to Ubuntu 22.04.3 LTS (GNU/Linux 5.15.0-88-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

devops@it209-ubuntu-sgp-dev01:~$ whoami
devops
devops@it209-ubuntu-sgp-dev01:~$ pwd
/home/devops
```

### 3.2. Kiểm tra đặc quyền quản trị `sudo`
```bash
devops@it209-ubuntu-sgp-dev01:~$ sudo whoami
[sudo] password for devops: 
root
devops@it209-ubuntu-sgp-dev01:~$ sudo id
uid=0(root) gid=0(root) groups=0(root)
```

---

## 4. Tổng kết & Đánh giá bảo mật

| Tiêu chí | Trước khi cấu hình | Sau khi cấu hình |
| :--- | :--- | :--- |
| **Tài khoản thao tác hàng ngày** | `root` (Nguy cơ gõ nhầm lệnh hủy diệt hệ thống như `rm -rf /`) | `devops` (User thường, an toàn) |
| **Cơ chế leo quyền** | Trực tiếp root | Sử dụng tiền tố `sudo` có xác thực mật khẩu & ghi log audit |
| **Xác thực SSH** | Chỉ hỗ trợ root | Hỗ trợ đăng nhập trực tiếp qua `devops@<IP>` bằng SSH Key |
| **Phân quyền `.ssh`** | Root-only | Phân quyền `700` và `600` chuẩn bảo mật OpenSSH cho `/home/devops/.ssh` |
