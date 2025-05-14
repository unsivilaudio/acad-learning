# Running a Laravel dockerized project

> _From the [mega-thread](https://www.udemy.com/course/docker-kubernetes-the-practical-guide/learn/lecture/34927676#questions/18527114) on Udemy_

## 📁 /

> docker-compose.yml

```
services:
    server:
        # image: 'nginx:stable-alpine'
        build:
            context: .
            dockerfile: dockerfiles/nginx.dockerfile
        ports:
            - '8000:80'
        volumes:
            - ./src:/var/www/html
            - ./nginx/nginx.conf:/etc/nginx/conf.d/default.conf:ro
        depends_on:
            - php
            - mysql
    php:
        build:
            context: .
            dockerfile: dockerfiles/php.dockerfile
        volumes:
            - ./src:/var/www/html:delegated
    mysql:
        image: mysql:5.7
        env_file:
            - ./env/mysql.env
    composer:
        build:
            context: .
            dockerfile: dockerfiles/composer.dockerfile
        volumes:
            - ./src:/var/www/html
    artisan:
        build:
            context: .
            dockerfile: dockerfiles/php.dockerfile
        volumes:
            - ./src:/var/www/html
        entrypoint: ['php', '/var/www/html/artisan']
    npm:
        image: node:14
        working_dir: /var/www/html
        entrypoint: ['npm']
        volumes:
            - ./src:/var/www/html
```

## 📁 **env/**

> env/mysql.env
>
> -   environment variables for our mysql service configuration

```
MYSQL_DATABASE=homestead
MYSQL_USER=homestead
MYSQL_PASSWORD=secret
MYSQL_ROOT_PASSWORD=secret
```

## 📁 **nginx/**

> nginx.conf
>
> -   configuration for our nginx server to forward incoming requests to the FPM server

```
server {
    listen 80;
    index index.php index.html;
    server_name localhost;
    root /var/www/html/public;
    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }
    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass php:9000;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }
}
```

## 📁 **dockerfiles/**

> composer.dockerfile

```
FROM composer:latest

RUN addgroup -g 1000 laravel && adduser -G laravel -g laravel -s /bin/sh -D laravel

USER laravel

WORKDIR /var/www/html

ENTRYPOINT [ "composer", "--ignore-platform-reqs" ]
```

> nginx.dockerfile

```
FROM nginx:stable-alpine

WORKDIR /etc/nginx/conf.d

COPY nginx/nginx.conf .

RUN mv nginx.conf default.conf

WORKDIR /var/www/html

COPY src .
```

> php.dockerfile

```
FROM php:8.4-fpm-alpine

WORKDIR /var/www/html

COPY src .

RUN docker-php-ext-install pdo pdo_mysql

# IF YOU GET PERMISSIONS ISSUES ON /var/www/html/storage
# RUN chown -R www-data:www-data .

RUN addgroup -g 1000 laravel && adduser -G laravel -g laravel -s /bin/sh -D laravel

USER laravel
```

---

## Build your project (delete any existing "src" directory)

`docker compose run composer create-project --prefer-dist laravel/laravel .`

## Start your project

`docker compose up -d server php mysql`

## Edit your environment variables

> we need to edit the Laravel environment variables with those defined in our `env/mysql.env` file, as well as point it to our docker mysql service.

`src/.env`

    # New field -- represents the database type
    DB_CONNECTION=mysql
    DB_HOST=mysql
    DB_PORT=3306
    DB_DATABASE=homestead
    DB_USERNAME=homestead
    DB_PASSWORD=secret

_Save the file._

## ⚠️ IMPORTANT

> the default configuration now uses flatstorage (sqlite), and now we need to migrate the data to our SQL database

-   migrate the database

    > `docker compose run artisan migrate`

-   update the view's cache

    > `docker compose run artisan view:cache`

-   generate an encryption key
    > `docker compose run artisan key:generate`

# 🎉 You should now be able to view your project at http://localhost:8000 🎉
