## Yêu cầu chung

Chạy một server sử dụng workpress
Cấu hình như sau:

PHP : 7.3-FPM (php-product)

- Port: 9000
- Cài mysqli, pdo_mysql:
  - docker-php-ext-install mysqli
  - docker-php-ext-install pdo_mysql
- Thư mục làm việc: /home/sites/site1

APACHE HTTPD: (c-httpd01)

- Port: 80, 443
- config: /usr/local/apache2/conf/httpd.conf
  - Nạp: mod_proxy, mod_proxy_fcgi
  - Thư mục làm việc: /home/sites/site1
  - index mặc định: index.php index.html
  - PHPHANDLER: AddHandler "proxy:fcgi://php-product:9000" .php

MYSQL: (mysql-product)

- Port:3304
- Config: /etc/mysql/my.cnf
  - default-authentication-plugin=mysql_native_password
- databases: /var/lib/mysql -> /mycode/db
- MYSQL_ROOT_PASSWORD: 123abc
- MYSQL_DATABASE: db_site
- MYSQL_USER: siteuser
- MYSQL_PASSWORD: sitepass

NETWORK

- bridge
- my-network

VOLUME: dir-site

- bind, device = /mycode/

## Các bước thực hiện

Chuẩn bị các file cấu hình

### PHP

Với PHP hãy tạo một image mới có chứa các dịch vụ cần thiết:

- Tạo một thư mục mới tên php; `mkdir php`
- Tạo một file tên là Dockerfile: touch ./php/Dockerfile
- Biên tập Dockerfile vừa tạo bằng visual code: code ./php/Dockerfile
- Image mới được tạo từ PHP:

```bash
FROM php:7.3-fpm
```

- Từ image cơ sở ta chạy lệnh cài đặt 2 gói cho PHP:
  - docker-php-ext-install mysqli
  - docker-php-ext-install pdo_mysql

```bash
FROM php:7.3-fpm

RUN docker-php-ext-install mysqli
RUN docker-php-ext-install pdo_mysql
```

- Thiết lập thư mục làm việc cho PHP sử dụng lệnh WORKDIR:

```bash
FROM php:7.3-fpm

RUN docker-php-ext-install mysqli
RUN docker-php-ext-install pdo_mysql

WORKDIR /home/sites/site1
```

### APACHE HTTPD

- Trước tiên để có thể cấu hình config như ý muốn ta cần có sẵn 1 file httpd.conf
  Cách nhanh nhất là tạo một container từ image mà docker có sẵn rồi copy nó ra thư mục dự án
  Lệnh path cho thư mục trên host luôn xuất phát từ đường dẫn host root

```bash
docker run --rm -v /Users/leminhluan4244/Documents/Project/x-marines/train-yourself/docker/docker-compose/mycode/:/home/ httpd cp /usr/local/apache2/conf/httpd.conf /home/

# --rm: Chạy xong tự xoá container
# -v /mycode/:home/ : Ánh xạ thư mục mycode vào thư mục home của container
# httpd: Container mới tạo chạy từ image httpd
# cp /usr/local/apache2/conf/httpd.conf /home/: Sau khi chạy thì copy /usr/local/apache2/conf/httpd.conf ra thư mục home
```

- Mở comment cho 2 module: mod_proxy, mod_proxy_fcgi

```bash
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_fcgi_module modules/mod_proxy_fcgi.so
```

- Thiết lập thư mục làm việc mặc định
  Chỉnh sửa lại DocumentRoot trong file httpd.conf đang mở

```bash
DocumentRoot "/home/sites/site1"
<Directory "/home/sites/site1">
```

- Thiết lập truy vấn mặc định index.html hoặc index.php. Tiếp tục tuỳ chỉnh file đang mở

```bash
<IfModule dir_module>
    DirectoryIndex  index.php index.html
</IfModule>
```

- Thêm vào Handler để gọi php thông qua proxy. Thêm dòng sau vào cuối file

```bash
AddHandler "proxy:fcgi://php-product:9000" .php
```

### MYSQL

- Tạo thư mục db để chứa database: ``
- Tương tự apache ta tạo một file config bằng cách copy từ một image docker có sẵn

```bash
docker run --rm -v /Users/leminhluan4244/Documents/Project/x-marines/train-yourself/docker/docker-compose/mycode/:/home/ mysql cp /etc/my.cnf /home/
```

- Tuỳ chỉnh file my.conf. Thêm vào tuỳ chỉnh sử dụng pass mặc định:
  Thêm vào cuối file

```bash
default-authentication-plugin=mysql_native_password
```

- Ánh xạ folder lưu csdl:

### Xây dụng docker compose

Từ các file có sẵn ta tạo docker compose

- Tạo file docker compose: `touch docker-compose.yml`
- Sử dụng version: `version: "3.8"`
- Định nghĩa network:

```yml
networks:
  my-network:
    driver: bridge
```

- Mô tả ổ đĩa. Tạo volume tên dir-site

```yml
volumes:
  dir-site:
    driver_opts:
      type: none
      device: /Users/leminhluan4244/Documents/Project/x-marines/train-yourself/docker/docker-compose/mycode/sites
      o: bind
```

- Sử dụng 3 dịch vụ (services): PHP, Apache, Mysql
  Mô tả 3 dịch vụ với mục services:

```yml
services:
  # container PHP
  my-php:
    container_name: php-product
    build:
      dockerfile: Dockerfile # Tên file khởi chạy contianer từ image
      context: ./php/ # Vị trí chứa Dockerfile tính từ thư mục chứ docker compose file
    hostname: php # Đặt tên cho container được tạo ra
    restart: always # Luôn tự khởi động lại khi có gì đó cập nhật mới hoặc dịch vụ PHP bị dừng
    networks:
      - my-network
    volumes:
      - dir-site:/home/sites # Ánh xạ ổ đĩa dir-site vào thư mục /home/sites của container
  # container HTTPD
  my-http:
    container_name: c-httpd01
    image: "httpd:latest" # Dịch vụ này không đọc config từ docker file mà lấy image trực tiếp từ dọcker hub
    hostname: httpd
    restart: always
    networks:
      - my-network
    volumes:
      - dir-site:/home/sites # Ánh xạ ổ đĩa dir-site vào thư mục /home/sites của container
      - ./httpd.conf:/usr/local/apache2/conf/httpd.conf # Ánh xạ file config trước đó để có thể tuỳ chỉnh cấu hình
    ports:
      - "9999:80" # Truy cập cổng 9999 trên máy host thì sẽ ánh xạ tới cổng 80 của container
      - "443:443" # tương tự
  # container MYSQL
  my-mysql:
    container_name: mysql-product
    image: "mysql:latest"
    hostname: mysql
    restart: always
    networks:
      - my-network
    volumes:
      - ./db:/var/lib/mysql # Ánh xạ thư mục lưu trữ data vào thư mục db trên máy host
      - ./my.cnf:etc/my.conf
    environment: # Thiết lập DB và tài khoản
      - MYSQL_ROOT_PASSWORD:123abc
      - MYSQL_DATABASE:db_site
      - MYSQL_USER:siteuser
      - MYSQL_PASSWORD:sitepass
```

- Khởi chạy: Sử dụng lệnh : `docker compose up`
- Thử cài đặt mã nguồn Joomla
