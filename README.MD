# 学习笔记
出处多源于相关的官方文档,结合自身情况做一些翻译整理,用于个人工作

# [Docker](https://github.com/bigqiang/study/tree/master/docker)

# Windows 10 (64) IIS+MySQL+PHP+memcached 开发环境部署

## 1. PHP 5.6.13 安装

在 http://windows.php.net/download 下找到相应的 VC11 x86 Non Thread Safe 版本下载。注因为除php7版本外的其他php的x64版本都属于试验性质，暂不使用。

本例下载 php-5.6.13-nts-Win32-VC11-x86.zip，解压到 “D:\Program Files (x86)\PHP” 下。
然后到 http://pecl.php.net/packages.php 中直载所需要的扩展文件放到 ext 文件夹下。

修改 php.ini 文件 将其中 `extension=` 的分号去除，保证其相关的扩展都在 ext 文件夹下都存在。
原则上，该文件应该复制到 c:\windows 目录下。但似乎，在 php 文件下也是可以的。

在php目录下命令行下执行 `php -v` 命令，如果正确运行则安装正确，如果出现 msvcr110.dll 缺失的报错，则需要安装 [Visual C++ Redistributable for Visual Studio 2012 Update 4](https://www.microsoft.com/zh-CN/download/details.aspx?id=30679) 。注意安装的程序要与php的对应，比如 x86 则对应下载 x86 的，这与 Windows 版本是否是64位无关。下载后安装即可。

## 2. MySQL 安装

这里以当前最新版本 5.7 社区版本来说明。先[官网网址下载相应的版本](http://dev.mysql.com/downloads/mysql/)，这里下载的是 64 位的版本。下载后，解压释放到 `D:\Program Files\MySQL\MySQL Server 5.7` 路径下面。

在环境变量里面设置 一条路径 `D:\Program Files\MySQL\MySQL Server 5.7\bin`。

配置 my.ini 文件，其中 `basedir = "D:/Program Files/MySQL/MySQL Server 5.7/"`；`datadir = "E:/MySQLData/"`，后者指向的是数据库文件的路径。

以管理员命令窗口下进入 bin 目录，在其中执行`mysqld install MySQL --defaults-file="D:\Program Files\MySQL\MySQL Server 5.7\my.ini"
`，可安装mysql服务。如果数据库文件尚未创建会显示错误信息`Fatal error: Can't open and lock privilege tables: Table 'mysql.user' doesn't existFor more information, see Help and Support Center at http://www.mysql.com.`。此时执行 `mysqld --initialize`，可进行初始化数据库创建，如果指定的文件夹已经有文件可能会报错，不妨清空这个文件夹再进行操作。然后，`net start mysql` 启动服务。

服务安装成功后，服务密码为空，第一次执行命令需要重设密码：
```
mysql -u root -p

mysql> use mysql
Database changed

mysql> update user set PASSWORD = PASSWORD('你的新密码') where USER='root' and HOST='localhost';
Query OK, 1 row affected (0.05 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> flush privileges;
mysql> exit

```

### 一些常见错误处理:
* 错误信息“'performance_schema.session_status' doesn't exist”。
执行命令`mysql_upgrade -u root -p --force`，然后重启mysql服务即可


## 3. IIS安装

控制面板 -> 程序和功能 -> 启用或关闭 Windows 功能 -> 勾选上 `Internet Information Services` 下的 `Web 管理工具` 和 `万维网服务`。`万维网服务`中的 `应用程序开发功能` 下的 `CGI`/`ISAPI扩展`/`ISAPI筛选器`一定要选中。保存退出。

进入 管理工具 中的 `Internet Information Services` 中，在 `处理程序映射`中，点击`添加模块映射`，请求路径填写`*.php`，模块选择`FastCgiModule`，可执行文件填写`D:\Program Files (x86)\PHP\php-cgi.exe`，即php安装路径下相应文件；名称填写`php`即可。

回到自己创建的网站上，点击`基本设置`，将物理路径指向自己的站点代码文件。

## 4. memcached 安装

本处下载的是 64位的版本，[链接在这里](http://www.urielkatz.com/archive/detail/memcached-64-bit-windows/) 。

解压释放到 `D:\memcached-win64` 下，然后进入其路径下的命令行，执行以下命令：
```
D:\memcached-win64\>memcached -d install(安装服务)

D:\memcached-win64\>memcached -d start(启动memcache服务，默认分配64M内存，使用11211端口)
```