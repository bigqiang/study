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

## 实际
  1. `docker pull million12/nginx-php:php56`


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

