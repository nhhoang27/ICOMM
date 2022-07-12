# Jenkins là gì?

Là 1 chương trình mà nguồn mở thực hiện chức năng tích hợp liên tục (CI) và xây dựng các tác vụ tự động hóa. Chức năng của `Jenkins` là thực hiện danh sách các bước đã được định trước. VD: Biên dịch mã nguồn Java và xây dựng ứng dụng JAR từ các lớp kết quả.

Các bước có thể thực hiện bởi `Jenkins` như:
- Thực hiện 1 xây dựng phần mềm bằng cách sử dụng 1 hệ thống xây dựng như Apache, Maven hoặc Gradle
- Thực hiện 1 kịch bản shell script.
- Lưu trữ kết quả build
- Chạy thử phần mềm

Jenkins giám sát việc thực hiện các bước và cho phép ngừng quá trình, nếu 1 trong các bước không thành công, Jenkins cũng có thể gửi thông báo trong trường hợp xây dựng thành công hoặc thất bại. Jenkins có thể được mở rộng bằng các trình cắm thêm bổ sung.

## Cài đặt

### Cài đặt Java Runtime cho Jenkins

Để chạy Jenkins cần phiên bản Java 8

### Cài đặt Jenkins

Thêm thông tin **GPG Key** của Jenkins
```
# sudo wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
```

Thêm thông tin **Jenkins Repository** vào hệ thống rồi cập nhật lại thông tin Repo
```
# sudo echo 'deb https://pkg.jenkins.io/debian-stable binary/' > /etc/apt/sources.list.d/jenkins.list
# sudo apt update
```

Cài đặt Jenkins
```
# sudo apt install jenkins -y
```

Cài qua Repository hay gói package .deb thì sẽ có các thuận lợi sau:
- Jenkins sẽ hỗ trợ startup script cho service Jenkins luôn thay vì chạy đơn lẻ file java.jar.
- User 'Jenkins' sẽ được khởi tạo để dùng cho service Jenkins.
- Các thư mục cấu hình, hoạt động của Jenkins sẽ gồm: /var/log/jenkins/, /var/lib/jenkins/, /var/cache/jenkins . Owner của các folder này là user ‘jenkins’.
- File cấu hình thông tin global : /etc/sysconfig/jenkins
- File log Jenkins : /var/log/jenkins/jenkins.log

Kiểm tra phiên bản Jenkins tại file cấu hình
```
# systemctl start jenkins
# cat /var/lib/jenkins/config.xml  | grep -i "version"
```

### Khởi động Jenkins

Khởi động Jenkins và kích hoạt startup-service cho Jenkins
```
# systemctl start jenkins
# systemctl enable jenkins
# systemctl status jenkins
```

Jenkins lắng nghe request ở port mặc định là 8080 TCP.
```
# netstat -atnp | grep 8080
```

### Mở firewall rule cho Jenkins

```
# iptables -P INPUT -p tcp --dport 8080 -j ACCEPT
```

### Truy cập Jenkins - setup cơ bản

Đăng nhập vào link ip server Jenkins và port 8080 sẽ thất thông báo 'Unlock Jenkins' nhằm đảm bảo người truy cập là quản trị viên. Jenkins yêu cầu phải lấy thông tin chuỗi mật khẩu chứa ở file
```
# cat /var/lib/jenkins/secrets/initialAdminPassword
```

### Quản lý role

Vào **Manage Roles** sẽ thấy 1 role admin, role admin này được cấp full tất cả mọi quyền
<img src="./Images/roleAdmin.png" alt="Role Admin" width="800" />

Role ở đây bao gồm:
- Global roles: Các role này có tầm ảnh hưởng tới toàn bộ hệ thống, nghĩa là 1 User A được gán role X có quyền build job, thì User A này có khả năng build mọi job của Project.

### Assign role cho User

Để gán quyền hạn cho 1 đối tượng User, truy cập `Assign Roles`

## Pipeline

