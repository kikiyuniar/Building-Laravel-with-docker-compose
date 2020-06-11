# Building-Laravel-with-docker-compose
build the Laravel Framework with docker-compose and make docker management.

Post in Linkedin : https://www.linkedin.com/in/kikiyuniar/

## Langkah memasang portainer dan docker-compose
* install [portainer](https://www.portainer.io/) di dalam satu container.
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
* buat file docker-compose.yml di dalam folder laravel-app yang ingin digunakan untuk proses compile dengan semua file yang terhubung *
example:
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

