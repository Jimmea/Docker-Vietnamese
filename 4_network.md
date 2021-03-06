#Mạng network bridge trong Docker kết nối các container với nhau
Cách tạo một network trong Docker, thực hành cài đặt Apache, PHP, MySQL và kết nối chúng vào mạng, cài đặt WordPress trên Docker

1. Liên kết mạng các Container
2. Cài đặt PHP
3. Cài đặt Apache
4. Liên kết Apache MySQL với nhau
5. Cài đặt MySQL
6. Cài đặt Wordpress

## Network trong Docker, liên kết mạng các container

Docker cho phép bạn tạo ra một network (giao tiếp mạng), sau đó các container kết nối vào network. Khi các container cùng một network thì chúng có thể liên lạc với nhau nhanh chóng qua tên của container và cổng (port) được lắng nghe của container trên mạng đó.


Để liệt kê các network đang có:


````
docker network ls
````

Các network được tạo ra theo một driver nào đó như `bridge`, `none`, `overlay`, `macvlan`. Trong phần này sẽ sử dụng đến bridge network: nó cho phép các container cùng network này liên lạc với nhau, cho phép gửi gói tin ra ngoài. Tạo một `bridge network` với tên network là `name-network`.

```
docker network create --driver bridge name-network
```

Khi tạo một container, ấn định nó nối vào network có tên là `name-network` thì thêm vào tham số `--network name-netowrk` trong lệnh docker run. Khi một container có tên `name-container` đã được tạo, để thiết lập nó nối vào network có tên `name-network`.

```
docker network connect name-network name-container
```

**Cổng - Port**

Các kết nối mạng thông qua các cộng, để thiết lập cấu hình các cổng của container chú ý: có cổng bên trong container, có cổng mở ra bên ngoài (public), có giao thức của cổng (tpc, udp). Khi chạy container (tạo) cần thiết lập cổng thì đưa vào lệnh `docker run` tham số dạng sau:

```
docker run -p public-port:target-port/protocol ... 
```

- `public-port`: cổng public ra ngoài (ví dụ 80, 8080 ...), các kết nối không cùng network đến container phải thông qua cổng này.
- `target-port`: cổng bên trong container, cổng public-port sẽ ánh xạ vào cổng này. Nếu các container cùng network có thể kết nối với nhau thông qua cổng này.

Sau đây sẽ thực hành tạo ra 3 container, chạy 3 dịch vụ khác nhau là `httpd, php-fpm, mysql`. Tạo một mạng `bridge` để liên kết những container này với nhau (giao tiếp với nhau qua cổng nội bộ `httpd:80, php-fpm:9000, mysql:443`). Trong đó thiết lập thêm httpd mở cổng public 8080 để ánh xạ vào 80 của nó.

## Cài đặt PHP trên Docker

Chọn cài đặt php:7.3-fpm (các phiên bản khác tương tự) đây là PHP 7.3 cài đặt sẵn PHP-FPM, mặc định cho phép apache gọi đến PHP thông qua proxy với cổng 9000,


Tạo một container chạy PHP từ image php:7.3-fpm, đặt tên container này là c-php

```
docker run -d --name c-php -h php -v "/mycode/php":/home/phpcode php:7.3-fpm
```

Các tham số tạo, chạy container như đã biết trong phần trước, ở đây cụ thể là:
- `-d` : container sau khi tạo chạy luôn ở chế độ nền.
- `--name c-php `: container tạo ra và đặt luôn cho nó tên là c-php (Tương tác với container dễ đọc hơn khi sử dụng tên thay vì phải sử dụng mã hash id, nếu không đặt tên thì docker sinh ra tên ngẫu nhiên).
- `-h php` đặt tên HOSTNAME của container là php
- `-v "/mycode/php":/home/phpcode` thư mục trên máy host /mycode/php (với Windows đường dẫn theo OS này như "c:\path\mycode\php") được gắn vào container ở đường dẫn /home/phpcode.
- `php:7.3-fpm` là image khởi tạo ra container, nếu image này chưa có nó tự động tải về.

