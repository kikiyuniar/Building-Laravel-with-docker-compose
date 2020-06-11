# Building-Laravel-with-docker-compose
![image laravel with docker](https://1.bp.blogspot.com/-y3LjMdXNZ5E/XuJt4xV0ClI/AAAAAAAASrg/ctGSeB6tmHcQT1RlWNm_YQqSaexD50eXQCLcBGAsYHQ/s1600/docker-laravel.jpg = 150x)
build the Laravel Framework with docker-compose and make docker management.

Post in Linkedin : https://www.linkedin.com/in/kikiyuniar/

Langkah memasang portainer dan docker-compose
* install [portainer](https://www.portainer.io/) di dalam satu container dan docker-compose.
example:

```html  
$ docker run -d -p 860:9000 -v /var/run/docker.sock:/var/run/docker.sock portainer/portainer
$ sudo apt install docker-compose
```
>catatan: 860 merupakan port yang akan di akses untuk membuka halaman portainer yang telah terpasang

### Step 1 Downloading Laravel and Installing Dependencies
```html  
$ cd ~
$ git clone https://github.com/laravel/laravel.git laravel-app
$ docker run --rm -v $(pwd):/app composer install
$ sudo chown -R $USER:$USER ~/laravel-app
```
### Step 2 Creating the Docker Compose File
```html
$ nano ~/laravel-app/docker-compose.yml
```
* buat file docker-compose.yml di dalam folder laravel-app yang ingin digunakan untuk proses compile dengan semua file yang terhubung


Copy file dibawah:
```txt
version: '3'
services:

  #PHP Service
  app:
    build:
      context: .
      dockerfile: Dockerfile
    image: digitalocean.com/php
    container_name: app
    restart: unless-stopped
    tty: true
    environment:
      SERVICE_NAME: app
      SERVICE_TAGS: dev
    working_dir: /var/www
    volumes:
      - ./:/var/www
      - ./php/local.ini:/usr/local/etc/php/conf.d/local.ini
    networks:
      - app-network

  #Nginx Service
  webserver:
    image: nginx:alpine
    container_name: webserver
    restart: unless-stopped
    tty: true
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./:/var/www
      - ./nginx/conf.d/:/etc/nginx/conf.d/
    networks:
      - app-network

  #MySQL Service
  db:
    image: mysql:5.7.22
    container_name: db
    restart: unless-stopped
    tty: true
    ports:
      - "3306:3306"
    environment:
      MYSQL_DATABASE: laravel
      MYSQL_ROOT_PASSWORD: your_mysql_root_password
      SERVICE_TAGS: dev
      SERVICE_NAME: mysql
    volumes:
      - dbdata:/var/lib/mysql
      - ./mysql/my.cnf:/etc/mysql/my.cnf
    networks:
      - app-network

#Docker Networks
networks:
  app-network:
    driver: bridge
#Volumes
volumes:
  dbdata:
    driver: local
```
### Step 3 Creating the Dockerfile
Dockerfile is a file that is used to create container images.
example :
```html
$ nano ~/laravel-app/Dockerfile
```
Copy file dibawah:
```txt
FROM php:7.2-fpm

# Copy composer.lock and composer.json
COPY composer.lock composer.json /var/www/

# Set working directory
WORKDIR /var/www

# Install dependencies
RUN apt-get update && apt-get install -y \
    build-essential \
    libpng-dev \
    libjpeg62-turbo-dev \
    libfreetype6-dev \
    locales \
    zip \
    jpegoptim optipng pngquant gifsicle \
    vim \
    unzip \
    git \
    curl

# Clear cache
RUN apt-get clean && rm -rf /var/lib/apt/lists/*

# Install extensions
RUN docker-php-ext-install pdo_mysql mbstring zip exif pcntl
RUN docker-php-ext-configure gd --with-gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ --with-png-dir=/usr/include/
RUN docker-php-ext-install gd

# Install composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# Add user for laravel application
RUN groupadd -g 1000 www
RUN useradd -u 1000 -ms /bin/bash -g www www

# Copy existing application directory contents
COPY . /var/www

# Copy existing application directory permissions
COPY --chown=www:www . /var/www

# Change current user to www
USER www

# Expose port 9000 and start php-fpm server
EXPOSE 9000
CMD ["php-fpm"]
```
### Step 4 Configuring PHP, NginX, MySQL
Create the php directory :
```html
$ mkdir ~/laravel-app/php
$ nano ~/laravel-app/php/local.ini
```
Copy file dibawah:
```txt
upload_max_filesize=40M
post_max_size=40M
```
Create the Nginx directory :
```html
$ mkdir -p ~/laravel-app/nginx/conf.d
$ nano ~/laravel-app/nginx/conf.d/app.conf
```
Copy file dibawah:
```txt
server {
    listen 80;
    index index.php index.html;
    error_log  /var/log/nginx/error.log;
    access_log /var/log/nginx/access.log;
    root /var/www/public;
    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass app:9000;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }
    location / {
        try_files $uri $uri/ /index.php?$query_string;
        gzip_static on;
    }
}
```
Create the MySQL directory :
```html
$ mkdir ~/laravel-app/mysql
$ nano ~/laravel-app/mysql/my.cnf
```
Copy file dibawah:
```txt
[mysqld]
general_log = 1
general_log_file = /var/lib/mysql/general.log
```
### Step 8 Modifying Environment Settings and Running the Containers
Sebagai langkah terakhir kita akan membuat salinan file ( .env.example ) yang Laravel sertakan secara default dan beri nama copy ( .env, ) yang merupakan file yang diharapkan oleh Laravel untuk mendefinisikan jaringannya:
>Catatan: Masuk ke dalam directory laravel-app
```html
$ cd ~/laravel-app
$ cp .env.example .env
$ nano .env
```
ikuti dan ubah sesuai dengan contoh dibawah ini :
```txt
DB_CONNECTION=mysql
DB_HOST=db
DB_PORT=3306
DB_DATABASE=laravel
DB_USERNAME=laraveluser
DB_PASSWORD=your_laravel_db_password
```
Menjalankan file docker-compose.yml yang ada di directory laravel-app
```html
$ docker-compose up -d
```

mengecek container yang sedang berjalan:
```html
$ docker ps
```
```txt
CONTAINER ID        IMAGE                  COMMAND                  CREATED             STATUS              PORTS                                      NAMES
416608aadf83        mysql:5.7.22           "docker-entrypoint.s…"   4 hours ago         Up 4 hours          0.0.0.0:3306->3306/tcp                     db
50134ed3453a        digitalocean.com/php   "docker-php-entrypoi…"   4 hours ago         Up 4 hours          9000/tcp                                   app
974d569b29ba        nginx:alpine           "/docker-entrypoint.…"   4 hours ago         Up 4 hours          0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp   webserver
1b05155a7a47        portainer/portainer    "/portainer"             5 hours ago         Up 5 hours          0.0.0.0:860->9000/tcp                      youthful_turing
```
### Step Terakhir untuk mencoba membuka dari hasil yang telah dilakukan di step sebelum-sebelumnya.
buka http://alamat_ip_address:port atau http://localhost:port anda.
lalu jika ingin membuka alamat [Laravel](https://laravel.com/) : http://localhost:80 .
Jika ingin membuka manajemen docker [Portainer](https://www.portainer.io/) : http://localhost:860 .
cek ip :
```html
$ ifconfig
```
Maka akan muncul hasil sebagai berikut :
![Gambar portainer and laravel](https://1.bp.blogspot.com/-rbBCHthjKbg/XuJdU82WHbI/AAAAAAAASrU/Vez9Csf-M1Ev7v2G6xKK7iwXaP_m4EYZwCLcBGAsYHQ/s1600/Screenshot%2Bfrom%2B2020-06-11%2B22-01-08.png)

---
Reference : https://www.digitalocean.com/community/tutorials/how-to-set-up-laravel-nginx-and-mysql-with-docker-compose

--- Terimakasi ---
