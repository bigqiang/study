# Windows下使用Docker
## Windows使用Docker Toolbox安装Docker。Docker Toolbox包含以下工具:
1. Docker CLI 客户端,用于运行Docker引擎以创建航镜像和容器.
2. Docker Machine, 可以用Windows 终端运行Docker引擎的命令
3. Docker Compose, 用于运行 docker-compose 命令
4. Kitematic, 用于Docker的GUI
5. Docker QuickStart shell, 用于预先配置Docker命令行环境
6. Oracle VM VirtualBox

Docker引擎使用了特定Linux内核,所以Windows不能原生运行Docker引擎. 必须使用Docker Machine命令 docker-machine 来为电脑创建添加一个小型Linux VM. 在Windows系统中,这个VM包含了Docker引擎.

## 理解镜像和容器
 Docker引擎提供了能运作镜像及容器的核心。Docker安装完成后可以运行命令“docker run hello-world”，通过该命令引擎完成了核心任务，这条命令有三部分组成：
  1. docker 告诉你使用的操作系统是docker程序
  2. run    子命令，创建和运行Docker容器
  3. hello-world    告知Docker加载哪个镜像到该容器中
  
容器是一个精简到基础版本的Linux操作系统。镜像是加载到容器中的软件。当你运行上述命令时，引擎会有如下动作：
* 检测是否存在 hello-world 软件镜像
* 从Docker Hub上下载该镜像
* 把镜像 加载到容器中“运行”它

因构建方法的不同，一个镜像可能只运行一个简单单一命令就退出了，就象hellow-world一样。但是一个Docker的镜像也可以做的更复杂。一个镜像可以启动复杂如数据库的软件，再等你或他人添加存储数据备用，然后处理下一个人。

是谁构建的 hello-world 软件呢？本例是由Docker做的，但任何人都可以做。Docker引擎允许任何人或公司通过Docker 镜像来创建分享软件。使用Docker引擎，就不必担心你的电脑是否能跑一个Docker 镜像中的软件，Docker容器总能让它跑起来。

## 实战：查找和运行whalesay 镜像   
   全世界都有人创建Docker 镜像，通过浏览Docker Hub找到这些镜像。本节找的镜像要供本次教程启盟使用。
   
### 步骤1：找到whalesay 镜像
 1. 打开浏览器访问 Docker Hub (https://hub.docker.com)。
 2. 点击 Browse(Explore) & Seache
 3. 搜索框中输入 whalesay
 4. 在结果列表中点击 docker/whalesay 镜像 ，浏览器会显示whalesay 镜像 信息的库内容页面。包含了诸如镜像中有何种软件、如何使用的信息。可以注意到whale 镜像是基于Ubuntu的Linux发行包。
### 步骤2：运行 whalesay 镜像。
 1. 先打开 Docker Quickstart Terminal
 2. 输入 docker run docker/whalesay cowsay boo 命令，然后回车
   第一次运行该软件 镜像，docker命令先在本地查找，如果不存在，会从hub上获取.
 3. 输入 “docker images” 命令可列出本地所有镜像。

## 构建自己的镜像