Sau lệnh này, một container đã tạo container có tên đặt là c-php từ image php:7.3-fpm. Hãy kiểm tra tình hình các container xem sao với lệnh:


```
docker ps -a
```

Như câu lệnh, một container có tên c-php được tạo ra, chạy với lệnh script mặc định `docker-php-entrypoint`
, dịch vụ `PHP-FPM` chạy và lắng nghe ở cổng `9000`. Script mặc định này mô tả trong một file `Dockerfile`, loại file này sẽ tìm hiểu sau. Ở đây bạn có thể xem thông tin file này kèm trong image bằng lệnh.

```
docker inspect image_name_or_id
```

```
docker inspect php:7.3-fpm
```

Trong image `php:7.3-fpm` thì đọc thông tin này, có mục:

```
"Entrypoint": [ "docker-php-entrypoint" ],
```

Vì `Entrypoint` quy định các lệnh chạy mặc định khi tạo - chạy container, nên bạn thấy script docker-php-entrypoint được chạy.

> Chú ý: trong một số image, Entrypoint chạy script kết thúc ngay dẫn đến container kết thúc ngay, kể cả khi khởi chạy lại container bằng docker start thì nó cũng kết thúc luôn. Những loại image như vậy, cần tiến trình hoạt động dài để giữ cho container sống. Để khắc phục, thêm vào nó tham số -i ở lệnh docker run
  

Giờ nếu muốn nhảy vào tương tác với container này (container phải đang chạy), thì gọi lệnh bash của nó bằng cách:

```
docker exec -it c-php bash
```

Đang trong container, hãy gõ lệnh kiểm tra php version:

```
php -v
```

Kết quả hiện thị thông tin phiên bản PHP, vậy bạn đã có một container PHP rồi!. Thử tạo một script PHP có tên test.php và chạy nó từ dòng lệnh của container này.

```
cd /home/phpcode/
echo "<?php echo 'Hello World';" > test.php
php test.php   #chạy test.php

#Kết quả: Hello World!
```

Bạn có thể copy, chỉnh sửa các file theo đường dẫn của máy Host /mycode/php và nó truy cập được từ containr trong đường dẫn /home/phpcode/

**Mở rộng**: có sẵn một script có tên là `docker-php-ext-configure` và `docker-php-ext-install` để bạn cấu hình, cài các Extension cho PHP, ví dụ nếu muốn có thêm OpCache thì gõ:

```
docker-php-ext-configure opcache --enable-opcache \
    && docker-php-ext-install opcache 
```

docker-php-ext-configure opcache --enable-mysqli \

```
docker-php-ext-configure opcache --enable-mysqli \
    && docker-php-ext-install mysqli
```


Cài thêm pdo_mysql - extension để kết nối PHP đến MySQL với thư viện PDO

```
docker-php-ext-configure opcache --enable-pdo_mysql \
    && docker-php-ext-install pdo_mysql 
```

File php.ini mà PHP nạp nằm ở /usr/local/etc/php, để có file ini này gõ lệnh:

```
cp /usr/local/etc/php/php.ini-production /usr/local/etc/php/php.ini 
```


Để có trình soạn thảo text nano gõ lệnh:
```
apt-get update && apt-get install nano
```

Nếu cần chỉnh sửa thiết lập nào đó, vào ini chỉnh sủa, gõ lệnh:

```
nano /usr/local/etc/php/php.ini
```

Khi cài đặt extension, thay đổi thiết lập để hiệu lực hãy khởi động lại container này bằng lệnh:

```
docker restart c-php
```


> Kiểm tra các phần mở rộng đã có trong PHP bằng lệnh php -m, cái nào cần dùng còn thiếu thì cài vào theo cách như trên.




## Cài đặt APACHE HTTPD trên Docker

