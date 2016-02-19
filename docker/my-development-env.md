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
  1. `docker pull ubuntu:trusty`