# Triển khai web app ASP.NET Core với reversed proxy Nginx

### Cài đặt ASP.NET Core Runtime

- Cài đặt ASP.NET Core Runtime theo phiên bản tương ứng:

```sh
sudo apt-get update && sudo apt-get install -y aspnetcore-runtime-*.0
```

- Trong đó, ```*``` là phiên bản ASP.NET Core Runtime tương ứng.

- Lưu ý, nếu cài ASP.NET Core 9.0 lên hệ điều hành Ubuntu phiên bản 24.04 trở về trước, cần thêm Ubuntu .NET backports package repository vào hệ điều hành đang chạy. Để thêm repository này, chạy lệnh sau:

```sh
sudo add-apt-repository ppa:dotnet/backports
```

- Nhấn **Enter** để tiếp tục cài đặt repository này:

![image](./images/Cài%20đặt%20ASP.NET%20Core%20Runtime%209.0.png)

### Cấu hình Kestrel

- Kestrel là một web server nhẹ và hiệu suất cao, nhưng không được thiết kế để chạy trên môi trường internet vì thiếu các tính năng bảo mật.

- Kestrel đã được cài đặt khi cài runtime, vì vậy chỉ cần cấu hình để chạy với web tự động bằng file service.

- Tạo file service tại đường dẫn ```/etc/systemd/system/```:

```sh
sudo nano /etc/systemd/system/<tên-service>.service
```

- Nội dung file service như sau:

```sh
[Unit]
Description=Example .NET Web API App running on Linux

[Service]
WorkingDirectory=/var/www/helloapp
ExecStart=/usr/bin/dotnet /var/www/helloapp/helloapp.dll
Restart=always
RestartSec=10
KillSignal=SIGINT
SyslogIdentifier=dotnet-example
User=www-data
Environment=ASPNETCORE_ENVIRONMENT=Production
Environment=DOTNET_PRINT_TELEMETRY_MESSAGE=false
Environment=ASPNETCORE_URLS=http://localhost:5000

[Install]
WantedBy=multi-user.target
```

- Trong đó, cần lưu ý:

    - ```Description```: Mô tả của service.

    - ```WorkingDirectory```: Thư mục hoạt động của trang web.

    - ```ExecStart```: Lệnh được thực hiện trong service, tại đây cần lưu ý ```/var/www/helloapp/helloapp.dll``` là đường dẫn file ```.dll``` của trang web.

    - ```RestartSec```: Thời gian (giây) service tự khởi động lại sau khi bị lỗi.

    - ```User```: User được thiết lập để chạy với trang web.

    - ```Environment=ASPNETCORE_URLS```: Cấu hình cổng mà trang web sẽ chạy. Mỗi trang web ASP.NET Core phải được cấu hình **chạy trên các cổng khác nhau**.

- Cuối cùng, nhấn ```Ctrl + X``` rồi nhấn ```y``` và ```Enter``` để đóng và lưu cấu hình.

- Khởi chạy và kiểm tra service:

```sh
sudo systemctl start <tên-service>.service
sudo systemctl status <tên-service>.service
```

- Để khởi chạy tự động cùng hệ thống, dùng lệnh sau:

```sh
sudo systemctl enable <tên-service>.service
```

### Cấu hình Nginx

- Nếu chưa cài đặt nginx, tham khảo cài đặt và cấu hình trang web chạy trên nginx tại [đây](https://github.com/quyen0508/trienkhaihethong/blob/main/Linux/Ubuntu/C%C3%A0i%20%C4%91%E1%BA%B7t%20v%C3%A0%20c%E1%BA%A5u%20h%C3%ACnh%20Nginx/C%C3%A0i%20%C4%91%E1%BA%B7t%20v%C3%A0%20c%E1%BA%A5u%20h%C3%ACnh%20Nginx.md).

- File cấu hình trang web như sau:

```sh
server {
    listen        80;
    server_name   example.com *.example.com;
    location / {
        proxy_pass         http://127.0.0.1:5000;
        proxy_http_version 1.1;
        proxy_set_header   Upgrade $http_upgrade;
        proxy_set_header   Connection keep-alive;
        proxy_set_header   Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header   X-Forwarded-Proto $scheme;
    }
}
```

- Trong đó cần lưu ý:

    - ```listen```: Cổng lắng nghe của web từ Internet.

    - ```server_name```: Tên miền của trang web.

    - ```proxy_pass```: Đường dẫn tới trang web chạy bởi Kestrel được cấu hình theo service ở trên.
