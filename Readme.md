# Docker

Docker là một dự án nguồn mở nó cho phép tự động hóa việc triển khai các ứng dụng bên trong các Container (Linux), cung như cung cấp chức năng đóng gói các thành phần cần để chạy ứng dụng vào Container. Docker cung cấp công cụ CLI (Command Line Interface) để quản lý vòng đời của các container. Sử dụng Docker là cách nhanh chóng để phát triển, triển khai, bảo trì các ứng dụng.

Docker có sự khác biệt so với máy ảo, máy ảo là một hệ thống đầy đủ với tất cả các phần mềm, hệ điều hành. Các Docker Container thì cung cấp cho ứng dụng một môi trường cách ly và được cấu hình tối thiểu để ứng dụng hoạt động được. Với Container nhân và các thành phần của hệ điều hành được chia sẻ.

Một số ưu điểm của Docker Container so với công nghệ ảo hóa:

- Tạo và hủy container rất nhanh và dễ dàng, Máy áo thì cần cài đặt đầy đủ mọi thứ và cần nhiều tài nguyên hệ thống hơn.
- Container rất nhỏ, vì vậy mà trên một máy Host số container chạy song song với nhau nhiều hơn số máy áo chạy song song.

Chuyên mục này là các bài viết hướng dẫn sử dụng Docker, chú trọng vào thực hành để có thể nhanh chóng áp dụng thực tế.
1) [Cài đặt Docker, Cơ bản](1_Install.md)
2) Lệnh Docker: commit, load
3) Chia sẻ dữ liệu
4) Mạng NetWork
5) Giám sát Container
6) Dockerfile
7) Docker Compose
8) MS SQL Server Container
9) HAProxy
10) Cài đặt ownCloud
11) Chạy ứng dụng GUI
12) Docker Machine
13) Registry riêng
14) Swarm Docker
15) Swarm Docker - Ổ đĩa mạng SMB
16) XDebug trong Container PHP

## Một số lệnh Docker hay dùng
```
docker --version
# kiểm tra phiên bản docker

docker info
#Thông tin hệ thống docker

docker images -a
#Liệt kê các image

docker pull nameimage:tag
#Tải về một image từ hub.docker.com

docker ps
#Liệt kê các container đang chạy

docker ps -a
#Liệt kê các container

docker container ls -a
#Liệt kê các container
```

## Tạo / chạy container

`docker run -it --name nameyourcontainer -h "nameyourhost" image_id`: tạo, chạy một container từ image với id (name) là image_id

   Một số tham số thêm vào khi tạo container:
   - `-v path-in-host:path-in-container` :Ánh xạ thư mục máy host vào container
   - `--volumes-from other-container-name`: Nhận chia sẻ thư mục đã ánh xạ từ container khác
   - `p public-port:target-port` : Container có cổng ngoài public-port ánh xạ vào cổng trong target-port
   - `--restart=always` : Thiết lập để Docker tự khởi động container

```
docker container attach containerid
# Vào terminal container đang chạy:
    
docker exec -it containeri
# chạy một lệnh command trên container đang hoạt động  

docker stop containerid
# Dừng hoạt động một containerd command
     
docker start -i containerid
# Chạy một container
    
docker restart containe
# Khởi động lại container 
    
docker rm container
# Xóa containerrid

CTRL +P, CTRL + Q    
# Thoát -it terminal nhưng container vẫn chạyid
    

docker commit containerid imagename
# imageversion: Lưu một container đang dừng thành Image

docker save --output myimag
# Lưu image ra đĩa
    
docker load -i myimage.tar
# Nạp Image trên đĩa vào Dockere.tar myimage_id
    
docker tag image_id imagename:version
#  Đổi tên Image
    
docker network ls
# Liệt kê các networkd imagename

docker network create --driver bridge name-network
# Tạo mạng kiểu bridge đặt tên là name-network
    
docker network connect name-network name-container
# Nối container vào mạng name-network
   
docker inspect name_or_id_of_image_
# Lấy thông tin về image hoặc containerainer

docker history name_or_id_of_imag
# Lấy thông lịch sử tạo thành image

docker diff container-name-or-id
# Theo dõi thay đổi các file trên containere
    
docker logs -f container-name-or-i
# Đọc log container
    
docker stats container-name-or-id
# Đo lường thông tin

```