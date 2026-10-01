# Bài 4: Cấu hình tường lửa bảo vệ máy chủ

## Bước 1: Đăng nhập vào Droplet

**Câu lệnh trên máy cá nhân:**

```powershell
ssh -i C:\Users\crou\.ssh\id_ed25519 devops@167.99.72.156
```

Trước khi kích hoạt tường lửa, phải cho phép cổng SSH để tránh mất kết nối tới máy chủ.

## Bước 2: Cài đặt UFW

UFW thường đã có sẵn trên Ubuntu. Có thể cài đặt hoặc bảo đảm gói đã tồn tại.

**Các câu lệnh trên Droplet:**

```bash
sudo apt update
sudo apt install ufw -y
```

## Bước 3: Thiết lập chính sách mặc định

Chặn toàn bộ kết nối đi vào nếu không có luật cho phép.

**Câu lệnh:**

```bash
sudo ufw default deny incoming
```

Cho phép máy chủ chủ động kết nối ra ngoài.

**Câu lệnh:**

```bash
sudo ufw default allow outgoing
```

## Bước 4: Mở các cổng cần thiết

Cho phép SSH qua cổng 22.

**Câu lệnh:**

```bash
sudo ufw allow 22/tcp comment 'SSH'
```

Cho phép HTTP qua cổng 80.

**Câu lệnh:**

```bash
sudo ufw allow 80/tcp comment 'HTTP'
```

**Câu lệnh kiểm tra các luật:**

```bash
sudo ufw show added
```

**Kết quả hiển thị:**

```text
Added user rules (see 'ufw status' for running firewall):
ufw allow 22/tcp comment 'SSH'
ufw allow 80/tcp comment 'HTTP'
```

## Bước 5: Kích hoạt UFW

**Câu lệnh kích hoạt:**

```bash
sudo ufw enable
```

Khi hệ thống cảnh báo việc bật tường lửa có thể làm gián đoạn kết nối SSH, nhập `y` để xác nhận. Do cổng 22 đã được cho phép trước đó nên phiên SSH vẫn hoạt động.

**Câu lệnh kiểm tra cấu hình:**

```bash
sudo ufw status verbose
```

**Kết quả hiển thị:**

```text
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW IN    Anywhere                   # SSH
80/tcp                     ALLOW IN    Anywhere                   # HTTP
22/tcp (v6)                ALLOW IN    Anywhere (v6)              # SSH
80/tcp (v6)                ALLOW IN    Anywhere (v6)              # HTTP
```

## Bước 6: Tạo DigitalOcean Cloud Firewall

1. Đăng nhập DigitalOcean Control Panel.
2. Chọn `Networking` trên thanh điều hướng.
3. Chọn thẻ `Firewalls`.
4. Chọn `Create Firewall`.
5. Đặt tên, ví dụ `ptit-devops-firewall`.
6. Trong phần Inbound Rules, xóa các luật không cần thiết và tạo hai luật sau:

| Loại | Giao thức | Cổng | Nguồn |
| --- | --- | --- | --- |
| SSH | TCP | 22 | Địa chỉ IP công khai của máy cá nhân hoặc All IPv4, All IPv6 |
| HTTP | TCP | 80 | All IPv4, All IPv6 |

Để an toàn hơn, luật SSH nên giới hạn nguồn là địa chỉ IP công khai của máy cá nhân. Nếu địa chỉ IP thường xuyên thay đổi, có thể tạm thời cho phép mọi nguồn trong phạm vi bài thực hành.

7. Trong phần Outbound Rules, giữ cấu hình mặc định cho phép lưu lượng đi ra ngoài để máy chủ cập nhật gói phần mềm và truy cập Internet.
8. Trong phần Apply to Droplets, chọn Droplet `ptit-devops-ss02`.
9. Chọn `Create Firewall`.
10. Kiểm tra lại để bảo đảm Droplet đã được gán vào Firewall.

