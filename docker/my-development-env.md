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
The following command starts another mysql container instance and runs the mysql command line client against your original mysql container, allowing you to execute SQL statements against your database instance:

$ docker run -it --link some-mysql:mysql --rm mysql sh -c 'exec mysql -h"$MYSQL_PORT_3306_TCP_ADDR" -P"$MYSQL_PORT_3306_TCP_PORT" -uroot -p"$MYSQL_ENV_MYSQL_ROOT_PASSWORD"'
... where some-mysql is the name of your original mysql container.

More information about the MySQL command line client can be found in the MySQL documentation

#### Container shell access and viewing MySQL logs
The docker exec command allows you to run commands inside a Docker container. The following command line will give you a bash shell inside your mysql container:

$ docker exec -it some-mysql bash
The MySQL Server log is available through Docker's container log:

$ docker logs some-mysql

#### Using a custom MySQL configuration file
The MySQL startup configuration is specified in the file /etc/mysql/my.cnf, and that file in turn includes any files found in the /etc/mysql/conf.d directory that end with .cnf. Settings in files in this directory will augment and/or override settings in /etc/mysql/my.cnf. If you want to use a customized MySQL configuration, you can create your alternative configuration file in a directory on the host machine and then mount that directory location as /etc/mysql/conf.d inside the mysql container.

If /my/custom/config-file.cnf is the path and name of your custom configuration file, you can start your mysql container like this (note that only the directory path of the custom config file is used in this command):

$ docker run --name some-mysql -v /my/custom:/etc/mysql/conf.d -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag
This will start a new container some-mysql where the MySQL instance uses the combined startup settings from /etc/mysql/my.cnf and /etc/mysql/conf.d/config-file.cnf, with settings from the latter taking precedence.

Note that users on host systems with SELinux enabled may see issues with this. The current workaround is to assign the relevant SELinux policy type to your new config file so that the container will be allowed to mount it:

$ chcon -Rt svirt_sandbox_file_t /my/custom

#### 环境变量
When you start the mysql image, you can adjust the configuration of the MySQL instance by passing one or more environment variables on the docker run command line. Do note that none of the variables below will have any effect if you start the container with a data directory that already contains a database: any pre-existing database will always be left untouched on container startup.

**MYSQL_ROOT_PASSWORD**
This variable is mandatory and specifies the password that will be set for the MySQL root superuser account. In the above example, it was set to my-secret-pw.

**MYSQL_DATABASE**
This variable is optional and allows you to specify the name of a database to be created on image startup. If a user/password was supplied (see below) then that user will be granted superuser access (corresponding to GRANT ALL) to this database.

**MYSQL_USER, MYSQL_PASSWORD**
These variables are optional, used in conjunction to create a new user and to set that user's password. This user will be granted superuser permissions (see above) for the database specified by the MYSQL_DATABASE variable. Both variables are required for a user to be created.

Do note that there is no need to use this mechanism to create the root superuser, that user gets created by default with the password specified by the MYSQL_ROOT_PASSWORD variable.

**MYSQL_ALLOW_EMPTY_PASSWORD**
This is an optional variable. Set to yes to allow the container to be started with a blank password for the root user. NOTE: Setting this variable to yes is not recommended unless you really know what you are doing, since this will leave your MySQL instance completely unprotected, allowing anyone to gain complete superuser access.

#### Initializing a fresh instance
When a container is started for the first time, a new database mysql will be initialized with the provided configuration variables. Furthermore, it will execute files with extensions .sh and .sql that are found in /docker-entrypoint-initdb.d. You can easily populate your mysql services by mounting a SQL dump into that directory and provide custom images with contributed data.

#### 注意事项
##### Where to Store Data
Important note: There are several ways to store data used by applications that run in Docker containers. We encourage users of the mysql images to familiarize themselves with the options available, including:

Let Docker manage the storage of your database data by writing the database files to disk on the host system using its own internal volume management. This is the default and is easy and fairly transparent to the user. The downside is that the files may be hard to locate for tools and applications that run directly on the host system, i.e. outside containers.
Create a data directory on the host system (outside the container) and mount this to a directory visible from inside the container. This places the database files in a known location on the host system, and makes it easy for tools and applications on the host system to access the files. The downside is that the user needs to make sure that the directory exists, and that e.g. directory permissions and other security mechanisms on the host system are set up correctly.
The Docker documentation is a good starting point for understanding the different storage options and variations, and there are multiple blogs and forum postings that discuss and give advice in this area. We will simply show the basic procedure here for the latter option above:

Create a data directory on a suitable volume on your host system, e.g. /my/own/datadir.
Start your mysql container like this:

$ docker run --name some-mysql -v /my/own/datadir:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag
The -v /my/own/datadir:/var/lib/mysql part of the command mounts the /my/own/datadir directory from the underlying host system as /var/lib/mysql inside the container, where MySQL by default will write its data files.

Note that users on host systems with SELinux enabled may see issues with this. The current workaround is to assign the relevant SELinux policy type to the new data directory so that the container will be allowed to access it:

$ chcon -Rt svirt_sandbox_file_t /my/own/datadir

##### No connections until MySQL init completes
If there is no database initialized when the container starts, then a default database will be created. While this is the expected behavior, this means that it will not accept incoming connections until such initialization completes. This may cause issues when using automation tools, such as docker-compose, which start several containers simultaneously.

##### Usage against an existing database
If you start your mysql container instance with a data directory that already contains a database (specifically, a mysql subdirectory), the $MYSQL_ROOT_PASSWORD variable should be omitted from the run command line; it will in any case be ignored, and the pre-existing database will not be changed in any way.

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