# 开发环境搭建

## 环境描述
  各部分要求容器相互独立，各自维护。如果合并成一个镜像，维护有些过于频繁（可以考虑如果合并成一个镜像时，如何解决镜像更新问题）
  1. Ubuntu/CentOS6
  2. MySQL
  3. Nginx
  4. PHP
  5. Memcache

## 搭建
  1. CentOS6 `docker pull centos:6`

  2. MySQL `docker pull mysql:5.6`

  3. Nginx `docker pull nginx:1.9.11`
  [描述](https://hub.docker.com/_/nginx/)

  4. PHP
  [描述](https://hub.docker.com/_/php/)

  5. Memcache

## 实际操作
### 1. `docker pull mysql:5.6`

#### 启动一个 MySQL 实例:

```
$ docker run --name some-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag
```

`some-mysql` 是你指派给容器的名称，`my-secret-pw`是MySQL的root用户的密码，`tag`是你想指定MySQL的版本号。

#### 其他容器的应用链接 MySQL
该镜像公开的是 MySQL的标准端口号 (3306)，因此 MySQL 实例可用于其他容器的应用。为链接MySQL容器，以如下方式启动你的应用程序的容器：
```
$ docker run --name some-app --link some-mysql:mysql -d application-that-uses-mysql
```

#### MySQL命令行客户端链接到 MySQL
用以下命令启动另一个mysql容器实例，然后针对原来的mysql容器运行命令行客户端，可以对数据库实例执行SQL语句：
```
$ docker run -it --link some-mysql:mysql --rm mysql sh -c 'exec mysql -h"$MYSQL_PORT_3306_TCP_ADDR" -P"$MYSQL_PORT_3306_TCP_PORT" -uroot -p"$MYSQL_ENV_MYSQL_ROOT_PASSWORD"'
```
`some-mysql`是原来mysql容器的名称。

MySQL 命令行客户端详细内容请查看[MySQL documentation](http://dev.mysql.com/doc/en/mysql.html)

#### 容器 shell 访问和查看 MySQL 日志
`docker exec`命令可以运行 Docker 容器内的命令。下面命令行显示的是 mysql 容器内的 bash shell：
```
$ docker exec -it some-mysql bash
```
通过 Docker 容器查看 MySQL Server 日志：
```
$ docker logs some-mysql
```

#### 自定义 MySQL 设置文件
MySQL 的启动设置内容在文件 `/etc/mysql/my.cnf`中指定，并且，该文件中包含的任何文件都存在于 `/etc/mysql/conf.d` 目录，都以 `.cnf` 结尾。对该目录这些文件的设置会扩展和(或)覆盖 `/etc/mysql/my.cnf` 中的设置。要使用自定义的 MySQL 设置，就要在主机的一个目录中创建自己的配置文件，然后把这个目录位置挂载到 mysql 容器内的 `/etc/mysql/conf.d` 中。

如果 `/my/custom/config-file.cnf` 是你自定义配置文件的路径名称，那就可以以下方式启动 mysql 容器(注意，自定义配置文件的目录路径仅在本命令中使用)：
```
$ docker run --name some-mysql -v /my/custom:/etc/mysql/conf.d -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag
```
这会启动一个新容器`some-mysql`，这个MySQL实例会把 /etc/mysql/my.cnf 与 /etc/mysql/conf.d/config-file.cnf 的设置合并到一起，并且后者设置优先生效。

注意，启用SELinux的主机系统可能会让挂载设置存在问题。当前解决办法是分派相应的 SELinux 策略类型给新配置文件，以便允许容器挂载它：
```
$ chcon -Rt svirt_sandbox_file_t /my/custom
```

#### 环境变量
启动这个 `mysql` 镜像时，需要`docker run`命令行传递一或多个环境变量给MySQL实例来调整它的配置。请注意，如果启动的容器已经带有一个包含数据库的目录，那么以下描述的所有环境变量都不会生效：任何预先存在的数据库都会被在容器启动时无损保留。

**MYSQL_ROOT_PASSWORD**
该变量是强制性的，必选变量，可以指定 MySQL 的 root 超级用户账号密码。前面例子中，它被设置成`my-secret-pw`。

**MYSQL_DATABASE**
该变量可选，允许在镜像启动时指定数据库的名称。如果用名及密码已指定(见下文)，那么会许可该用户有访问该数据库的超级用户访问权([对应 `GRANT ALL`](http://dev.mysql.com/doc/en/adding-users.html))。

**MYSQL_USER, MYSQL_PASSWORD**
可选变量，用于创建新用户和设置用户密码。该用户会被授予`MYSQL_DATABASE`变量指定的数据库的超级用户访问权限（见上文）。用户创建时，两个变量都要使用。

注意，不要用此机制去创建 root 账号，该账号在 `MYSQL_ROOT_PASSWORD` 变量指定密码时已经默认创建。

**MYSQL_ALLOW_EMPTY_PASSWORD**
可选变量。设置`yes`，允许容器让root账号以空密码启动。注意，不建议设置值为`yes`，除非你明白该操作的意义，这会让 MySQL 实例全无防护，任何人都可以得到超级用户的所有访问权。

#### 新实例的初始化
容器初次启动时，新数据库会根据配置变量进行初始化。还会执行位于`/docker-entrypoint-initdb.d`当中扩展名为`.sh`、`.sql`的几个文件。 通过 [挂载一个 SQL dump 到该目录](dockervolumes.md#主机文件挂载成数据卷) 很容易填充 MySQL 服务，使用提供数据的自定义镜像也很容易。

#### 注意事项
##### 数据存储位置
重要说明：Docker中运行的应用有多种方式存储使用的数据。我们鼓励`mysql`镜像用户熟悉这些方法的可用选选择，包括：

- 让 Docker 管理你的数据库数据的存储。可以这样实现，[让主机系统的磁盘使用自身内部的卷管理系统，把数据库的文件写到这些硬盘上](dockervolumes.md#数据卷的添加)。 这是默认方式，也简单，对用户也相当透明。缺点在于，对于直接运行于主机系统（即容器外部）中的工具和应用难于定位 这些文件。
- 在主机系统（容器外部）创建数据目录，并且，[挂载它到一个对容器内部可见的目录上](dockervolumes.md#挂载一个主机目录作数据卷)。数据库文件存放于主机系统中已知位置，主机系统中的工具和应用易于访问。确点在于，用户必须确保目录存在， 而且主机系统上还要正确设置该目录的许可权限及其他安全机制。

理解不同存储方法的选择和变化从 Docker 文档开始是个好的开端，有多篇博客及论坛帖子针对这个问题进行了讨论并给出了建议。这里我们只对上面的后一种选择显示基本的操作：

1. 在主机系统的适当卷上创建一个数据目录，如 `/my/own/datadir`。
2. 以如下方式启动 `mysql` 容器：
```
$ docker run --name some-mysql -v /my/own/datadir:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag
```
命令的 `-v /my/own/datadir:/var/lib/mysql` 部分表示，挂载主机系统的 `/my/own/datadir` 目录，作为容器内部的 `/var/lib/mysql`，这是 MySQL 默认会写数据文件的位置。

注意，启用 SELinux 的主机系统可能会出问题。当前解决办法是分派相应的 SELinux 策略类型给新配置文件，以便允许容器挂载它：
```
$ chcon -Rt svirt_sandbox_file_t /my/own/datadir
```

##### MySQL 初始化完成才能连接
在容器启动，数据库还没有初始化时，会创建一个默认数据库。 这是预料之中的情况，也就是说，初始化结束后才会接收连接。对于使用了诸如 docker-compose 这样自动化工具，同时启动多个容器时，就可能产生问题。

##### 针对已有数据库的用法
如果启动的 `mysql` 容器实例有一个已经包含数据库的数据目录(特别是一个 `mysql` 的子目录)，则在 run 命令行中的`$MYSQL_ROOT_PASSWORD`变量应该略写；任何情况下它都会被忽视，任何情况下预先存在的数据库都不会被修改。

### 2. `docker pull million12/nginx-php:php56`



## 镜像 million12/nginx-php 说明(Nginx + PHP-FPM docker container)
 [million12/nginx-php](https://registry.hub.docker.com/u/million12/nginx-php/) docker container 是 Nginx + PHP-FPM 组合版。

不同PHP版本可查看不同版本库分支。可在 Docker Hub 的不同 tag 中找到他们：
* `million12/nginx-php:latest` - PHP 7.0 # built from `master` branch [![Circle CI](https://circleci.com/gh/million12/docker-nginx-php.svg?style=svg)](https://circleci.com/gh/million12/docker-nginx-php)
* `million12/nginx-php:php70` - PHP 7.0 # built from `php70` branch [![Circle CI](https://circleci.com/gh/million12/docker-nginx-php/tree/php70.svg?style=svg)](https://circleci.com/gh/million12/docker-nginx-php/tree/php70)
* `million12/nginx-php:php56` - PHP 5.6 # built from `php56` branch [![Circle CI](https://circleci.com/gh/million12/docker-nginx-php/tree/php56.svg?style=svg)](https://circleci.com/gh/million12/docker-nginx-php/tree/php56)
* `million12/nginx-php:php55` - PHP 5.5 # built from `php55` branch [![Circle CI](https://circleci.com/gh/million12/docker-nginx-php/tree/php55.svg?style=svg)](https://circleci.com/gh/million12/docker-nginx-php/tree/php55)


### 包括内容：

#### Nginx

该镜像基于 [million12/nginx](https://github.com/million12/docker-nginx)。
**默认vhost** 配置于 `/data/www/default`。.php文件放在该位置可被执行。

#### - PHP-FPM

vhost 默认运行的是 **PHP 5.6**。访问.php文件的请求会被重定向给PHP upstream。 参阅 [/etc/nginx/conf.d/php-location.conf](container-files/etc/nginx/conf.d/php-location.conf).

为避免每个vhost重复相同配置项，文件 [/etc/nginx/fastcgi_params](container-files/etc/nginx/fastcgi_params) 已经做配置优化。该配置在多数PHP应用中工作良好 (如 Symfony2, TYPO3, Wordpress, Drupal)。

自定义 PHP.ini 的指令存放于 [/etc/php.d](container-files/etc/php.d/) 内。

#### 目录结构
```
/data/www # 放web内容之处
/data/www/default # 默认vhost的根目录
/data/logs/ # Nginx、PHP日志
/data/tmp/php/ # PHP临时文件目录
```

#### 错误日志

PHP 错误信息会转发给 stderr (设置 INI error_log 项值为空)，supervisor 会捕捉到该错误信息。可轻松使用 `docker logs [container]` 查看这些信息。另外，这也会被Nginx的worker进程捕捉到，记录到 `/data/logs/nginx-error.log` 文件中，PHP-FPM 会记录到 `/data/logs/php-fpm*.log` 文件中。

##### - 预定义 PHP 后端的 FastCGI cache

本设置在 location {} 上下文环境中才有用。在 vhost 中添加的设置类似下面：
```
location ~ \.php$ {
    # Your standard directives...
    include               fastcgi_params;
    fastcgi_pass          php-upstream;

    # Use the configured cache (adjust fastcgi_cache_valid to your needs):
    fastcgi_cache         APPCACHE;
    fastcgi_cache_valid   60m;
}
```

#### Web app 开发的常用开发工具

* Ruby 2.0, Bundler
* NodeJS and NPM
* NPM packages like gulp, grunt, bower, browser-sync

### 使用

```
docker run -d -v /data --name=web-data busybox
docker run -d --volumes-from=web-data -p=80:80 --name=web million12/nginx-php
```

操作完成后，浏览器中打开 http://CONTAINER_IP:PORT/ 可看到默认vhost内容(类似这样: '*default vhost created on [timestamp]*') 。

也可以把 `/data/www/default/index.html` 换成 `index.php`，比如用 phpinfo() 检查已安装PHP设置情况。可以这样实现，使用一个独立容器，把它挂载到 /data 卷 (`docker run -ti --volumes-from=web-data --rm busybox`)，然后把文件添加到这个卷里。


### 自定义

定制这个容器有几种办法，即可在运行时也可以在构建新镜像时进行：

* Nginx的定制及添加新vhost等的信息可参考 [million12:nginx](https://github.com/million12/docker-nginx) 。
* 必要的话，可覆盖 `/etc/nginx/fastcgi_params`。
* 把自己定制 PHP 的 `*.ini` 文件添加到 `/etc/php.d/`中。
* 把息的 PHP-FPM 的 .conf 文件添加到 `/data/conf/php-fpm-www-*.conf`中，以修改 PHP-FPM 的 www pool。

### ENV 变量

  1. **NGINX_GENERATE_DEFAULT_VHOST**
默认值： `NGINX_GENERATE_DEFAULT_VHOST=false`
示例： `NGINX_GENERATE_DEFAULT_VHOST=true`
当值为 `true` 时，会给 Nginx vhost 虚设一个默认的万能配置文件保存在 `/etc/nginx/hosts.d/default.conf`。此外，还会创建一个默认的 index.php 文件显示 `phpinfo()` 执行结果。**忠告**：因为这公开了详细的PHP配置，所以会存在安全泄露的隐患，在生产环境下要记得移除它。

  2. **STATUS_PAGE_ALLOWED_IP**
默认值：`STATUS_PAGE_ALLOWED_IP=127.0.0.1`
示例：`STATUS_PAGE_ALLOWED_IP=10.1.1.0/16`
配置IP地址，可通过URL页面 `/fpm_status` 查看PHP-FPM状态。





## 命令示例

```
$ docker run --rm -ti ubuntu /bin/bash
```
上一条命令的简要说明：
* --rm：告诉Docker一旦运行的进程退出就删除容器。这在进行测试时非常有用，可免除杂乱
* -ti：告诉Docker分配一个伪终端并进入交互模式。这将进入到容器内，对于快速原型开发或尝试很有用，但不要在生产容器中打开这些标志
* ubuntu：这是容器立足的镜像
* /bin/bash：要运行的命令，因为我们以交互模式启动，它将显示一个容器的提示符

在运行run命令时，你可指定链接、卷、端口、窗口名称（如果你没提供，Docker将分配一个默认名称）等等。


在后台运行一个容器

    $ docker run -d ubuntu ping 8.8.8.8
    31c68e9c09a0d632caae40debe13da3d6e612364198e2ef21f842762df4f987f
    $

输出是分配的ID，因是随机的，所见未必与图示相同。检查一下容器是否运行

    $ docker ps
    CONTAINER ID IMAGE         COMMAND         CREATED        STATUS        PORTS  NAMES
    31c68e9c09a0 ubuntu:latest "ping 8.8.8.8"  2 minutes ago  Up 2 minutes         loving_mcclintock


`docker help` 所有Docker命令可以用它查看

名为`sample_job`的容器，可以使用以下命令来停止：`docker stop $sample_job`

使用以下命令可以重新启动该容器：`docker restart $sample_job`

如果要完全移除容器，需要先将该容器停止，然后才能移除。像这样：
```
docker stop $sample_job
docker rm $sample_job
```

将容器的状态保存为镜像，使用以下命令：`docker commit $sample_job job1``

注意，镜像名称只能取字符[a-z]和数字[0-9]。

现在，你就可以使用以下命令查看所有镜像的列表：`docker images`

在registry中的镜像可以使用以下命令查找到：`docker search (image-name)`

查看镜像的历史版本可以执行以下命令：`docker history (image_name)`

最后，使用以下命令将镜像推送到registry：`docker push (image_name)`

非常重要的一点是，你必须要知道存储库不是根存储库，它应该使用此格式(user)/(repo_name)。

Docker提供了一个非常强大的命令diff，它可以列出容器内发生变化的文件和目录。这些变化包括添加（A-add）、删除（D-delete）、修改（C-change）。该命令便于Debug，并支持快速的共享环境。
语法是：`docker diff container`

`docker events`： 打印指定时间内的容器的实时系统事件。

`docker import` ：Docker可以导入远程文件、本地文件和目录。使用HTTP的URL从远程位置导入，而本地文件或目录的导入需要使用-参数。从远程位置导入的语法是：`docker import http://example.com/example.tar`

`docker export`: 类似于`import`，`export`命令用于将容器的系统文件打包成tar文件。

cp： 这个命令是从容器内复制文件到指定的路径上。语法如下：`docker cp container:path hostpath`

login： 此命令用来登录到Docker registry服务器，语法如下：`docker login [options] [server]`
如要登录自己主机的registry请使用：
`docker login localhost:8080`


inspect： Docker inspect命令可以收集有关容器和镜像的底层信息。这些信息包括：
* 容器实例的IP地址
* 端口绑定列表
* 特定端口映射的搜索
* 收集配置的详细信息

该命令的语法是：`docker inspect container/image`
```
docker inspectroot@tankywoo-docker:~# docker inspect 49bfc7a9817f
 ...
 "Env": [
     "name=tanky",
     "HOME=/",
     "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
 ],
 ...
```

可以通过在`docker run`设置或修改环境变量:`docker run -i -t --env name="tanky" ubuntu:newtest /bin/bash`

kill：发送SIGKILL信号来停止容器的主进程。语法是：`docker kill [options] container`

rmi：
该命令可以移除一个或者多个镜像，语法如下：
`docker rmi image`
镜像可以有多个标签链接到它。在删除镜像时，你应该确保删除所有相关的标签以避免错误。

wait：
阻塞对指定容器的其它调用方法，直到容器停止后退出阻塞。

load：
该命令从tar文件中载入镜像或仓库到STDIN。

save：
类似于load，该命令保存镜像为tar文件并发送到STDOUT。语法如下：
`docker save image`