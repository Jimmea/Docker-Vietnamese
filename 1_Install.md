# Cài đặt docker trên hệ điều hành
Hiện giờ Docker có đầy đủ bản cài đặt trên Windows, MacOS, Linux. Tùy thuộc bạn muốn chạy trên môi trường nào thì tải về bản cài đặt tương ứng để cài đặt.

## Cài Docker trên macOS
Tải bộ cài tại [Docker Desktop for Mac](https://hub.docker.com/editions/community/docker-ce-desktop-mac), cài đặt đơn giản như các công cụ thông thường.

## Cài Docker trên Windows 10
*Đảm bảo hệ thống là Windows 10 Pro*

Tải bộ cài tại Docker Desktop for Windows, tiến hành cài đặt. Đối với Windows phải kích hoạt chế độ Hyper-V virtualization (Ở chế độ này bạn không dùng được VirtualBox nữa).

> Nếu chưa kích hoạt Hyper-V thì kích hoạt theo hướng dẫn Enable Hyper-V
  

Chạy lệnh PowerShell sau để kích hoạt:

```
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All
```

**Kích hoạt thông qua thiết lập Windows**
1. Nhấn phải chuột vào biểu tượng cửa sổ, chọn Apps and Features
2. Chọn Programs and Features
3. Chọn Turn Windows Features on or off
4. Đánh dấu vào Hyper-V như hình dưới

![hyper-v](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/quick-start/media/enable_role_upd.png)

## Cài Docker trên Ubuntu
   
Chạy các lệnh để cài đặt:

```
sudo apt update
sudo apt install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"
sudo apt update
apt-cache policy docker-ce
sudo apt install docker-ce
sudo systemctl status docker
```

Sau khi cài đặt, bạn có thể cho user hiện tại thuộc group docker, để khi gõ lệnh không cần xin quyền sudo

```
sudo usermod -aG docker $USER
```

Logout sau đó login lại để có hiệu lực.

Ngoài ra khi sử dụng đến thành phần ```docker-compose``` thì bạn cài thêm
```
sudo apt install docker-compose
```

Nếu có nhu cầu sử dụng Docker Machine trên Ubuntu, (công cụ tạo - quản lý các máy ảo chạy Docker Engine, các máy ảo này tạo bởi VirtualBox, bạn sử dụng Docker-machine để thực hành các ví dụ kết nối nhiều máy chạy Docker Engine khác nhau tạo thành cụm Server), thì cài thêm Docker Machine. Phiên bản mới nhất lấy tại [Docker Machine](https://github.com/docker/machine/releases)

Ví dụ cài bản v0.16.1, gõ lệnh sau:

```
curl -L https://github.com/docker/machine/releases/download/v0.16.1/docker-machine-`uname -s`-`uname -m` >/tmp/docker-machine &&
chmod +x /tmp/docker-machine &&
sudo cp /tmp/docker-machine /usr/local/bin/docker-machine
```

Tất nhiên là Docker Machine cần VirtualBox để làm việc, nếu chưa có thì cài thêm

```
sudo apt install virtualbox
```

##  Cài Docker trên CentOS7/RHEL7
```
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo yum install docker-ce
sudo usermod -aG docker $(whoami)
sudo systemctl enable docker.service
sudo systemctl start docker.service

#Cài thêm Docker Compose
sudo yum install epel-release
sudo yum install -y python-pip
sudo pip install docker-compose
sudo yum upgrade python*
docker-compose version
```


## Cài Docker trên CentOS8/RHEL8
   
```
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo yum install docker-ce

# Nếu lỗi, chạy lệnh sau
sudo dnf install --nobest docker-ce -y


sudo usermod -aG docker $(whoami)
sudo systemctl enable docker.service
sudo systemctl start docker.service

#Cài thêm Docker Compose
sudo yum install epel-release
sudo yum -y install python2-pip
sudo pip2 install docker-compose
sudo yum upgrade python*
docker-compose version
```

## Làm quen ban đầu Docker
Giờ hãy mở giao diện lệnh `termial` (macOS, Linux) hay `PowerShell` của Windows nên và gõ:

**Kiểm tra phiên bản Docker**
```
docker --version
```

Hoặc lệnh thông tin chi tiết hơn:
```
docker info
```

**image và container trong Docker**

`image` là một gói phần mềm trong đó chứa những thứ cần như thư viện, các file cấu hình, biến môi trường để chạy mội ứng dụng nào đó. Nó giống như cái USB chứa bộ cài đặt hệ điều hành Windows!


Khi một phiên bản của `image` chạy, phiên bản chạy đó gọi là container - (vậy muốn có container phải có image). Bất ký lúc nào bạn cũng có thể kiểm tra xem có bao nhiêu container đang chạy và nó sinh ra từ image nào.

**Tải về một image**

Kiểm tra các image bạn đang có

```
docker images -a
```
- `Repository` tên image trên kho chứa, ví dụ hệ điều hành CentOS có tên trên hub.docker.com là `centos` ([xem centos](https://hub.docker.com/_/centos))
- `TAG` là phiên bản image, với giá trị `latest` có nghĩa là bản cuối. Muốn tải về bản khác latest vào mục TAGS trên hub.docker.com tìm bản phù hợp.
- `IMAGE` ID một chuỗi định danh duy nhất (tên) của image trên hệ thống của bạn.

Để tải về một image nào đó, dùng lệnh

```
docker pull nameimage:tag

#Hoặc tải về bản cuối
docker pull nameimage
```

Bây giờ hệ thống không có hệ điều hành Ubuntu, muốn tải nó về - vì trên hub.docker.com nó có tên ubuntu, ta sẽ tải về bản mới nhất bằng lệnh:
```
docker pull ubuntu
```
Giờ thì hệ thống sẽ có thêm image với tên ubutu, có thể góc lại lệnh kiểm tra docker images -a ở trên để xem.

## Tạo và chạy container
Như trên, container là một phiên bản chạy của image, trước tiên

Kiểm tra có các container nào đang chạy:
```
docker ps
```

Liệt kê tất cả các container:
```
docker container ls --all
```

Các container bạn đã tạo, liệt kê ra hãy chú ý mấy thứ

- `CONTAINER` ID một con số (mã hash) gán cho container, bạn dùng mã này để quản lý container này, như xóa bỏ, khởi động, dừng lại ...
- `IMAGE` cho biết `container` sinh ra từ image nào.
- `COMMAND` cho biết lệnh, ứng dụng chạy khi container chạy (/bin/bash là terminate)
- `STATUS` cho biết trạng thái, (exit - đang dừng)

Để tạo và chạy một container theo cú pháp:

```
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```
Các tham số đó là:
- `[OPTIONS]` các thiết lập khi tạo container, có rất nhiều thiết lập tùy mục đích tạo container, sẽ tìm hiểu các thiết lập qua từng trường hợp cụ thể.
- `IMAGE` tên image hoặc ID của image từ nó sinh ra container.
- `[COMMAND] [ARG...]` lệnh và tham số khi container chạy.

Trở lại phần trên, đang có image hệ điều hành Ubuntu, giờ muốn chạy nó (tạo một container từ nó), hãy gõ lệnh sau:

```
docker run -it ubuntu
```

Bạn chú ý có tham số -it để khi container chạy, bạn có terminate làm việc ngay với Ubuntu. Tham số này có nghĩa là
- `-t` nó có nghĩa là console, cho phép kết nối với terminal để tương tác
- `-i` có nghĩa duy trì mở stdin để nhập lệnh.

Sau lệnh này, bạn đang chạy Ubuntu, bạn có thể gõ vài lệnh Ubuntu kiểm tra xem. Ví dụ kiểm tra phiên bản Ubuntu, xem hình dưới với lệnh `cat /ect/*release.` 
Như vậy bạn đã tạo một container

Bạn đang ở `terminal` của Ubuntu, có thể kết thúc bằng lệnh `exit`, hoặc nhấn nhanh `CTRL+P,CTRL+Q` để thoat `termial` nhưng `container` vẫn đang chạy.

**Một vài tham số khác:**

Tạo và chạy container, container tự xóa khi kết thúc thì thêm vào tham số `--rm`
```
docker run -it --rm ubuntu
```

Tạo và chạy container - khi container chạy thi hành ngay một lệnh nào đó, ví dụ `ls -la`
```
docker run -it --rm debian ls -la
```

Tạo và chạy container - ánh xạ một thự mục máy host vào một thư mục container, chia sẻ dữ liệu

```
docker run -it --rm -v path-in-host:path-in-container debian
```

**Vào container đang đang chạy**

Kiểm tra xem bằng lệnh docker ps, nếu có một container với ID là containerid đang chạy, để quay quay trở lại terminal của nó dùng lệnh:

```
docker container attach containerid
```

Chạy một container đang dừng

```
docker container start -i containerid
```

Nếu cần xóa bỏ hẳn một container thì dùng lệnh
```
docker container rm containerid
OR
docker rm containerid
# DOcker container running 
OR Docker rm containerid -f 
```

## Tóm tắt các lệnh làm quen

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