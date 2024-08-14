# Reverse Proxy Apache_Nginx

### 1. Xây dựng mô hình reverse proxy kết hợp giữa Nginx và Apache

Yêu cầu: tạo 2 web kết hợp giữa nginx và apache, cài PHPmyadmin, mysql và PHP 8.1

- Cài đặt Apache & PHP-FPM
    
    ```jsx
    sudo apt install apache2 php-fpm
    ```
    
    - Tải cài cài đặt Module FastCGI Apache
    
    ```jsx
    wget https://mirrors.edge.kernel.org/ubuntu/pool/multiverse/liba/libapache-mod-fastcgi/libapache2-mod-fastcgi_2.4.7~0910052141-1.2_amd64.deb
    dpkg -i libapache2-mod-fastcgi_2.4.7~0910052141-1.2_amd64.deb 
    ```
    
- Cấu hình Apache và PHP-FPM
    - Backup lại file port.conf của Apache
    
    ```jsx
    sudo mv /etc/apache2/ports.conf /etc/apache2/ports.conf.bak
    ```
    
    - Tạo 1 file ports.conf với port 8080 (8080 port tùy chọn)
    
    ```jsx
    echo Listen 8080 > /etc/apache2/ports.conf
    ```
    
    - Disable site default
    
    ```bash
    sudo a2dissite 000-default
    ```
    
- Cấu hình mode_fastcgi
    - Bật mod_action và backup lại file conf
    
    ```bash
    a2enmod actions
    mv /etc/apache2/mods-enabled/fastcgi.conf /etc/apache2/mods-enabled/fastcgi.conf.default
    ```
    
    - Tạo file conf để chuyển tiếp các yêu cầu file .php sang socket UNIX PHP-FPM
    
    ```bash
    nano /etc/apache2/mods-enabled/fastcgi.conf
    
    <IfModule mod_fastcgi.c>
        AddHandler fastcgi-script .fcgi
        FastCgiIpcDir /var/lib/apache2/fastcgi
        AddType application/x-httpd-fastphp .php
        Action application/x-httpd-fastphp /php-fcgi
        Alias /php-fcgi /usr/lib/cgi-bin/php-fcgi
        FastCgiExternalServer /usr/lib/cgi-bin/php-fcgi -socket /run/php/php8.1-fpm.sock -pass-he>
        <Directory /usr/lib/cgi-bin>
            Require all granted
        </Directory>
    </IfModule>
    ```
    
    Kiểm tra lại và reload nếu Systax OK
    
    ```bash
    apachectl -t
    systemctl reload apache2
    ```
    
- Thêm source web
    
    Upload source web với đường dẫn đến docroot tùy chỉnh
    
    ```bash
    ### Wordpress ###
    /var/www/new/wordpress
    
    ###### Laravel #######
    /var/www/laravelReverseProxy
    
    systemctl start nginx mysql php8.1-fpm 
    systemctl enable nginx mysql php8.1-fpm 
    
    curl -sS https://getcomposer.org/installer | php
    sudo mv -v composer.phar /usr/local/bin/composer
    sudo chmod +x /usr/local/bin/composer
    
    cd /var/www/html
    sudo composer create-project --prefer-dist laravel/laravel laravel
    
    Output shoud be: 
    > @php artisan vendor:publish --tag=laravel-assets --ansi --force
    
       INFO  No publishable resources for tag [laravel-assets].  
    
    No security vulnerability advisories found.
    > @php artisan key:generate --ansi
    
       INFO  Application key set successfully. 
       
    ### phân quyền lại ###
    sudo chown -R www-data: /var/www/laravelReverseProxy/laravel
    sudo chmod -R 775 /var/www/laravelReverseProxy/laravel
    
    cd laravel
    php artisan key:generate
    
    ### chỉnh sửa data phù hợp ###
    nano /var/www/laravelReverseProxy/laravel/.env
    ```
    
- Tạo Vhost cho Apache
    
    Tạo 2 Vhost cho 2 web wordpress và apache
    
    ```bash
    ###  Laravel  ###
    vi /etc/apache2/sites-available/phuongduy-laravel.vietnix.xyz.conf 
    
    <VirtualHost *:8080>
            DocumentRoot /var/www/laravelReverseProxy/laravel/public
            ServerName phuongduy-laravel.vietnix.xyz
            
            <Directory /var/www/laravelReverseProxy/laravel>
            AllowOverride All
            Require all granted
            </Directory>
    
            ErrorLog ${APACHE_LOG_DIR}/error.log
            CustomLog ${APACHE_LOG_DIR}/access.log combined
    
    <FilesMatch \.php$>
            SetHandler "proxy:unix:/var/run/php/php8.1-fpm.sock|fcgi://localhost"
    </FilesMatch>
    
    </VirtualHost>
    
    ### Wordpress ###
    vi /etc/apache2/sites-available/phuongduy.vietnix.xyz.conf 
    
    <VirtualHost *:8080>
            DocumentRoot /var/www/new/wordpress
            ServerName phuongduy.vietnix.xyz
            
            <Directory /var/www/new/wordpress>
            AllowOverride All
            Require all granted
            </Directory>    
    
            ErrorLog ${APACHE_LOG_DIR}/error.log
            CustomLog ${APACHE_LOG_DIR}/access.log combined
    
    <FilesMatch \.php$>
            SetHandler "proxy:unix:/var/run/php/php8.1-fpm.sock|fcgi://localhost"
    </FilesMatch>
    
    </VirtualHost>
    
    ```
    
    - Vhost chạy trên port 8080
    - **Document root**: nơi chứa source web
    - **Servername**: tên miền của web
    - **<Directory … /Directory>**: áp dụng qui tắc lên đường dẫn chỉ định bên trong
        - **AllowOverride All**: cho phép cho phép bật tính năng hỗ trợ .htaccess.
        - **Require all granted**: cho phép tất cả các yêu cầu truy cập vào thư mục được chỉ định.
    - **<FilesMatch \.php$> … </FilesMatch>**: áp dụng qui tắc lên các file .php
    - **SetHandler "proxy:unix:/var/run/php/php8.1-fpm.sock|fcgi://localhost"**: các file php được PHP-FPM (FastCGI) xử lý, giúp tăng hiệu suất hơn.
    
