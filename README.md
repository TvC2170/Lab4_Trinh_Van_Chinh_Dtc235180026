# Lab 4 — Quản lý cấu hình hạ tầng với Ansible

Bộ bài này triển khai các yêu cầu trong [Lab4_HuongDan.md](https://github.com/PhamThuongBlog/VHBTPM_DevOps_2026/blob/main/Labs/B%C3%A0i%204/Lab4_HuongDan.md): dựng 3 managed nodes, khai báo inventory, chạy ad-hoc, cấu hình Apache và chạy playbook nhiều máy chủ.

## Yêu cầu môi trường

- Docker Desktop đang chạy.
- Ansible 9 trở lên trên control node. Trên Windows, nên chạy các lệnh Ansible trong WSL Ubuntu.
- Python 3.9 trở lên.

## Chạy lab

Trong thư mục repo:

```bash
docker compose up -d
docker compose ps
ansible --version
ansible-inventory --list
ansible all -m ping
```

Chạy các lệnh ad-hoc của lab:

```bash
ansible all -m shell -a "cat /etc/os-release | head -3"
ansible all -m shell -a "df -h /"
ansible all -m shell -a "free -m | head -2"
ansible webservers -m shell -a "hostname"
ansible all -m file -a "path=/tmp/ansible-test state=touch"
ansible all -m apt -a "name=curl state=present update_cache=yes"
```

Chạy playbook web tier và kiểm tra trang:

```bash
ansible-playbook playbook-webserver.yml --syntax-check
ansible-playbook playbook-webserver.yml --check
ansible-playbook playbook-webserver.yml
curl http://localhost:8081
curl http://localhost:8082
```

Chạy full playbook, kiểm tra kết quả và tính idempotent:

```bash
ansible-playbook playbook-full.yml --syntax-check
ansible-playbook playbook-full.yml
ansible all -m shell -a "/opt/myapp/health-check.sh"
ansible all -m shell -a "cat /opt/myapp/config.env"
ansible-playbook playbook-webserver.yml
```

Lần chạy playbook web tiếp theo không có thay đổi sẽ báo `changed=0`. Khi cập nhật nội dung template, handler sẽ restart Apache.

## Dọn dẹp

```bash
docker compose down
```

## Cấu trúc

- `inventory.ini`: ba máy, nhóm web/database/production và thông tin SSH.
- `ansible.cfg`: cấu hình inventory mặc định và output.
- `playbook-webserver.yml`: cài Apache, publish trang từ Jinja2 template, handler restart.
- `templates/index.html.j2`: trang HTML có facts của từng máy.
- `playbook-full.yml`: gói chung, cấu hình ứng dụng và công cụ monitoring.
- `docker-compose.yml`: ba Ubuntu SSH containers với port mapping.

> Mật khẩu `root` trong inventory chỉ dùng cho lab cục bộ. Không dùng thông tin xác thực này trong môi trường thật.

## Ảnh chụp để nộp

Chụp terminal sau khi chạy thành công các lệnh ở trên và lưu lần lượt vào `screenshots/` theo tên trong yêu cầu môn học (`01-ansible-ping.png` đến `05-idempotent.png`). Ảnh cần được chụp trên máy có Docker và Ansible; không tạo ảnh giả lập kết quả chạy.

Điền họ tên/MSSV vào `ho_ten_mssv.txt` trước khi nén bộ bài nộp.
