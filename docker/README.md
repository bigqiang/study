# Docker笔记

> 注：boot2docker缺省的用户名是docker,密码是tcuser。可通过 ssh方式登录。比较省事的做法是用命令 `docker-machine ssh default` 直接登录进去，不需要密码

> [一些常见问题](https://github.com/docker/kitematic/wiki/Common-Issues-and-Fixes)

# Docker 参考文件 [官方来源](https://github.com/docker/docker/blob/master/docs/reference/index.md)
 1. [Dockerfile 参考](Dockerfile.md)
 2. Docker run reference
 3. Command line reference
 4. API Reference


## [Dockerfile 文件格式说明](Dockerfile.md)

## [容器中数据的管理](dockervolumes.md)

## PHP开发者搭建Docker开发环境
 1. [如何制作一个定制的 PHP 基础 Docker 镜像](php-docker01.md)
 2. [如何开发一个 PHP 的 Docker 化应用](php-docker02.md)
 3. [如何开发一个 PHP + MySQL 的 Docker 化应用](php-docker03-mysql.md)
 4. [如何配置一个 Docker 化持续集成的 PHP 开发环境](php-docker04-ci.md)
 5. [如何开发一个 PHP + New Relic 的生产级 Docker 化应用](php-docker05-newrelic.md)
 6. [如何开发一个 Laravel + MySQL 框架的 Docker 化应用](php-docker06-laravel.md)

# [我的开发环境](my-development-env.md)

# 配置镜像加速
docker官方镜像速度缓慢，经常难以加载成功，需要换成国内镜像源网站速度会更快些。这里以网易镜像源为例。

## CentOS 6.7
在文件 `/etc/sysconfig/docker` 添加一条命令：
```
other_args="--registry-mirror=http://hub-mirror.c.163.com"
```
然后用 `service docker restart` 重启docker服务

## Mac OS X & Windows
首先进入 `Docker Quickstart Terminal`：
```
docker-machine ssh default
sudo vi /var/lib/boot2docker/profile
```
在 `profile` 文件末尾加上一行 `EXTRA_ARGS='--registry-mirror=http://hub-mirror.c.163.com'`，保存退出。

**注意：** 如果虚机重新启动则 profile 文件会被恢复成默认配置，所以镜像源得重新设置。


