### Images

- Chỉ có thể đọc, không thể sửa đổi
- Khi images khởi chạy, phiên bản thực thi của nó được gọi là container

1. Kiểm tra danh sách images: `docker images`
   Danh sách kết quả trả ra
   | REPOSITORY | TAG | IMAGE ID |CREATED |SIZE|
   |----------|:-------------:|------:|------:|------:|
   |Tên Images| phiên bảng |ID của Images |Thời gian tạo |Dung lượng|
   |postgres| latest |aa7ac7cd82f1 |12 days ago |360MB|
1. Kiểm tra danh sách các lệnh với image: `docker image --help`
1. Kiểm tra danh sách option của một lệnh image: `docker image build --help`
1. Tìm kiếm image muốn sử dụng: `docker search <tên từ khoá>`
1. Tải một image: `docker pull <tên image>:<tag (phiên bản)>`. Mặc định nếu không điền tag sẽ là `lastest`
1. Xoá một image bằng tên: `docker rm <tên image>:<tag (phiên bản)>`
1. Xoá một image bằng ID: `docker rm <gõ đầy đủ hoặc 1 phần đầu của ID>`

```
#kiểm tra phiên bản
docker --version

#liệt kê các image
docker images -a

#xóa một image (phải không container nào đang dùng)
docker images rm imageid

#tải về một image (imagename) từ hub.docker.com
docker pull imagename

#liệt kê các container
docker container ls -a

#xóa container
docker container rm containerid

#tạo mới một container
docker run -it imageid

#thoát termial vẫn giữ container đang chạy
CTRL +P, CTRL + Q

#Vào termial container đang chạy
docker container attach containerid

#Chạy container đang dừng
docker container start -i containerid

#Chạy một lệnh trên container đang chạy
docker exec -it containerid command

```
