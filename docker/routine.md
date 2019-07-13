# 一些常见操作

## 使用镜像定制一个Web服务器容器
```
$ docker run --name webserver -d -p 80:80 nginx
```
以 `nginx` 镜像启动一个容器，容器命名为 `webserver`，并且映射80端口。这样就可以用浏览器访问 `nginx` 服务器。本机上运行可用 http://localhost 访问默认页面。

## 进入指定容器终端
```
$ docker exec -it webserver bash
```
以交互式终端方式进入 `webserver` 容器，执行 `bash` 命令，可进行Shell操作。

## 将容器修改的内容记录于容器存储层，形成新的镜像
进入容器终端后，可以修改配置或相关内容。在容器退出后，这些内容会消失。如果想成为镜像保存下来，要用到 `docker commit` 命令。换言之，就是在原有镜像上叠加上容器的存储层，构成新镜像。
格式：
```
docker commit [选项] <容器ID或容器名> [<仓库名>[:<标签>]]
```
命令示例：
```
$ docker commit \
    --author "bigqiang <bigqiang@gmail.com>" \
    --message "修改了默认网页" \
    webserver \
    nginx:v2
sha256:07e33465974800ce65751acc279adc6ed2dc5ed4e0838f8b86f0c87aa1795214
```
其中 `--author` 是指定修改的作者，而 `--message` 则是记录本次修改的内容。这点和 git 版本控制相似，不过这里这些信息可以省略留空。

不建议使用该命令制作镜像。

## 查看镜像历史记录
```
$ docker history nginx:latest
```