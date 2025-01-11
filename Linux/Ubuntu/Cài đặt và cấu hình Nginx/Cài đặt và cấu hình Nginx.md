# Cài đặt và cấu hình Nginx trên Ubuntu

## Cài đặt Nginx

- Cập nhật hệ thống và các gói phần mềm lên phiên bản mới nhất:

```sh
sudo apt-get update && sudo apt-get upgrade -y
```

- Tiến hành cài nginx:

```sh
sudo apt install nginx -y
```

## Cấu hình Nginx

- Cấu hình Nginx (bao gồm cấu hình các trang web) nằm tại đường dẫn ```/etc/nginx```.

- Khi cấu hình các trang web, hãy tận dụng hai thư mục ```sites-available``` và ```site-enabled``` tại thư mục cấu hình Nginx (```/etc/nginx```) để tiện quản lý các trang web. Trong đó, ```sites-available``` dùng để lưu tất cả cấu hình của các trang web, còn ```sites-enabled``` dùng để kích hoạt cấu hình các trang web mong muốn.

- Để bảo mật hơn, mỗi trang web nên được chạy bằng một user riêng biệt tránh việc bị tấn công chéo.

- Tạo user mới:

```sh
sudo useradd -r -s /bin/bash <tên-user>
```

- Chuyển quyền sở hữu thư mục hoạt động của trang web sang user vừa tạo:

```sh
sudo chown -R <tên-user>:<tên-user> /đường/dẫn/mã/nguồn/trang/web
```

- Thêm user của Nginx (www-data) vào nhóm người dùng vừa tạo:

```sh
sudo usermod -aG <tên-user> www-data
```

- Cấp lại quyền hợp lý cho thư mục hoạt động của trang web:

```sh
sudo chmod -R 750 /đường/dẫn/mã/nguồn/trang/web
```
