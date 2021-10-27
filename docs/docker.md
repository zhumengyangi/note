# 我用 Docker 部署了 Larvel-admin，并数据持久化到宿主机

## 开始
> 在阿里云 [Linux](https://www.aliyun.com/) 服务器安装 [Docker](https://yq.aliyun.com/articles/110806?spm=5176.8351553.0.0.4f7e1991hQiWF8) 镜像并部署 [Laravel-admin](https://laravel-admin.org/) ，使用 [Docker-Network](https://docs.docker.com/engine/reference/commandline/network/)网络，数据持久化到宿主机

## 版本信息
- Linux 3.10.0-1127.19.1.el7.x86_6
- Docker version 19.03.13
- Laravel Framework 5.5.50
- Laravel-admin 1.5.x-dev

## docker 镜像要求
-   基于[Alpine Linux](https://hub.docker.com/_/alpine/)的容器。
-   Nginx Web服务器。
-   PHP 7.0或更高版本。
-   OpenSSL PHP扩展。
-   PDO PHP扩展。
-   Mbstring PHP扩展。
-   Tokenizer PHP扩展。
-   XML PHP扩展。


### 1. 在合适的目录下创建 `adminDocker`目录

```
mkdir adminDocker
```

### 2. 进入目录`cd adminDocker`  创建 Dockerfile 文件 `vim Dockerfile`

```
FROM nginx:mainline-alpine
LABEL maintainer="zhumengyang <XXX@163.come>" 

COPY start.sh /start.sh
COPY nginx.conf /etc/nginx/nginx.conf
COPY supervisord.conf /etc/supervisord.conf
COPY site.conf /etc/nginx/sites-available/default.conf

RUN echo "#aliyun" > /etc/apk/repositories
RUN echo "https://mirrors.aliyun.com/alpine/v3.10/main/" >> /etc/apk/repositories
RUN echo "https://mirrors.aliyun.com/alpine/v3.10/community/" >> /etc/apk/repositories
RUN apk update

RUN apk add --update curl  php7-fpm php7  php7-dev  php7-apcu  php7-bcmath  \
php7-xmlwriter  php7-ctype  php7-curl  php7-exif  php7-iconv  php7-intl  \
php7-json  php7-mbstring php7-opcache  php7-openssl  php7-pcntl  php7-pdo \
php7-mysqlnd  php7-mysqli  php7-pdo_mysql  php7-pdo_pgsql  php7-phar  \
php7-posix  php7-session  php7-xml  php7-simplexml  php7-mcrypt  php7-xsl  \
php7-zip  php7-zlib  php7-dom  php7-redis php7-tokenizer  php7-gd  \
php7-fileinfo  php7-zmq  php7-memcached  php7-xmlreader 

RUN curl -sS https://getcomposer.org/installer | \
php -- --install-dir=/usr/bin/ --filename=composer

RUN apk add --update bash vim openssh-client supervisor

RUN apk add --no-cache --repository http://dl-3.alpinelinux.org/alpine/edge/testing gnu-libiconv
ENV LD_PRELOAD /usr/lib/preloadable_libiconv.so php

RUN mkdir -p /etc/nginx && \
mkdir -p /etc/nginx/sites-available && \
mkdir -p /etc/nginx/sites-enabled && \
mkdir -p /run/nginx && \
ln -s /etc/nginx/sites-available/default.conf /etc/nginx/sites-enabled/default.conf && \
mkdir -p /var/log/supervisor && \
rm -Rf /var/www/* && \
chmod 755 /start.sh

RUN sed -i -e "s/;cgi.fix_pathinfo=1/cgi.fix_pathinfo=0/g" \
-e "s/variables_order = \"GPCS\"/variables_order = \"EGPCS\"/g" \
/etc/php7/php.ini && \
sed -i -e "s/;daemonize\s*=\s*yes/daemonize = no/g" \
-e "s/;catch_workers_output\s*=\s*yes/catch_workers_output = yes/g" \
-e "s/user = nobody/user = nginx/g" \
-e "s/group = nobody/group = nginx/g" \
-e "s/;listen.mode = 0660/listen.mode = 0666/g" \
-e "s/;listen.owner = nobody/listen.owner = nginx/g" \
-e "s/;listen.group = nobody/listen.group = nginx/g" \
-e "s/listen = 127.0.0.1:9000/listen = \/var\/run\/php-fpm.sock/g" \
-e "s/^;clear_env = no$/clear_env = no/" \
/etc/php7/php-fpm.d/www.conf

EXPOSE 443 80
WORKDIR /var/www

CMD ["/start.sh"]
```

### 3.目录下创建 `start.sh` 文件

```
vim start.sh
```

### 4.文件内容：

```
#!/bin/bash

# ----------------------------------------------------------------------
# Create the .env file if it does not exist.
# ----------------------------------------------------------------------
if [[ ! -f "/var/www/.env" ]] && [[ -f "/var/www/.env.example" ]];
then
	cp /var/www/.env.example /var/www/.env
fi

# ----------------------------------------------------------------------
# Run Composer
# ----------------------------------------------------------------------
if [[ ! -d "/var/www/vendor" ]];
then
	cd /var/www
	composer update
	composer dump-autoload -o
fi

# ----------------------------------------------------------------------
# Start supervisord
# ----------------------------------------------------------------------
exec /usr/bin/supervisord -n -c /etc/supervisord.conf

```

### 5. `adminDocker` 目录下创建 `nginx.conf` 文件

```
vim nginx.conf
```

### 6.`nginx.conf`文件内容

```
user nginx;
worker_processes 1;

error_log /var/log/nginx/error.log warn;
pid /var/run/nginx.pid;

events {
	worker_connections 1024;
}

http {
	include /etc/nginx/mime.types;
	default_type application/octet-stream;
	access_log off;
	sendfile on;
	#tcp_nopush on;
	keepalive_timeout 65;
	#gzip on;
	client_max_body_size 50m;
	include /etc/nginx/sites-enabled/*.conf;
}
```

### 7. 同样创建 `site.conf` 文件

```
vim site.conf
```

### 8. `site.conf` 文件内容

```
server {
	listen 80;

	root /var/www/public;
	index index.php index.html;

	location / {
		try_files $uri $uri/ /index.php?$query_string;
	}

	location ~ /\. {
		deny all;
	}

	location ~ \.php$ {
		try_files $uri = 404;
		fastcgi_split_path_info ^(.+\.php)(/.+)$;
		fastcgi_pass unix:/var/run/php-fpm.sock;
		fastcgi_index index.php;
		fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
		include fastcgi_params;
	}
}
```

### 9.创建 `supervisord.conf` 文件

```
vim supervisord.conf
```

### 10. `supervisord.conf` 文件内容

```
[unix_http_server]
file=/dev/shm/supervisor.sock

[supervisord]
logfile=/tmp/supervisord.log
logfile_maxbytes=50MB
logfile_backups=10
loglevel=warn
pidfile=/tmp/supervisord.pid
nodaemon=false
minfds=1024
minprocs=200
user=root

[rpcinterface:supervisor]
supervisor.rpcinterface_factory = supervisor.rpcinterface:make_main_rpcinterface

[supervisorctl]
serverurl=unix:///dev/shm/supervisor.sock

[program:php-fpm7]
command = /usr/sbin/php-fpm7 --nodaemonize --fpm-config /etc/php7/php-fpm.d/www.conf
autostart=true
autorestart=true
priority=5
stdout_logfile=/dev/stdout
stdout_logfile_maxbytes=0
stderr_logfile=/dev/stderr
stderr_logfile_maxbytes=0

[program:nginx]
command=/usr/sbin/nginx -g "daemon off;"
autostart=true
autorestart=true
priority=10
stdout_logfile=/dev/stdout
stdout_logfile_maxbytes=0
stderr_logfile=/dev/stderr
stderr_logfile_maxbytes=0
```

### 11.  `adminDocker` 目录下创建 `src` 文件夹，把你的 `Laravel-admin` 项目代码git clone下来，或者 `cpoy` 进去。 `adminDocker` 目录下面执行 `ls` 应该是下面几个目录文件

```
[root@bogon www]# ls
Dockerfile  site.conf  src  start.sh  supervisord.conf
[root@bogon www]# cd src/
[root@bogon src]# ll
总用量 4
drwxrwxrwx. 12 root root 4096 12月 10 15:18 adminData
# 这里我 clone 下来的是个 adminData 项目
```

### 12. 在 `/var/www/adminDocker` 路径下构建 `docker` 镜像

```
// 构建名称为 myImage 镜像文件
docker build -t  myImage .
```

### 13. 拉取 `Mysql` 镜像

```
// 这里我用的是 Mysql 5.6
docker pull mysql:5.6
```

### 14. 创建数据卷

```
mkdir /var/www/adminDocker/src/adminMysqlData
```

### 15. 在 `Docker` 中创建网络（只需要执行第一个命令）


```
// 创建名字为 adminNet 的网络
docker network create adminNet

// 查看所有的 docker network
docker network ls

// 查看名字为 adminNet 中的 Docker Network
docker network inspect adminNet

// 删除名字为 adminNet 的 Docker Network 
docker network rm adminNet
```


### 16. 查看构建的 `Dokcer` 镜像

```
// 会有 myImage 和 mysql5.6 的镜像文件
docker images
```

### 17.容器运行, 终端上运行以下命令：

```

// Mysql 容器
// -d 后台运行容器
// --restart always ，Docker重启后自动重启该容器
// -p 指定端口映射，格式为：主机(宿主)端口:容器端口。这里是 3307 映射到 3306
// -v 绑定数据卷（刚刚创建的 adminMysqlData 和 adminData 文件夹下）
// --name 给容器重命名
// --network 链接哪一个 Docker Network
// --network-alias 给当前镜像文件在 Network 中取别名
// -e 这里楼主不是很清楚就不写了哈
// 最后写上刚刚拉下来的 mysql:5.6 即可

docker run -d --restart always -p 3307:3306 -v /var/www/adminDocker/src/adminMysqlData:/var/lib/mysql --name adminMysql --network adminNet --network-alias adminMysql -e MYSQL_ROOT_PASSWORD=root mysql:5.6


// Laravel-admin 容器
// -p 指定端口映射，格式为：主机(宿主)端口:容器端口。这里是 8000 映射到 80
// -v 绑定数据卷（刚刚创建的 adminMysqlData 和 adminData 文件夹下）
// --privileged=true 以 root 权限去运行，否则在该 Docker 容器中没有权限使用命令
// -e 这里楼主不是很清楚就不写了哈
// 最后写上刚刚拉下来的 myImage 即可

docker run -d --restart always -p 8000:80 -v /var/www/adminDocker/src/adminData:/var/www --privileged=true --name adminWeb --network adminNet --network-alias adminWeb -e MYSQL_ROOT_PASSWORD=root myImage


```

### 18. 在 `Mysql` 容器中创建数据库并导入文件

```
// 查看 Mysql 的 CONTAINER ID
docker ps -a
root@bogon src]# docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                   PORTS               NAMES
22e0082dbf46        myImage      "/docker-entrypoin..."   1 days ago          Exited (0) 4 days ago                        adminWeb
a1ca8c42ac4f        mysql:5.6           "docker-entrypoint..."   4 days ago          Exited (0) 4 days ago                     adminMysql

// 将 data.sql 导入到 Mysql 中的根路径下 （原数据库文件）
// a1ca8c42ac4f 为 Mysql 的 CONTAINER ID
docker cp /var/www/adminDocker/src/adminData/data.sql a1ca8c42ac4f:/

// 进入 Mysql 容器
docker exec -it a1ca8c42ac4f  bash

// 查看刚刚复制过来的文件
root@a1ca8c42ac4f:/# ls
bin   docker-entrypoint-initdb.d  home	 media	proc  sbin	sys  var
boot  entrypoint.sh		  lib	 mnt	root  srv	tmp
dev   etc			  lib64  opt	run   data.sql	usr

// 连接数据库
// --default-character-set=utf8 设置 utf8 字符集，不然导入的中文数据会乱码
// -u 用户名
// -p 回车，会让你输入密码，设置的是 root
mysql --default-character-set=utf8 -uroot -p

// 然后创建 adminDb 数据库
create database adminDb;

// 选择该数据库
use adminDb;

// 导入文件
source /data.sql;

// 退出 Mysql 连接
exit;

// 退出 Mysql 容器
exit;

```

### 19. 在 Laravel-admin 容器中初始化项目

```
// 在 18 步中获取到的容器 CONTAINER ID
docker exec -it 22e0082dbf46 bash

// 进来之后是在 /var/www 路径下，可以看到你 clone 下来的项目文件
// 建议把本地的 .env文件内容 复制过来一份

// 复制 .env 文件
cp .env.example .env

// 编辑 .env 文件
vim .env

// 加上自定义前缀： ADMIN_ROUTE_PREFIX=/laravel

// 这里 DB_HOST 要写刚刚创建 Mysql 容器时候的 --name 值，端口还是写3306
// 可能有些同学会迷糊，为什么刚刚端口映射的是 3307 现在还要写 3306 。
// 因为那个 3307 是对外端口，这里连接的 Mysql 是对内的（自己理解哈）

DB_CONNECTION=mysql
DB_HOST=adminMysql
DB_PORT=3306
DB_DATABASE=adminDb
DB_USERNAME=root
DB_PASSWORD=root

// 初始化项目要添加 key 
php artisan key:gen

// 顺便修改一下存放日志的文件夹可能没有权限（权限已实际情况分配）
chmod -R 755 storage/*

// 测试一下是否连接上了数据库，如果连接不上看一下当时 run Mysql 时候 --name 值 
php artisan migrate

```

### 20. 访问你的IP + 端口号8000 + ADMIN_ROUTE_PREFIX自己写的配置

```
192.168.0.217:8000/admin
```

参考文档： [尝试使用 docker 部署 Laravel 项目](https://learnku.com/articles/33074#replies)

本文链接：https://learnku.com/articles/52585

> 刚开始记录，觉得不错可以点个赞，有问题可以评论下来哈:grin:






