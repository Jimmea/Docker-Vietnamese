# Chia sẻ dữ liệu giữa Docker Host và Container

Cách mount thư mục máy Host hoặc tạo ổ đĩa để chia sẻ dữ liệu vào Container cũng như chia sẻ dữ liệu các container với nhau

Trong phần này tìm hiểu cách chia sẻ dữ liệu giữa máy Host và Container, giữa các Container với nhau bằng cách sử dụng một thư mục trên máy Host làm nơi lưu trữ tập trung. (Tạo ổ đĩa rồi mount nó vào các container sẽ tìm hiểu sau, phần này sử dụng trực tiếp các thư mục).


![mounts_volumn](https://docs.docker.com/storage/images/types-of-mounts-volume.png)

`Máy Host` là hệ thống bạn đang chạy Docker Engine. Một thư mục của máy Host có thể chia sẻ để các Container đọc, lưu dữ liệu.

- Chia sẻ dữ liệu Host/Container
- Chia sẻ dữ liệu giữa các Container
- Quản lý các ổ đĩa

## Container - ánh xạ thư mục máy Host
Thông tin:
- Thư mục cần chia sẻ dữ liệu trên máy host là: path_in_host
- Khi chạy container thư mục đó được mount - ánh xạ tới path_in_container của container

Để có kết quả đó, tạo - chạy container với tham số thêm vào `-v path_to_data:path_in_container`

Ví dụ:
```
docker run -it -v /home/sitesdata:/home/data my_ubuntu ubuntu
```

Lúc này, dữ liệu trên thư mục `/home/sitesdata/` của máy Host thì trong container có thể truy cập, cập nhật sửa đổi ... thông qua đường dẫn `/home/data`

## Chia sẻ dữ liệu giữa các Container
Có container với id hoặc name là `my_ubuntu`, trong đó nó có mount thư mục Host vào (hoặc đã được chia sẻ tử Container khác)

Giờ chạy, tạo container khác cũng nhận thư mục chia sẻ dữ liệu như `my_ubuntu`. Để làm điều đó thêm vào tham số `--volumes-from container_first`

Ví dụ:
```
docker run -it --name my_ubuntu_v2 --volumes-from my_ubuntu ubuntu
```

Bạn đã tạo ra một Container nhận thư mục chia sẻ như container có ID hoặc tên là my_ubuntu tạo trước đó.

## Quản lý các ổ đĩa với docker volume 
Có thể tạo và quản lý các ổ đĩa bên ngoài container, lệnh làm việc với ổ đĩa là docker volume với các trường hợp cụ thể:

Liệt kê danh sách các ổ đĩa:
```
docker volume ls
```

Tạo một ổ đĩa:
```
docker volume create name_volume
```

Xem thông tin chi tiết về đĩa:
```
docker volume inspect name_volume
```

Xóa một ổ đĩa:
```
docker volume rm name_volume
```

**Mount một ổ đĩa vào container (--mount)**
```
# Tạo ổ đĩa có tên firstdisk
docker volume create disk_one

# Mount ổ đĩa vào container
# container truy cập tại /home/disk_one

docker run -it --name my_ubuntu_v3 --mount source=disk_one,target=/home/disk_one ubuntu
docker rm my_ubuntu_v3
docker run -it --name my_ubuntu_v4 --mount source=disk_one,target=/home/disk_one ubuntu
```

**Gán ổ đĩa vào container khi tạo container (-v)**:

Nếu muốn ổ đĩa bind dữ liệu đến một thư mục cụ thể của máy HOST thì tạo ổ đĩa với tham số như sau:
```
docker volume create --opt device=path_in_host --opt type=none --opt o=bind  DISK1
```

Sau đó ổ đĩa này gán vào container với tham số -v (không dùng --mount)
Ví dụ:

```
# Tạo ổ đĩa có tên mydisk (dữ liệu lưu tại /home/mydata)
docker volume create --opt device=/home/hung/Desktop/data_docker/ --opt type=none --opt o=bind Disk1
 
# Gán ổ đĩa vào container tại (/home/data_docker)
docker run -it -v mydisk:/home/data_docker ubuntu
```

Xóa tất cả các ổ đĩa không được sử dụng bởi container nào:
```
docker volume prune
```

## Tổng hợp
1. ánh xạ thư mục:
```
docker run -it -v /home/hung/Desktop/data_docker:/home/data_docker --name my_ubuntu ubuntu
```
2. Chia sẻ thư mục
```
docker run -it --name my_ubuntu_v2 --volumes-from my_ubuntu ubuntu
```

3. Tạo 1 ổ đĩa
```
docker volume create disk_one
docker volume inspect disk_one
```

4. Mount môt ổ đĩa 
```
docker run -it --name my_ubuntu_v3 --mount source=disk_one,target=/home/data_docker ubuntu
```

5. Gán một ổ đĩa mount tới folder của host
```
docker volume create --opt device=/home/hung/Desktop/data_docker/ --opt type=none --opt o=bind disk_two
docker volume inspect disk_two
```

6. Gán ổ đĩa vào container
```
docker run -it -v disk_two:/home/data_docker --name my_ubuntu_disk_host ubuntu
```

7. Xóa tất cả các ổ đĩa không được sử dụng bởi container nào:
```
docker volume prune
```
