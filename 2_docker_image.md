# Cập nhật Image lưu Image ra file và nạp Image từ file trong Docker
Từ container cập nhật lại vào image với lệnh docker commit, cách lưu image ra file trên đĩa và nạp một file image vào Docker
```
- # Lưu container thành image
- # Lưu image ra file
```

## Lưu Container thành Image

Như đã biết từ một Image bạn có thể sinh ra các Container, mỗi Container là bản thực thi của Image, khi sử dụng Container bạn có thể cấu hình, cài đặt thêm vào nó các package, đưa thêm dữ liệu ...

Đến một lúc, bạn muốn lưu những thay đổi này và ghi lại thành một Image để sau này bạn sinh ra các Container khác bản thân nó đã chữa những thay đổi bạn đã lưu. Giả sử bạn có một container có tên (hoặc id) là mycontainer nếu muốn lưu thành image thực hiện lệnh:

```
docker commit mycontainer myimage:version
```

Trong đó myimage và version là tên và phiên bản do bạn đặt. Nếu lưu cùng tên với image tạo ra container này, coi như image cũ được cập nhật mới.


Thực hành ví dụ sau: (tạo một container hệ điều hành centos, cài đặt thêm vào nó gói SSH Client, lưu container này lại thành image, tạo một container từ image mới có)

**1. Tải hệ điều hành centos về nếu chưa có**
```
docker pull centos
```

**2. Tạo / chạy một container đặt tên là mycentos của centos**
```
docker run -it --name mycentos centos
```

**3. Cài SSH Client vào container mycentos Do centos cung cấp ở image trên không kèm theo SSH, nên muốn chạy SSH Client thì sẽ cài thêm. Giờ đang ở lệnh bash của container gõ lệnh sau để cài**
```
yum install openssh-clients
```

**4. Lưu container lại thành image: trước tiên nếu container đang chạy thì cho dừng lại**
```
docker stop mycontainer
```

Gõ lệnh lưu container mycontainer thành image đặt tên là centos-ssh:v1
```
docker commit mycentos centos-ssh:v1
```

> Như vậy bạn đã có một image mới, nó chính là centos có cài thêm ssh-client

Bạn có thể xóa đi container cũ và tạo ra container mới sinh ra từ image mới
```
#xóa container cũ
docker container rm mycentos
```

Tạo mới từ image centos-ssh:v1
```
docker run --name newcontainer -it centos-ssh:v1
```
Khi container mới tạo ra (newcontainer), chạy nó đã có sẵn SSH


## Lưu Image ra file, Nạp image từ file

Nếu muốn chia copy image ra máy khác ngoài cách đưa lên repository có thể lưu ra file, lệnh sau lưu image có tên myimage rà file
```
#lưu ra file, có thể chỉ ra đường dẫn đầy đủ nơi lưu file
docker save --output myimage.tar myimage
```

File này có thể lưu trữ, copy đến máy khác và nạp vào docker, để nạp vào docker
```
docker load -i myimage.tar
```

Đổi tên một Image đang có
```
docker tag image_id imagename:version
```
