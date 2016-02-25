<!--[metadata]>
+++
aliases = ["/engine/userguide/dockervolumes/"]
title = "Manage data in containers"
description = "How to manage data inside your Docker containers."
keywords = ["Examples, Usage, volume, docker, documentation, user guide, data,  volumes"]
[menu.main]
parent = "engine_learn"
+++
<![end-metadata]-->
# 容器中数据的管理

[Docker的基础概念](https://github.com/docker/docker/blob/master/docs/userguide/containers/usingdocker.md)已有介绍，也了解了如何操作 [Docker 镜像](https://github.com/docker/docker/blob/master/docs/userguide/containers/dockerimages.md) 以及 [容器间的网络与连接](https://github.com/docker/docker/blob/master/docs/userguide/networking/default_network/dockerlinks.md)。本部分讨论Docker容器间数据的管理。

Docker 主要有两种管理数据的方式：

* 数据卷
* 数据卷容器

## 数据卷

*数据卷*是一个或多个容器内经特殊设计的目录，它绕过了 [*Union File System*](https://github.com/docker/docker/blob/master/docs/reference/glossary.md#union-file-system)。数据卷为持久化或共享数据提供了几种有用的功能：

- 卷在容器创建时才初始化。如果容器基础镜像(base image)在指定挂载点已经包含数据，则已有数据会在新卷初始化时复制到卷中。(注意，当 [挂载一个主机目录作数据卷](#挂载一个主机目录作数据卷)时，这不适用。)
- 数据卷可在多个容器间共享和复用。
- 数据卷的修改会直接生效。
- 镜像升级不会影响数据卷的变化。
- 数据卷是持久化的，甚至容器自身被删除也如此。

设计数据卷就是为了持久化数据，独立在容器生命周期之外的。因此删除容器时，Docker *重不* 自动删除卷，对没有容器引用的卷也不会进行“垃圾回收”。

### 数据卷的添加

`docker create` and `docker run` 命令使用`-v`参数就可以给容器新增一个卷。使用多个 `-v` 就可挂载多个数据卷。现在试着挂载一个卷到Webapp容器里。

    $ docker run -d -P --name web -v /webapp training/webapp python app.py

容器内会生成一个新卷 `/webapp`。

> **注意**：
> 给镜像创建的任何容器也可以用`Dockerfile`文件里的`VOLUME`指令去新增一个或多个新卷。

### Locating a volume

可以用`docker inspect`命令查找主机上卷位置。

    $ docker inspect web

这会输出包括卷信息在内的容器配置的很多细节。输出结果类似下面这个样子：

    ...
    Mounts": [
        {
            "Name": "fac362...80535",
            "Source": "/var/lib/docker/volumes/fac362...80535/_data",
            "Destination": "/webapp",
            "Driver": "local",
            "Mode": "",
            "RW": true,
            "Propagation": ""
        }
    ]
    ...

注意，上面信息中，`Source`指明了主机位置，`Destination`指明了容器内部的卷位置。`RW`则表明该卷的可读写状态。

### 挂载一个主机目录作数据卷

除使用`-v`创建新卷外，也可以从 Docker daemon 的主机挂载一个目录给容器。

```
$ docker run -d -P --name web -v /src/webapp:/opt/webapp training/webapp python app.py
```

该命令挂载了主机目录 `/src/webapp` 到容器的 `/opt/webapp` 上。如果路径 `/opt/webapp` 已经存在于容器的镜像中，则 `/src/webapp` 挂载内容会覆盖但不会移除已有内容。一旦挂载移除，原有内容可再次访问到。写 `mount` 命令预期行为一致。

`container-dir`一定是象 `/src/docs` 这样的绝对路径。`host-dir`可以是绝对路径或是一个`name`值。如果给出`host-dir`的是绝对路径，那么Docker 会绑定挂载到指定路径上去；如果给出的是一个`name`，Docker会根据这个`name`创建一个命名卷。

`name`值必须以英文字母开头，后面可跟`a-z0-9`、`_` (下划线)、 `.` (句点) 或 `-` (连字符)。绝对路径要以 `/` 开头（正斜杠）。

比如，分别给`host-dir`一个 `/foo` 和 `foo`。前者Docker会按其路径创建，后者会创建一个命名卷。

如果在 Mac 和 Windows 上使用Docker机器，Docker daemon 限制了对 OS X 或 Windows 文件系统的访问。Docker Machine 会尽力自动共享 `/Users` (OS X) 或 `C:\Users` (Windows) 目录。所以使用 OS X 可以这样挂载文件或目录：

```
docker run -v /Users/<path>:/<container path> ...
```

使用 Windows，可以这样挂载目录：

```
docker run -v /c/Users/<path>:/<container path> ...`
```

所有其他路径都来自于虚拟机的文件系统，如果想让其他主机文件可以共享，需要做些额外的工作。 对于 VirtualBox，如果想让主机中文件夹被 VirtualBox所共享，那么可以使用 Docker 的 `-v` 参数。

挂载主机目录用于测试很管用。比如，挂载源代码到一个容器内，修改源码会在应用中实时看到结果。主机目录必须以绝对路径指明，如果该目录不存在，Docker会自动为你创建。主机路径的自动创建已被[*弃用*](https://github.com/docker/docker/blob/master/docs/deprecated.md#auto-creating-missing-host-paths-for-bind-mounts).

Docker 卷挂载的默认设置是可读写模式，也可以设置成挂载只读。

```
$ docker run -d -P --name web -v /src/webapp:/opt/webapp:ro training/webapp python app.py
```

这里，同样是挂载 `/src/webapp` 目录，添是多了 `ro` 选项以指明此挂载是只读的。

由于 [`mount`功能的局限](http://lists.linuxfoundation.org/pipermail/containers/2015-April/035788.html)，主机源目录中的子目录的移动会影响容器对主机文件系统的访问。对于一个恶意用户只要能对主机及挂载目录有访问权即可。

>**注意**：本质上，主机目录是依赖主机而存在的。因此，不要在`Dockerfile`中挂载主机目录，原因是构建的镜像应该是便于移植的。主机目录对所有潜在的主机都未必可用。

### 卷标

与SELinux类似，卷标系统（Labeling systems）要求挂载到容器中的卷内容有正确的标签。没有标签，安全机制可能阻止容器内运行的进程使用卷内容。默认情况下，Docker不改变 OS 设置的卷标。

要修改容器上下文环境的卷标，就要在挂载卷中添加后缀`:z` 或 `:Z`。后缀的意思是告诉 Docker 要对共享卷上的文件对象重新打标。`z` 选项告诉 Docker 有两个容器在共享卷上下文环境。 结果，Docker 用共享内容标签给该内容打标签。共享的卷标允许所有容器读写内容。 `Z`选项告诉 Docker 以私有非共享的标签打标内容。

### 主机文件挂载成数据卷

`-v` 参数也可用于挂载单个主机文件，不仅仅是目录。

    $ docker run --rm -it -v ~/.bash_history:/root/.bash_history ubuntu /bin/bash

这会进入新容器的bash shell，却可以拥有主机的 bash 历史记录；退出容器后，主机上也拥有了容器中键入的命令历史。

> **注意**：
> 包括`vi`和`sed --in-place`在内的很多工具经常修改文件，这可能导致 inode 变动。Docker v1.1.0 版本以后，这种情况会产生一个错误信息，象这样的 "*sed: cannot rename ./sedKdJ9Dy: Device or resource busy*"。如果想修改挂载文件，最简单的办法就是挂载它的父目录。

## 数据卷容器的创建和挂载

如果想在两个容器间共享一些持久化的数据，或者想从多个非持久化的容器中使用持久化数据，最好的办法就是创建一个命名的数据卷容器，然后从中挂载它的数据。

我们来创建带一个共享卷的新命名容器。该容器不运行应用，它复用 `training/postgres` 镜像，所有镜像都使用共同的层，以节省空间。

    $ docker create -v /dbdata --name dbstore training/postgres /bin/true

可以使用 `--volumes-from` 参数挂载 `/dbdata` 卷到其他容器。

    $ docker run -d --volumes-from dbstore --name db1 training/postgres

再添加一个：

    $ docker run -d --volumes-from dbstore --name db2 training/postgres

本例中，如果 `postgres` 镜像包含了一个称作 `/dbdata` 的目录，那么挂载 `dbstore` 容器的卷时，会隐藏所有`postgres`镜像中 `/dbdata` 内文件。仅 `dbstore` 容器中文件可见。

也可以多次使用 `--volumes-from` 参数以结合几个容器的数据卷。关于`--volumes-from`详情可参阅 [Mount volumes from container](https://github.com/docker/docker/blob/docs/docs/reference/commandline/run.md#mount-volumes-from-container-volumes-from) 中`run` 命令参考。

You can also extend the chain by mounting the volume that came from the
`dbstore` container in yet another container via the `db1` or `db2` containers.

    $ docker run -d --name db3 --volumes-from db1 training/postgres

如果移除挂载卷的容器，包括最初的 `dbstore` 容器在内，或是后面的 `db1` 和 `db2` 容器，该卷不会删除。删除硬盘上的卷必须显式地对引该卷的最近一次的容器调用 `docker rm -v`。可以升级，或者在两个容器间进行数据卷迁移。

> **注意**： 当移除一个容器而没有`-v`选项参数进行卷删除时，Docker不会进行提醒。不使用`-v`参数进行容器移除操作，会产生“无依靠”的卷；这种卷没有容器引用它。使用 `docker volume ls -f dangling=true` 可以查看这种无依靠的卷。使用`docker volume rm <volume name>` 可以移除不再需要的卷。

## 数据卷的备份、恢复或迁移

使用卷进行备份、恢复或迁移非常有用。先用 `--volumes-from` 参数创建一个挂载卷的新容器，如下：

    $ docker run --rm --volumes-from dbstore -v $(pwd):/backup ubuntu tar cvf /backup/backup.tar /dbdata

该操作启动一个新容器，并且挂载了`dbstore`容器的卷。接下来挂载一个本地方机目录 `/backup`。最后，传递一个使用 `tar` 备份 `dbdata` 卷内容到`/backup`目录内 `backup.tar`文件中的命令。命令完成，容器终止，我们得到一个 `dbdata` 卷的备份。

然后在同一容器恢复它，或者在其它容器中恢复。创建一个新容器：

    $ docker run -v /dbdata --name dbstore2 ubuntu /bin/bash

然后在新容器的数据卷中 un-tar 这个备份文件：

    $ docker run --rm --volumes-from dbstore2 -v $(pwd):/backup ubuntu bash -c "cd /dbdata && tar xvf /backup/backup.tar --strip 1"

可以使用自己喜欢的工具，利用上面的技术自动化地进行备份、迁移和恢复的工作

## 使用共享卷的重要提示

多个容器可以共享一个或多个数据卷。不过，多个容器向单个共享卷进行写操作的话，会导致数据损坏。要保证应用的设计是对共享的数据存储进行写操作。

Docker 主机可以直接访问数据卷。也就是说，可以使用常规的Linux工具对他们进行读写操作。通常不要这样操作，如果容器不知道有人直接访问的话，就可能导致数据损坏。
