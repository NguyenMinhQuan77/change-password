# Ansible: Đổi mật khẩu User hunghv trên các VM Linux

Repository chuyên dụng chạy thủ công từ máy **`192.168.200.195`** để SSH vào danh sách các máy ảo (VM) khai báo trong `inventory.ini`, nâng quyền `sudo` và đổi mật khẩu mới cho user `hunghv`.

---

## 📁 Cấu trúc thư mục

```text
change_user_password/
├── ansible.cfg                    # Cấu hình tối ưu SSH connection, forks=20, pipelining=True
├── inventory.ini                  # Danh sách IP các VM cần đổi mật khẩu
├── site.yml                       # Playbook chính thực hiện đổi mật khẩu
├── playbook_verify.yml            # Playbook phụ kiểm tra SSH & sudo (không đổi mật khẩu)
├── group_vars/
│   └── all.yml                    # Khai báo user (hunghv), tự động gán pass sudo cho SSH
├── roles/
│   └── change_password/
│       └── tasks/
│           └── main.yml           # Tasks: kiểm tra user, chpasswd qua stdin, verify sudo
└── README.md                      # Hướng dẫn sử dụng
```

---

## 🚀 Hướng dẫn sử dụng trên máy `192.168.200.195`

### 1. Chuẩn bị danh sách VM
Mở file `inventory.ini` và thêm/bỏ chú thích (`#`) các IP máy chủ cần đổi:
```ini
[servers]
192.168.200.151
192.168.200.152
192.168.200.154
...
```

---

### 2. Chạy đổi mật khẩu bằng lệnh `ansible-playbook`

Chỉ cần chạy lệnh sau:
```bash
ansible-playbook site.yml -K
```
*(Hoặc dùng `-k -K` nếu mật khẩu SSH và mật khẩu sudo của bạn khác nhau: `ansible-playbook site.yml -k -K`)*

#### 👉 Trình tự Ansible sẽ hỏi bạn:
1. `BECOME password[sudo]:` ➔ **Nhập mật khẩu CŨ của user hunghv** (Ansible sẽ dùng mật khẩu này để SSH vào VM và leo quyền sudo).
2. `🔒 Nhập mật khẩu MỚI cho user hunghv:` ➔ **Nhập mật khẩu MỚI cần đổi**.
3. `🔒 Xác nhận lại mật khẩu MỚI:` ➔ **Nhập lại mật khẩu MỚI** để xác nhận.

Sau khi nhập xong, Ansible sẽ tự động chạy qua toàn bộ các server trong `inventory.ini`, đổi mật khẩu bằng `chpasswd` và kiểm tra xác thực quyền sudo ngay lập tức!

---

### 3. Các tùy chọn chạy nâng cao

#### Chạy thử nghiệm trên 1 server cụ thể trước:
Dùng cờ `-l <IP>` để test trước trên 1 server:
```bash
ansible-playbook site.yml -K -l 192.168.200.154
```

#### Chỉ kiểm tra SSH & sudo mà KHÔNG đổi mật khẩu:
Dùng playbook `playbook_verify.yml` để kiểm tra kết nối với mật khẩu:
```bash
ansible-playbook playbook_verify.yml -K
```
*(Nhập mật khẩu khi được hỏi, Ansible sẽ kết nối và in ra trạng thái thành công của từng VM).*

---

## 🔒 Cơ chế bảo mật
- **`no_log: true`**: Đảm bảo mật khẩu không bao giờ bị lộ ra màn hình terminal hay file log.
- **`chpasswd qua stdin`**: Không sử dụng chuỗi lệnh shell trực tiếp, do đó các mật khẩu chứa ký tự đặc biệt (`@`, `$`, `!`, `"`, `'`, `&`, `\`) không bao giờ bị lỗi hay mất ký tự.
- **`Xác thực tự động`**: Ngay sau khi đổi, Ansible tự xóa cache `sudo -k` và kiểm tra `sudo whoami` với mật khẩu mới. Nếu sai hoặc lỗi, playbook sẽ báo đỏ ngay lập tức.
