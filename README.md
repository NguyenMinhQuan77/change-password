# Ansible: Đổi mật khẩu User hunghv trên các VM Linux

Repository chuyên dụng chạy thủ công từ máy **`192.168.200.195`** (hoặc `jenkin-slave2`) để SSH vào danh sách các máy ảo (VM) khai báo trong `inventory.ini`, nâng quyền `sudo` và đổi mật khẩu mới cho user `hunghv`.

---

## 📁 Cấu trúc thư mục

```text
change_user_password/
├── ansible.cfg          # Cấu hình SSH: forks=20 (chạy song song 20 server), pipelining=True
├── inventory.ini        # Khai báo danh sách IP các máy ảo
├── site.yml             # Playbook chính thực hiện đổi mật khẩu
├── playbook_verify.yml  # Playbook phụ test SSH & sudo (không đổi mật khẩu)
├── group_vars/
│   └── all.yml          # Cấu hình mặc định target_user=hunghv, ansible_user=hunghv
├── roles/
│   └── change_password/
│       └── tasks/
│           └── main.yml # Tasks: kiểm tra user, chpasswd qua stdin, verify sudo
└── README.md            # Hướng dẫn chi tiết
```

---

## 🚀 Hướng dẫn sử dụng trên máy `192.168.200.195` (hoặc `jenkin-slave2`)

### 1. Chuẩn bị danh sách VM
Mở file `inventory.ini` và thêm/bỏ chú thích (`#`) các IP máy chủ cần đổi:
```ini
[servers]
192.168.200.67
192.168.200.154
...
```

---

### 2. Bước 1 (Khuyên dùng): Kiểm tra kết nối trước khi đổi
Chạy playbook kiểm tra để chắc chắn mật khẩu hiện tại đăng nhập và sudo thành công:

```bash
# Kiểm tra thử trên 1 server cụ thể:
ansible-playbook playbook_verify.yml -l 192.168.200.67

# Hoặc kiểm tra toàn bộ server trong inventory:
ansible-playbook playbook_verify.yml
```
👉 **Playbook sẽ hỏi duy nhất 1 câu:**
`🔑 Nhập mật khẩu hiện tại của user hunghv (dùng chung cho cả SSH và sudo):`
*(Bạn chỉ cần gõ mật khẩu hiện tại của hunghv 1 lần duy nhất, Ansible sẽ tự dùng mật khẩu đó để vừa SSH vừa test quyền sudo).*

---

### 3. Bước 2: Chạy đổi mật khẩu hàng loạt
Khi đã kiểm tra OK, chạy playbook đổi mật khẩu:

```bash
# Chạy thử nghiệm trên 1 server trước:
ansible-playbook site.yml -l 192.168.200.67

# Chạy cho toàn bộ danh sách server:
ansible-playbook site.yml
```

👉 **Playbook sẽ hỏi 3 câu:**
1. `🔑 Nhập mật khẩu HIỆN TẠI của user hunghv:` ➔ Gõ pass cũ (dùng để SSH & sudo).
2. `🔒 Nhập mật khẩu MỚI cần đổi cho user hunghv:` ➔ Gõ pass mới cần đặt.
3. `🔒 Xác nhận lại mật khẩu MỚI:` ➔ Gõ lại pass mới để xác nhận.

Sau khi nhập xong, Ansible sẽ tự động kết nối qua toàn bộ server, dùng lệnh `chpasswd` đổi mật khẩu và xác thực ngay quyền sudo bằng mật khẩu mới!
