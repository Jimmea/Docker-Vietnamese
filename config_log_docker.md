# Thiết lập cấu hình log trong Docker
Cấu hình log cho các container và docker daemon, tránh log quá lớn chiếm hết dung lượng đĩa

Mỗi Docker daemon có một driver log, để thực hiện cơ chế ghi nhận log cho hoạt động của Docker daemon cũng như log của các container. Mặc định cấu hình log của container sử dụng giống của Docker daemon trừ trường hợp bạn cấu hình riêng cho từng container.

Như vậy các log trong container được cấu hình về giới hạn, kích thước bởi Docker, một file log có thể lớn dần lên đến mức có thể chiếm hết cả ổ cứng. Do vậy, bạn cũng cần lưu ý về cấu hình log cho Docker và các Container.


## Cấu hình Log cho Docker Daemon
Mở file `daemon.json` ra để thiết lập, nếu chưa có file này thì tạo mới nó nằm ở đường dẫn: `/etc/docker/daemon.json`, trên Windows thì ở đường dẫn `C:\ProgramData\docker\config\daemon.json`


Bạn có thể biên tập nội dung của file này như sau:
```
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3",
    "labels": "production_status",
    "env": "os,customer"
  }
}
```
Cấu hình trên mặc định thiết lập driver sử dụng xuất ra file log dạng json, kích thước tối đa của mỗi file là 10m và cho phép lưu tối đa 3 file log. Bạn chỉ việc điều chỉnh cỡ theo nhu cầu và khởi động lại Docker

```sh
systemctl restart docker
```

## Cấu hình Log cho Container
Bạn có thể thiết lập cho Container sử dụng riêng cấu hình lưu log, sử dụng bằng tham số khi chạy container bằng lệnh docker run như tham số:

`--log-opt max-size	`: Chỉ ra cơ lớn nhất của 1 file log (mặc định là 20m), bạn có thể sử dụng các đơn vị k, m, g
`--log-opt max-file	`: chỉ ra số lượng file log, mặc định là 5

Ví dụ:
```sh
docker run -it --log-opt max-size=2m --log-opt max-file=2 ubuntu
```

Bạn có thể thực hiện thiết lập trong docker-compose.yaml, ví dụ:

```
version: '3'

services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.8.0
    container_name: elasticsearch
    ... 
    logging:
      options:
        max-size: "1m"
        max-file: "2"

# networks: ...
# volumes: ...
```