Apache là máy chủ HTTPD rất phổ biến, nhất là các server Linux. Tải về httpd bản mới nhất:

```
docker pull httpd
```

Tạo một container httpd đặt tên là `c-httpd`

```
docker run -di -p 8080:80 -p 443:443 --name c-httpd -h httpd -v "/mycode/php":/home/phpcode httpd
```

Trong lệnh trên, có tham số -p ` 8080:80` để thiết lập cổng cho container, nếu không thiết lập thì mặc định một container không mở cổng nào cả, với thiết lập trên - khi truy cập cổng 8080 trên HOST, sẽ ánh xạ vào cổng 80 của container, cổng mà máy chủ httpd đang lắng nghe.

Giờ từ trình duyệt, vào địa chỉ `httpd://localhost:8080` thấy có kết quả

![result](https://raw.githubusercontent.com/xuanthulabnet/swift-learning/master/images/docker/docker-httpd.png)

Vậy đã có một container - httpd đang chạy!

Muốn chỉnh sửa file config` httpd.conf`
- File config tại: `/usr/local/apache2/conf/httpd.conf`
- Nếu muốn cài trình soạn thảo nano gõ lệnh `apt-get update & apt-get install nano`

Vậy để soạn thảo config bạn gõ (nếu chưa có nano thì cài vào như ở trên) `nano /usr/local/apache2/conf/httpd.conf`

### Thiết lập PHP Handler
    
 ứng dụng PHP nằm ở container c-php, tên này nếu cũng dùng để container khác cùng mạng liên lạc đến mà không cần dùng IP cụ thể, (Ở phần sau sẽ thiết lập 2 container này cùng nối vào một network). Vì vậy, thêm vào httpd.conf dòng cấu hình sau để khi truy vấn đến file .php thì nó sẽ chạy PHP từ Proxy:

```
AddHandler "proxy:fcgi://c-php:9000" .php
```

Do Handler này gọi PHP thông qua proxy, nên Apache phải kích hoạt module liên quan đến proxy, mở file `httpd.conf` và bỏ commnet trên các dòng:
```
LoadModule proxy_module modules/mod_proxy.so
    LoadModule proxy_fcgi_module modules/mod_proxy_fcgi.so
```

Sửa xong khởi động lại apache:

```
apachectl -k restart
```

### Liên kết Apache và PHP-FPM

Để hai container `c-php` và `c-httpd` liên lạc được với nhau (ví dụ từ httpd với `proxy:fcgi://c-php:9000`), thì ta thiết lập chúng cùng một mạng. Trước tiên, kiểm tra xem có những mạng nào!

```
docker network ls
```


Giờ ta sẽ tạo ra một mạng với Driver là bridge đặt tên là www-net (cùng cấu hình mạng Host)

```
docker network create --driver bridge www-net
```

Danh sách mạng của Docker đã có mạng www-net vừa tạo ra. Giờ ta thiết lập container c-php và c-httpd nối vào mạng này.

```
docker network connect www-net c-php
docker network connect www-net c-httpd
```

Kiểm tra thông tin mạng www-net

```
docker network inspect www-net
```

Thấy có đoạn thông tin

```
"Containers": {
    "a0112f016627ee477376bdbd93fdf2948d55b54d2326fb488865592a99f3be1c": {
        "Name": "c-httpd",
        "EndpointID": "bd2e5af336ddb27891bb130c53c6707f4090a0ffcf67068d0f4762cd45eeafdb",
        "MacAddress": "02:42:ac:12:00:03",
        "IPv4Address": "172.18.0.3/16",
        "IPv6Address": ""
    },
    "c4dd6ecd43b48d47653fed19bc7fea483c378f807bf08b6fda42ac7b7d35fcff": {
        "Name": "c-php",
        "EndpointID": "2464bdc048650e87484c89a73f04f7b582a6ad2840f7d7095ffe5360a0e572ff",
        "MacAddress": "02:42:ac:12:00:02",
        "IPv4Address": "172.18.0.2/16",
        "IPv6Address": ""
    }
},
```

Như vậy 2 container trên đã cùng một network với địa chỉ IP cho từng container. Hai container này đã có thể liên lạc với nhau qua giao tiếp mạng, bạn có thể kiểm tra bằng cách vào container này và dùng lệnh ping đến IP của container kia.

Chú ý để có lệnh ping cần cài đặt

```
apt-get update && apt-get install iputils-ping -y
```

Đến đây thì đã Apache đã gọi được PHP, ta sẽ thử cấu hình và chạy một file php. Trước tiên, mở httpd.conf và chỉnh sửa thư mục gốc mặc định nằm ở ` /home/phpcode`. Tìm đến:
```
DocumentRoot "/usr/local/apache2/htdocs"
<Directory "/usr/local/apache2/htdocs">
```

Thay bằng:
```
DocumentRoot "/home/phpcode"
<Directory "/home/phpcode">
```


Hãy tạo file /home/phpcode/test.php
```
nano /home/phpcode/test.php
```

Nhập vào nội dung:
```
<php
    phpinfo();
```

Lưu lại, và từ máy Host vào trình duyệt với địa chỉ, `http://localhost:8080/index.php`, thấy nội dung sau:
![result](https://raw.githubusercontent.com/xuanthulabnet/swift-learning/master/images/docker/docker-php-apache.png)

Như vậy đã hoàn thành thiết lập Apache - PHP trên 2 container khác nhau, chia sẻ dữ liệu và cùng một mạng

## Cài đặt MySQL

Trong phần này sẽ tạo và cài đặt mysql trên một container để cuối cùng có bộ ba container Apache HTTP - PHP 7.3 - MySQL 8, như vậy có một webserver hoàn chỉnh để cài đặt, phát triển các ứng dụng web



Tạo, chạy container từ image `mysql` (image này nếu chưa có ở local, nó sẽ tự tải về), đặt tên là `c-mysql`
```
docker run -it --network www-net --name c-mysql -h mysql \
        -v "/mycode/db":/var/lib/mysql -e MYSQL_ROOT_PASSWORD=abc123  mysql
```

Các tham số của lệnh trên:

- `--network www-net` container sẽ kết nối với mạng có tên www-net (cùng mạng với c-php, c-httpd)
- `-e MYSQL_ROOT_PASSWORD=abc123` đặt password cho user root, quản trị mysql là abc123
- `-v "/mycode/db":/var/lib/mysql` nơi lưu trữ các database là ở thư mục `/mycode/db` của máy Host, làm điểu này để có không mất db khi cần xóa container, hoặc khi cần sử dụng lại các db cũ.


Sau lệnh này, bạn đang chạy MySQL cùng mạng với PHP, APACHE và cổng mở là 3306, file cấu hình ở /etc/mysql/my.cnf, có thể thoát khỏi terminal đang chạy với tổ hợp phím CTRL + P, CTRL + Q


Đây là mysql 8, mặc định không sử dụng plugin mysql_native_password, để các ứng dụng thông dụng kết nối dễ dàng tương thích bạn có thể chỉnh file cấu hình để sử dụng loại plugin này.


Trước tiên vào lại container c-mysql cài đặt trình soạn thảo nano

```
docker exec -it c-mysql bash
apt-get update
apt-get install nano
```


Mở file `my.cnf`
```
nano /etc/mysql/my.cnf
```

Thêm vào nội dung:

```
[mysqld]
default-authentication-plugin=mysql_native_password
```

Ra khỏi container và khởi động lại `docker restart c-mysql`, giờ thì đã có server MySQL đang chạy, các ứng dụng cùng mạng muốn sử dụng có thể kết nối đến nó qua cổng 3306

```
docker exec -it c-mysql bash                       #nhảy vào container
mysql -uroot -pabc123                              #kết nối vào MySQL Server

#Từ dấu nhắc MySQL, Tạo một user tên testuser với password là testpass
CREATE USER 'testuser'@'%' IDENTIFIED BY 'testpass';

#Tạo db có tên db_testdb
create database db_testdb;

#Cấp quyền cho user testuser trên db - db_testdb
GRANT ALL PRIVILEGES ON db_testdb.* TO 'testuser'@'%';
flush privileges;


show database;            #Xem các database đang có, kết quả có db bạn vừa tạo
exit;                     #Ra khỏi MySQL Server
```

Đến đây đã hoàn hàm xây dựng xong các Container với 3 thành phần: `APACHE HTTPD - MYSQL - PHP` để phát triển và chạy các ứng dụng Web. Sau đây sẽ là ví dụ cài đặt `Wordpress`

## Cài đặt Wordpress trên Docker
Ta sẽ cấu hình để chạy một tên miền ảo, ví dụ này chọn `mywordpressblog.com`, trước tiên vào chỉnh file host của máy tại `/etc/host` (trên Windows tại `C:\Windows\System32\drivers\etc\hosts`), thêm vào:
```
127.0.0.1	mywordpressblog.com
```

Tải Wordpress (Download Wordpress) về, giải nén vào thư mục máy host /mycode/php/blog (Tương ứng chính là thư mục /home/phpcode/blog trong container). Tiếp theo vào container c-httpd, tạo một VirtualHost cho tên miền này, cấu hình để thư mục làm việc là /home/phpcode/blog/:

Mở `httpd.conf` thêm vào cuối:
<VirtualHost *:80>
    ServerName mywordpressblog.com
    ServerAdmin hungokata@gmail.com
    DocumentRoot /home/phpcode/blog/
    CustomLog /dev/null combined
    #LogLevel Debug
    ErrorLog /home/phpcode/blog/error.log
    <Directory /home/phpcode/blog/>
        Options -Indexes -ExecCGI +FollowSymLinks -SymLinksIfOwnerMatch
        DirectoryIndex index.php
        Require all granted
        AllowOverride None
    </Directory>
</VirtualHost>

Lưu lại và khởi động lại `c-httpd` (apache).
Vào container `c-mysql` để tạo một database cho user `testuser` làm database wordpress.

```
docker exec -it c-mysql bash
mysql -uroot -pabc123
create database wp_blog;
GRANT ALL PRIVILEGES ON wp_blog.* TO 'testuser'@'%';
flush privileges;
```

Tạo xong, ra khỏi container c-mysql

Trong code Wordpress tải về, đổi tên file `wp-config-sample.php thành wp-config.php`, rồi chỉnh các thông tin thành như sau:

```
/**Tên database đã tạo wp_blog*/
define( 'DB_NAME', 'wp_blog' );

/** MySQL database username: user đã tạo ở c-mysql */
define( 'DB_USER', 'testuser' );

/** MySQL database password: password này đã tạo ở c-mysql */
define( 'DB_PASSWORD', 'testpass' );

/** MySQL hostname: tên của container chứa MySQL Server */
define( 'DB_HOST', 'c-mysql' );
```


Bắt đầu cài đặt, vào trình duyệt gõ địa chỉ http://mywordpressblog.com:8080

Sau khi cài đặt, truy cập địa chỉ http://mywordpressblog.com:8080

Vậy là một trang WordPess đã hoạt động thành công, điều đó chứng tỏ 3 container độc lập ở trên đã kết nối lại với nhau chạy ứng dụng. Mặc dùng có thể dùng đến file Dockerfile, quá trình tạo các container như trên nhanh và đơn giản hơn, nhưng ở thời điểm đang tìm hiểu hãy thực hiện thủ công như vậy sẽ nắm rõ cách thức hoạt động hơn.

## Tổng kết


## Reference
https://xuanthulab.net/mang-network-bridge-trong-docker-ket-noi-cac-container-voi-nhau.html#mysql 
