
# Blog:  Test Two

## I have lost control of Test One
My certificates expired while *pez* was offline and I don't feel like fiddling with it.  Plus, this blog thing might go in a whole new direction anyway.

## Clean up.
I'm starting by removing all the images, containers and volumes that exist in *pez*.  Some were left over from other attempts to make this work, and I want a clean slate.

* I'm still using docker on pez.
* The blog's files will live in `pez:/home/blog`
* I still have the compose file from the last working version

```
version: '3.3'
services:
  wordpress:
    depends_on:
      - db
    image: wordpress:latest
    volumes:
      - /home/blog/site:/var/www/html
    ports:
      - "8088:80"
    restart: always
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: socks
      WORDPRESS_DB_PASSWORD: CFAS2mPxNaVm2qU2YkHH
db:
  image: mysql:5.7
  volumes:
    - /home/blog/mysql:/var/lib/mysql
  restart: always
  environment:
    MYSQL_ROOT_PASSWORD: 2sgved3qGy3w6Gp7BeyM
    MYSQL_DATABASE: wordpress
    MYSQL_USER: socks
    MYSQL_PASSWORD: CFAS2mPxNaVm2qU2YkHH
```

## Installation Notes

1.  Digital Ocean has some instructions.
[https://www.digitalocean.com/community/tutorials/how-to-install-wordpress-with-docker-compose](https://www.digitalocean.com/community/tutorials/how-to-install-wordpress-with-docker-compose)

2.  Found a letsencrypt + nginx proxy + wordpress and it didn't quite work.  

3.  Portainer shows an "application template" with wordpress in it. `CYdE8x9s3FqLRKKJN9`  First time, that didn't deploy.  I changed the name to `candle_blog` to be non-spacey and it worked.  I can connect to WP with `pez:30000`

Username:  `candle`
pwd:  `qE1ePZW28hP9LoRECV`


> Written with [StackEdit](https://stackedit.io/).
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE5NzQ0MjA4MzJdfQ==
-->