# 🔍 Ansible Audit System

## 📌 Giới thiệu

**Ansible Audit System** là một tool nội bộ dùng để kiểm tra nhanh trạng thái hệ thống Linux theo từng nhóm service.

Tool giúp:

* Kiểm tra service có chạy đúng không
* Kiểm tra service chạy đúng user hay không
* Kiểm tra các cấu hình hệ thống quan trọng
* Xuất báo cáo theo từng server và từng group

👉 Phù hợp cho:

* Audit hệ thống định kỳ
* Pre-check trước khi deploy
* Chuẩn hóa môi trường production

---

## 🏗️ Cấu trúc project

```
ansible-audit/
├── ansible.cfg
├── hosts.ini
├── group_vars/
│   └── all.yml
├── playbooks/
│   └── quick_check.yml
├── reports/
└── logs/
```

---

## ⚙️ Nguyên lý hoạt động

### 1. Inventory (hosts.ini)

Các server được chia theo nhóm service:

```
[canal]
node-2 ansible_host=172.16.0.3
node-3 ansible_host=172.16.0.4

[datax]
node-4 ansible_host=172.16.0.5

[dophin-scheduler]
node-1 ansible_connection=local
```

👉 Mỗi group đại diện cho 1 loại service

---

### 2. Cấu hình service (group_vars/all.yml)

Chỉ sử dụng **1 file duy nhất** để map service:

```yaml
report_dir: "./reports"

service_checks:
  canal:
    service_name: "canal"
    service_user: "canal"

  datax:
    service_name: "datax"
    service_user: "datax"

  dophin-scheduler:
    service_name: "dophin-scheduler"
    service_user: "dolphin"
```

👉 Playbook sẽ tự động:

* Nhận diện host thuộc group nào
* Lấy config tương ứng để kiểm tra

---

### 3. Playbook (quick_check.yml)

Playbook thực hiện:

1. Xác định group của host
2. Load config service tương ứng
3. Thực hiện các kiểm tra hệ thống
4. Xuất báo cáo

---

## 🔎 Nội dung kiểm tra

### 🚀 1. Service chính (theo group)

* Service có tồn tại không
* Service có đang chạy (`active`) không
* Lấy `MainPID`
* Xác định:

  * User đang chạy service
  * Command đang chạy
* So sánh với user mong muốn
* Nếu là Java:

  * Lấy `-Xms`, `-Xmx`

---

### 👤 2. User hệ thống

* Lấy danh sách từ `/etc/passwd`
* Hiển thị **5 user cuối cùng có nologin**

---

### ⏰ 3. NTP / Chrony

* Kiểm tra service `chrony` / `chronyd`
* Kiểm tra trạng thái
* Xác định NTP server đang sync

---

### ☕ 4. Java process

* Liệt kê tất cả process Java đang chạy

---

### 🔥 5. Firewall (UFW)

* Kiểm tra có cài đặt không
* Hiển thị trạng thái

---

### 🛡 6. KESL (Kaspersky)

* Kiểm tra service hoặc process
* Xác định trạng thái hoạt động

---

### 💾 7. Mount `/data`

* Kiểm tra trong `/etc/fstab`
* Kiểm tra mount thực tế

---

## 📄 Output

Báo cáo được lưu theo từng group:

```
reports/
├── canal/
│   ├── 172.16.0.3.log
│   └── 172.16.0.4.log
├── datax/
│   └── 172.16.0.5.log
└── dophin-scheduler/
    └── node-1.log
```

---

## ❌ Cơ chế fail

Playbook hoạt động theo nguyên tắc:

* ❗ Không fail giữa chừng
* ✅ Luôn ghi report trước
* ❌ Fail ở cuối nếu có lỗi

### Điều kiện fail:

* Service không tồn tại / không chạy / sai user
* Chrony lỗi hoặc không chạy
* KESL không hoạt động
* `/data` chưa mount đúng

---

## ▶️ Cách sử dụng

### 1. Chạy toàn bộ

```bash
ansible-playbook playbooks/quick_check.yml -K
```

---

### 2. Chạy theo group

```bash
ansible-playbook playbooks/quick_check.yml -K -l canal
```

---

## 🔐 Yêu cầu

* Ansible >= 2.9
* SSH access tới các server
* User có quyền sudo (become)

---

## 🚀 Best Practice (Production)

* Sử dụng SSH key thay vì password
* Không hardcode password trong `group_vars`
* Chạy qua cron hoặc CI/CD
* Log tập trung về hệ thống monitoring

---

## 📈 Mở rộng

Có thể phát triển thêm:

* Check nhiều service trên cùng 1 host
* Xuất report JSON
* Tích hợp Prometheus / Grafana
* Push log về ELK / Loki
* Dashboard theo trạng thái hệ thống

---

## 🎯 Use Case

* Audit hệ thống định kỳ
* Kiểm tra nhanh production
* Chuẩn hóa cấu hình server
* Pre-deploy validation

---

## 👨‍💻 Author

Thisorp
