# Windows下使用Docker 
## Windows使用Docker Toolbox安装Docker。Docker Toolbox包含以下工具:
1. Docker CLI 客户端,用于运行Docker引擎以创建航镜像和容器.
2. Docker Machine, 可以用Windows 终端运行Docker引擎的命令
3. Docker Compose, 用于运行 docker-compose 命令
4. Kitematic, 用于Docker的GUI
5. Docker QuickStart shell, 用于预先配置Docker命令行环境
6. Oracle VM VirtualBox

Docker引擎使用了特定Linux内核,所以Windows不能原生运行Docker引擎. 必须使用Docker Machine命令 `docker-machine` 来为电脑创建添加一个小型Linux VM. 在Windows系统中,这个VM包含了Docker引擎.

## 理解镜像和容器
 Docker引擎提供了能运作镜像及容器的核心。Docker安装完成后可以运行命令`docker run hello-world`，通过该命令引擎完成了核心任务，这条命令有三部分组成：
  1. docker 告诉你使用的操作系统是docker程序
  2. run    子命令，创建和运行Docker容器
  3. hello-world    告知Docker加载哪个镜像到该容器中
  
容器是一个精简到基础版本的Linux操作系统。镜像是加载到容器中的软件。当你运行上述命令时，引擎会有如下动作：
* 检测是否存在 `hello-world` 软件镜像
* 从Docker Hub上下载该镜像
* 把镜像 加载到容器中“运行”它

因构建方法的不同，一个镜像可能只运行一个简单单一命令就退出了，就象`hellow-world`一样。但是一个Docker的镜像也可以做的更复杂。一个镜像可以启动复杂如数据库的软件，再等你或他人添加存储数据备用，然后处理下一个人。

是谁构建的 `hello-world` 软件呢？本例是由Docker做的，但任何人都可以做。Docker引擎允许任何人或公司通过Docker 镜像来创建分享软件。使用Docker引擎，就不必担心你的电脑是否能跑一个Docker 镜像中的软件，Docker容器总能让它跑起来。

## 实战：查找和运行whalesay 镜像   
   全世界都有人创建Docker 镜像，通过浏览Docker Hub找到这些镜像。本节找的镜像要供本次教程启盟使用。
   