- Cài đặt và cấu hình Nginx
    
    ```bash
    apt install nginx
    
    ### xóa default (option)
    rm /etc/nginx/sites-enabled/default
    ```
    
    - Tạo file cấu hình
    
    ```bash
    ### laravel ###
    vi /etc/nginx/conf.d/phuongduy-laravel.vietnix.xyz.conf (tùy chỉnh)
    
    server {
            listen 80;
            server_name phuongduy-laravel.vietnix.xyz;
            return 301 https://$host$request_uri;
    }
    
    server {
            listen 443 ssl;
            server_name phuongduy-laravel.vietnix.xyz;
            ssl_certificate /home/phuongduy-laravel.vietnix.xyz.crt;
            ssl_certificate_key /home/phuongduy-laravel.vietnix.xyz.key;
            ssl_trusted_certificate /home/phuongduy-laravel.vietnix.xyz_ca.crt;
    
        # Xử lý các file tĩnh
            location ~* \.(html|css|js|jpg|jpeg|png)$ {
            root /var/www/laravelReverseProxy/laravel;
        }
    
            location / {
                    proxy_pass http://14.225.253.130:8080;
                    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                    proxy_set_header X-Forwarded-Proto https;
                    proxy_set_header Host $host;
                    }
    }
    
    ### wordpress ###
    server {
            listen 80;
            server_name phuongduy.vietnix.xyz;
            return 301 https://$host$request_uri;
    }
    
    server {
            listen 443 ssl;
            server_name phuongduy.vietnix.xyz;
            ssl_certificate /home/phuongduy.vietnix.xyz.crt;
            ssl_certificate_key /home/phuongduy.vietnix.xyz.key;
            ssl_trusted_certificate /home/phuongduy.vietnix.xyz_ca.crt;
    
        # Xử lý các file tĩnh
            location ~* \.(html|css|js|jpg|jpeg|png|gif|ico|woff|woff2|ttf|svg)$ {
            root /var/www/new/wordpress;
        }
    
            location / {
                    proxy_pass http://14.225.253.130:8080;
                    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                    proxy_set_header X-Forwarded-Proto https;
                    proxy_set_header Host $host;
                    }
    }
    ```
    
    - **listen 80:** hoạt động trên port 80
    - **servername:** tên miền của web
    - **return 301 https://$host$request_uri**: chuyển hướng sang https
    - **Xử lý các file tĩnh**
        - locationlocation ~* \.(…)$ định nghĩa các file với định dạng tĩnh
        - root vị trí Nginx sẽ tìm thấy file các file này
    - **Xử lý các file còn lại**
        - proxy_pass: chuyển hướng đến IP server và port tương ý
        - X-Forwarded-For: là 1 header  để ghi lại IP của client
        - $proxy_add_x_forwarded_for: là biến trong Nginx bao gồm địa chỉ IP của client và tất cả các giá trị `X-Forwarded-For` đã tồn tại từ các proxy khác. Điều này đảm bảo rằng tất cả các proxy trung gian trong chuỗi yêu cầu đều được ghi lại.
        - X-Forwarded-Proto https: là 1 header dùng để chỉ định dùng https
        - `Host` là một header HTTP tiêu chuẩn, được trình duyệt gửi trong mỗi request HTTP. Header này chứa domain và có thể bao gồm cả port mà người dùng đã truy cập.
            
            `$host` lấy giá trị header từ Host 
            

### 2. Cài đặt phpmyadmin

```bash
apt update

apt install phpmyadmin php-mbstring php-zip php-gd php-json php-curl

systemctl restart apache2
```

- Cấu hình thêm 1 vhost cho phpmyadmin

```bash
### vào tạo file cấu hình cho vhost phpmyadmin ###
vi /etc/nginx/conf.d

server {
    listen 80;
    server_name 14.225.253.130;

    location /phpmyadmin/ {
        alias /usr/share/phpmyadmin/;
        index index.php;
        proxy_pass http://14.225.253.130:8080/phpmyadmin/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Các thành phần tương tụ các file wordpress, laravel ở trên.

Nhìn chung cấu hình này phục vụ các yêu cầu đến phpMyAdmin thông qua proxy Nginx. Khi người dùng truy cập `http://14.225.253.130/phpmyadmin/`, Nginx sẽ chuyển tiếp yêu cầu đó đến Apache trên cổng 8080.