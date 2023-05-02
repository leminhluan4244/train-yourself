### Sơ lượt về docker

- Kiểm tra phiên bản: `docker --version`

### Images

- Chỉ có thể đọc, không thể sửa đổi
- Khi images khởi chạy, phiên bản thực thi của nó được gọi là container

#### Danh sách lệnh

- Kiểm tra danh sách các lệnh với image: `docker image --help`
- Kiểm tra danh sách option của một lệnh image: `docker image build --help`

###### List

- Kiểm tra danh sách images: `docker images` hoặc `docker images -a`

  Danh sách kết quả trả ra
  | REPOSITORY | TAG | IMAGE ID |CREATED |SIZE|
  |----------|:-------------:|------:|------:|------:|
  |Tên Images| phiên bảng |ID của Images |Thời gian tạo |Dung lượng|
  |postgres| latest |aa7ac7cd82f1 |12 days ago |360MB|

###### Search

- Tìm kiếm image muốn sử dụng: `docker search <tên từ khoá>`

###### Create

- Tải một image: `docker pull <tên image>:<tag (phiên bản)>`. Mặc định nếu không điền tag sẽ là `lastest`

> Mỗi lần chạy lệnh này một container mới sẽ được tạo và thực thi tức thì

###### Delete

- Xoá một image bằng tên: `docker rm <tên image>:<tag (phiên bản)>`
- Xoá một image bằng ID: `docker rm <gõ đầy đủ hoặc 1 phần đầu của ID>`

### Container

###### List

- Liệt kê danh sách các container: `docker container ls -a` hoặc `docker container ps`

###### Create

- Tạo mới container: `docker run -it <imageid>`

###### Delete

- Xoá một container bằng tên: `docker container rm <containerid>`

###### Keep

- Thoát termial vẫn giữ container đang chạy: CTRL +P, CTRL + Q

###### Run

- Chạy container đang dừng: `docker container start -i <containerid>`
- Chạy một lệnh trên container đang chạy từ bên ngoài: `docker exec -it <containerid> command`
- Vào termial container đang chạy: `docker container attach <containerid>`

###### Turn off

- Tắt 1 container : `exit`

###### Save

- Commit một container thành một iamge: `docker commit <tên>:<version>`.
  > Lệnh này chỉ thực hiện khi container đang dừng.

#### Chia sẽ dữ liệu

- Chia sẽ dữ liệu khi tạo container từ một image với máy host

```bash
docker run -it -v <Địa chỉ trên máy góc>:<Địa chỉ muốn hiển thị trên container>
```

- Chia sẽ dữ liệu giữa các container

```bash
docker run -it --volumes-from <Tên container bạn muốn share>
```

- Chia sẽ dữ liệu thông qua việc tạo và quản lý ổ đĩa. Ổ đĩa cho phép lưu trữ dữ liệu trong docker, dù container liên kết tới nó có bị xoá thì data vẫn còn trong ổ đĩa.

```bash
# Kiểm tra có ổ đĩa nào không
docker volume ls

# Tạo một ổ đĩa
docker volume create <tên ổ đĩa>

# Kiểm tra thông tin ổ đĩa
docker volume inspect <tên ổ đĩa>

# Xoá một ổ đĩa
docker volume rm <tên ổ đĩa>

# Gán ổ đĩa vào container
docker run -it --name D1 --mount source=<Tên ổ muốn gán>,target=<Địa chỉ ánh xạ> <image>:<phiên bản >
```

- Tạo ra một ổ đĩa, và ánh xạ nó đến máy host

```bash
# Tạo ổ đĩa
docker volume create --opt device=<Đại chỉ trên máy host> --opt type=none --opt o=bind <Tên ổ đĩa>

# Gắn ổ đĩa
docker run -it -v DISK1:<địa chỉ trong container> <image>:<version>
```

#### Dockerfile

##### Yêu cầu ban đầu

Tạo một image mới có tên myimage:v1 từ iamge cơ sở là centos:latest trong image mới có cài đặt:

- httpd
- htop
- vim
- copy một số file dữ liệu index.html và thư mục /var/www/html
- thi hành nền /usr/sbin/httpd khi container từ image chạy

##### Ví dụ về sử dụng docker mà không dùng Dockerfile

1. Tạo một thư mực mycode, tạo thư mục myimage, trong thư mục tạo file test.html, bên trong hiển thị một thẻ h1 nội dung là 'Test image'
1. Chạy một container từ centos:latest : `docker run -it --name cent centos:latest`
1. Chạy `yum update`
1. Cài đặt httpd và httpd tool; `yum install httpd httpd-tools`. Kiểm tra xem cài hay chưa; `httpd -v`.
1. Cài đặt vim: `yum install vim`
1. Cài đặt các core mở rộng: `yum install epel-release -y`
1. Cập nhật lại centos: `yum update -y`
1. Cài đặt htop: `yum install htop -y`
1. Copy dữ liệu html và httpd của container: `docker cp /mycode/myimage/test.html cent:/var/www/html/`
1. Lưu container thành image: `exit`. Sau đó chạy `docker commit cent myimage:v1`
1. Kiểm tra image có chưa: `docker image`
1. Xoá đi container cũ: `docker rm cent`
1. Chạy thử: `docker run --rm -p 9876:80 myimage:v1 httpd -D FOREGROUND`

##### Sử dụng Dockerfile

Vào thư mục /mycode/myimage tạo một file Dockerfile:

```dockerfile
# xây dựng image mới từ image centos:latest (CENTOS 7)
FROM centos:latest

# Thêm hai lệnh sau nếu CENTOS 8 không cập nhật được
# RUN dnf -y --disablerepo '*' --enablerepo=extras swap centos-linux-repos centos-stream-repos
# RUN dnf -y distro-sync



# Cập nhật các gói và cài vào đó HTTPD, HTOP, VIM
RUN cd /etc/yum.repos.d/
RUN sed -i 's/mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-*
RUN sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-*
RUN yum update -y
RUN yum install httpd httpd-tools -y
RUN yum install epel-release -y \
    && yum update -y \
    && yum install htop -y \
    && yum install vim -y

#Thiết lập thư mục hiện tại
WORKDIR /var/www/html
# Copy tất cả các file trong thư mục hiện tại (.)  vào WORKDIR
ADD . /var/www/html

#Thiết lập khi tạo container từ image sẽ mở cổng 80
# ở mạng mà container nối vào
EXPOSE 80

# Khi chạy container tự động chạy ngay httpd
ENTRYPOINT ["/usr/sbin/httpd"]

#chạy terminate
CMD ["-D", "FOREGROUND"]
```

Sau khi tạo xong thì thực thi

```bash
docker build -t myimage:v1 -f Dockerfile .
```

Sau khi tạo xong thì chạy thử:

```bash
docker run -p 6789:80 myimage:v1
```

#### Docker compose

Docker compose gần giống như Dockerfile