### 步骤1：找到whalesay 镜像
 1. 打开浏览器访问 Docker Hub (https://hub.docker.com)。
 2. 点击 Browse(Explore) & Seache
 3. 搜索框中输入 whalesay
 4. 在结果列表中点击 docker/whalesay 镜像 ，浏览器会显示whalesay 镜像 信息的库内容页面。包含了诸如镜像中有何种软件、如何使用的信息。可以注意到whale 镜像是基于Ubuntu的Linux发行包。
### 步骤2：运行 whalesay 镜像。
 1. 先打开 Docker Quickstart Terminal
 2. 输入 `docker run docker/whalesay cowsay boo` 命令，然后回车
   第一次运行该软件 镜像，docker命令先在本地查找，如果不存在，会从hub上获取.
 3. 输入 `docker images` 命令可列出本地所有镜像。

## 构建自己的镜像
whalesay镜像可以升级。不必思考要说什么感觉会不错。只要输入一堆东西让whalsay去讲。如：
```docker run docker/whalesay cowsay boo-boo```

本节会通过, you will improve the whalesay image by building a new version that “talks on its own” and requires fewer words to run.

### 步骤1: 打开 Docker Quickstart Terminal
### 步骤2: 写 Dockerfile 文件
本步骤中，要使用 Windows Notepad 应用写一个简短的 Dockerfile 文件。Dockerfile文件可描述了构成镜像的软件元件，也描述了在容器中要使用或运行命令的环境。本例中 Dockerfile 文件会很简短。

1. `cd testdocker`
进入你的工作目录

2. 创建 Dockerfile 文件，确保文件名中字母D是大写，Linux文件名区别大小写
`$ touch Dockerfile`

3. 输入`notepad Dockerfile&`打记事本（不要漏写`&`符号）

4. 在文本中输入`FROM docker/whalesay:latest`，形式如下：
![Line one](https://docs.docker.com/windows/images/note-pad2.png)

`FROM`关键词表示向Docker表明,你的镜像是基于哪个镜像而来，本例中你的新镜像是由已存在的whalesay镜像修改而来。

5. 现在添加`fortunes`程序到镜像
![Line two](https://docs.docker.com/windows/images/note-pad3.png)

`fortunes`程序有一个命令可以为我们的 `whalesay` 提供名言警句。 首先安装这个程序，安装 fortune 程序要使用`apt-get`程序。

6. 一旦镜像有了所需软件，就可以在镜像加载后给软件指令
![Line three](https://docs.docker.com/windows/images/note-pad4.png)
这行命令是告诉`fortunes`程序把它的名言警句发送给`cowsay`程序

7. 保存 Dockerfile 文件后退出
到此为止，你已在Dockerfile文件中描述好了所有软件元件及操作，为构建新的镜像已经万事俱备。

### 步骤3: 从 Dockerfile 构建镜像
1. 确保Dockerfile文件在当前工作目录中
```
$ cat Dockerfile
FROM docker/whalesay:latest

RUN apt-get -y update && apt-get install -y fortunes

CMD /usr/games/fortune -a | cowsay
```

2. 构建镜像，输入命令`docker build -t docker-whale .` （不要遗漏`.`号）。效果如下：

```
$ docker build -t docker-whale .
Sending build context to Docker daemon 158.8 MB
...snip...
Removing intermediate container a8e6faa88df3
Successfully built 7d9495d03763
```
该命令要运行几秒种后才报告结果。在对新镜像操作前，先花几分钟了解一下Dockfile文件构建过程。

### 步骤4: 了解构建过程        
`docker build -t docker-whale .`命令会在当前目录中调用Dockerfile文件，然后在本地构建名为`docker-whale`的镜像。该命令耗时约1分钟左右，返回结果内容较长且复杂，本节会解释这些信息。

首先 Docker 会检查构建所需的所有构件。
```
Sending build context to Docker daemon 158.8 MB
```

然后，Docker加载whalesay镜像。前面已经下载该镜像，所以Docker不必再下载了。
```
Step 0 : FROM docker/whalesay:latest
 ---> fb434121fc77
```

Docker moves onto the next step which is to update the apt-get package manager. This takes a lot of lines, no need to list them all again here.

Step 1 : RUN apt-get -y update && apt-get install -y fortunes
 ---> Running in 27d224dfa5b2
Ign http://archive.ubuntu.com trusty InRelease
Ign http://archive.ubuntu.com trusty-updates InRelease
Ign http://archive.ubuntu.com trusty-security InRelease
Hit http://archive.ubuntu.com trusty Release.gpg
....snip...
Get:15 http://archive.ubuntu.com trusty-security/restricted amd64 Packages [14.8 kB]
Get:16 http://archive.ubuntu.com trusty-security/universe amd64 Packages [134 kB]
Reading package lists...
---> eb06e47a01d2
Then, Docker installs the new fortunes software.

Removing intermediate container e2a84b5f390f
Step 2 : RUN apt-get install -y fortunes
 ---> Running in 23aa52c1897c
Reading package lists...
Building dependency tree...
Reading state information...
The following extra packages will be installed:
  fortune-mod fortunes-min librecode0
Suggested packages:
  x11-utils bsdmainutils
The following NEW packages will be installed:
  fortune-mod fortunes fortunes-min librecode0
0 upgraded, 4 newly installed, 0 to remove and 3 not upgraded.
Need to get 1961 kB of archives.
After this operation, 4817 kB of additional disk space will be used.
Get:1 http://archive.ubuntu.com/ubuntu/ trusty/main librecode0 amd64 3.6-21 [771 kB]
...snip......
Setting up fortunes (1:1.99.1-7) ...
Processing triggers for libc-bin (2.19-0ubuntu6.6) ...
 ---> c81071adeeb5
Removing intermediate container 23aa52c1897c
Finally, Docker finishes the build and reports its outcome.

Step 3 : CMD /usr/games/fortune -a | cowsay
 ---> Running in a8e6faa88df3
 ---> 7d9495d03763
Removing intermediate container a8e6faa88df3
Successfully built 7d9495d03763


### 步骤5:

# Mac 使用