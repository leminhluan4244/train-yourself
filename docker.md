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
- Chạy một lệnh trên container đang chạy: `docker exec -it <containerid> command`
- Vào termial container đang chạy: `docker container attach <containerid>`

###### Turn off

- Tắt 1 container : `exit`

######
