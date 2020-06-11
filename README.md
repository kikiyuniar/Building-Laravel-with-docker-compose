# Building-Laravel-with-docker-compose
build the Laravel Framework with docker-compose and make docker management.

Post in Linkedin : https://www.linkedin.com/in/kikiyuniar/

## Langkah pertama yang harus dilakukan adalah memasang portainer dan docker-compose
* install [portainer](https://www.portainer.io/) di dalam satu container.
example:

```html  
$ docker run -d -p 860:9000 -v /var/run/docker.sock:/var/run/docker.sock portainer/portainer
```
>catatan: 860 merupakan port yang akan di akses untuk membuka halaman portainer yang telah terpasang

### Downloading Laravel and Installing Dependencies