## Ảnh chụp cấu hình Cloud Firewall

Trước khi nộp bài, chụp màn hình trang DigitalOcean hiển thị đầy đủ tên Firewall, hai Inbound Rules và Droplet được gán. Lưu ảnh vào cùng thư mục báo cáo với tên `SS02_HW04_Cloud_Firewall.png`, sau đó bỏ dấu phần ghi chú trong dòng dưới để hiển thị ảnh:

```markdown
![Cấu hình DigitalOcean Cloud Firewall](./SS02_HW04_Cloud_Firewall.png)
```

## Bước 7: Kiểm tra kết nối SSH

Mở một cửa sổ Terminal mới trên máy cá nhân.

**Câu lệnh kiểm tra SSH:**

```powershell
ssh -i C:\Users\crou\.ssh\id_ed25519 devops@167.99.72.156
```

Kết nối phải thành công. Không đóng phiên SSH hiện có cho đến khi lần kiểm tra mới hoàn tất.

## Bước 8: Kiểm tra dịch vụ HTTP

**Câu lệnh kiểm tra HTTP trên máy cá nhân:**

```powershell
curl.exe -I http://167.99.72.156
```

**Kết quả hiển thị:**

```text
HTTP/1.1 200 OK
Server: nginx/1.18.0 (Ubuntu)
Content-Type: text/html
```

Cũng có thể mở trình duyệt và truy cập `http://167.99.72.156` để xác nhận trang PTIT được hiển thị.

## Bước 9: Kiểm tra một cổng không được mở

Từ PowerShell trên máy cá nhân, thử kiểm tra cổng 8080.

**Câu lệnh kiểm tra cổng:**

```powershell
Test-NetConnection 167.99.72.156 -Port 8080
```

**Kết quả hiển thị:**

```text
ComputerName     : 167.99.72.156
RemoteAddress    : 167.99.72.156
RemotePort       : 8080
InterfaceAlias   : Wi-Fi
SourceAddress    : 192.168.1.12
TcpTestSucceeded : False
```

Lưu ý: Lệnh `ping` sử dụng giao thức ICMP, không kiểm tra trực tiếp luật TCP cho cổng 22 hoặc 80. Vì vậy, kiểm tra một cổng TCP không được mở sẽ phản ánh chính xác hơn hiệu quả của tường lửa.

## Log Terminal kiểm tra

**Log Terminal ghi nhận trạng thái hoạt động của UFW:**

```text
devops@ptit-devops-ss02:~$ sudo ufw status verbose
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW IN    Anywhere                   # SSH
80/tcp                     ALLOW IN    Anywhere                   # HTTP
22/tcp (v6)                ALLOW IN    Anywhere (v6)              # SSH
80/tcp (v6)                ALLOW IN    Anywhere (v6)              # HTTP
```

**Log kiểm tra từ máy cá nhân:**

```text
PS C:\Users\crou> ssh -i C:\Users\crou\.ssh\id_ed25519 devops@167.99.72.156
Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 5.15.0-157-generic x86_64)
devops@ptit-devops-ss02:~$ exit
logout
Connection to 167.99.72.156 closed.

PS C:\Users\crou> curl.exe -I http://167.99.72.156
HTTP/1.1 200 OK
Server: nginx/1.18.0 (Ubuntu)
Content-Type: text/html

PS C:\Users\crou> Test-NetConnection 167.99.72.156 -Port 8080
WARNING: TCP connect to (167.99.72.156 : 8080) failed
TcpTestSucceeded : False
```

## Kết luận

Droplet đã được bảo vệ bằng hai lớp tường lửa. UFW chặn mặc định lưu lượng đi vào và chỉ cho phép SSH cùng HTTP. DigitalOcean Cloud Firewall áp dụng cùng nguyên tắc ở tầng hạ tầng. Kết nối SSH và HTTP hoạt động bình thường, trong khi cổng TCP không được cho phép bị chặn.
