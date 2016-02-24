# Dockerfile 文件格式详解

Docker 通过读取`Dockerfile`文件可自动构建镜像。`Dockerfile`是纯文本格式文件。使用`docker build`能创建一个执行多个命令行指令的构建版本。

本页面描述了能在`Dockerfile`中使用的命令。阅读时请参考 [`Dockerfile` Best
Practices](dockerfile_best-practices.md) 。



>参考中文:
> 1. http://dockone.io/article/103
> 2. http://www.jb51.net/os/other/392845.html
> 3. http://blog.csdn.net/we_shell/article/details/38445979
> 4. http://blog.csdn.net/wsscy2004/article/details/25878223



## 用法

[`docker build`](https://github.com/docker/docker/blob/master/docs/reference/commandline/build.md) 命令根据`Dockerfile`及 *上下文环境* 来构建一个镜像。构建的上下文环境是指该文件所在的特定位置`PATH`或`URL`。`PATH`是指本地文件系统的目录。`URL`是指 Git 版本库的位置。

上下文环境可以被递归处理。所以，`PATH`包含任意子目录，并且`URL`包含了版本库及其子模块。使用当前目录的上下文环境可以简化命令：

    $ docker build .
    Sending build context to Docker daemon  6.51 MB
    ...

该构建是由 Docker daemon 运行的，而非 CLI。构建进程做的头一件事就是把整个上下文环境（递归）发送给daemon。多数情况下，最好是在一个空目录中做上下文环境启动，把
Dockerfile 文件放在该目录中。仅添加构建该 Dockerfile 文件所必须的文件在里面。

>**警告**: 不要用目录 `/` 作 `PATH` ，它可能让构建传送硬盘所有内容给 Docker daemon。

为了使用在构建上下文环境中的某个文件，`Dockerfile` 文件中的指令要具体地指定该文件。例如 `COPY` 指令。为提升构建性能，在上下文环境目录中可以添加一个 `.dockerignore` 文件以排除不要的文件和目录 。 详情查看本文档的如何创建一个[`.dockerignore`文件](#dockerignore文件)。

习惯上，`Dockerfile`被称作`Dockerfile`，并且位于上下文环境的根路径中。`docker build`用`-f`标志参数指向文件系统中任意位置的 Dockerfile 文件。

    $ docker build -f /path/to/a/Dockerfile .

如果构建成功，可以指定一个版本库并标记存储这个新镜像：

    $ docker build -t shykes/myapp .

构建完成后，如果想标记该镜像到多个版本库中，可以在运行`build`命令时添加多个 `-t` 参数：

    $ docker build -t shykes/myapp:1.0.2 -t shykes/myapp:latest .

在最终生成新镜像的ID前，Docker daemon 是按顺序依次执行 `Dockerfile` 文件中指令的，同时在必要的情况下，会把每条指令的执行结果提交到新镜像中。Docker daemon 会自动清理干净发送过来的上下文环境描述信息。

注意，每条指令都是独立运行的，都会产生一个新镜像，所以`RUN cd /tmp`不会对下一条指令有任何影响。

只要有可能，Docker就会重用中间镜像(cache)，这可以极大加速`docker build`进程。可通过控制台中的`Using cache`信息显示这一点。
(详情参阅 [Build cache section](dockerfile_best-practices.md#build-cache)) `Dockerfile`文件最佳实践指南：

    $ docker build -t svendowideit/ambassador .
    Sending build context to Docker daemon 15.36 kB
    Step 0 : FROM alpine:3.2
     ---> 31f630c65071
    Step 1 : MAINTAINER SvenDowideit@home.org.au
     ---> Using cache
     ---> 2a1c91448f5f
    Step 2 : RUN apk update &&      apk add socat &&        rm -r /var/cache/
     ---> Using cache
     ---> 21ed6e7fbb73
    Step 3 : CMD env | grep _TCP= | (sed 's/.*_PORT_\([0-9]*\)_TCP=tcp:\/\/\(.*\):\(.*\)/socat -t 100000000 TCP4-LISTEN:\1,fork,reuseaddr TCP4:\2:\3 \&/' && echo wait) | sh
     ---> Using cache
     ---> 7ea8aef582cc
    Successfully built 7ea8aef582cc

构建完成后就要研究一下 [*push 一个版本库到它的 registry*](https://github.com/docker/docker/blob/master/docs/userguide/containers/dockerrepos.md#contributing-to-docker-hub).

## Format

`Dockerfile`文件格式如下:

    # Comment
    INSTRUCTION arguments

指令 INSTRUCTION 不区别大小写，不过习惯上为便于和参数区别都会是大写的。

Docker 按顺序运行`Dockerfile`文件中的指令。 **第一条指令必须是 \`FROM\`** ，这是为了指名所构建镜像的 [*Base Image*](https://github.com/docker/docker/blob/master/docs/reference/glossary.md#base-image)。

Docker 会把`#`开始的行作为注释。在其他任何位置的 `#` 的标记会被当作参数，如：

    # Comment
    RUN echo 'we are running some # of cool things'

以下是 `Dockerfile` 文件构建镜像使用的指令集。

### Environment replacement

环境变量（由[`ENV`语句](#env)声明）使用指定指令作变量，`Dockerfile`文件可以解释它。也可用类变量的语法进行转义处理成语句。

`Dockerfile` 文件中，可用`$variable_name`或是`${variable_name}`来引用环境变量。二者是等效的。大括号语法常用于处理变量名无空格时的问题，比如`${foo}_bar`。

`${variable_name}` 语法也支持几种标准`bash`修饰符，如下：

* `${variable:-word}` 表示如果 `variable` 值已经设置，那么结果就是该值；否则，`variable`的结果值是`word`。
* `${variable:+word}` 表示如果 `variable` 值已经设置，那么结果就是`word`；否则，结果是空字符串。

任何情况下，`word`可以是任意字符串，包含其他环境变量。

可在变量前加上`\`进行转义，如： `\$foo` 或 `\${foo}`，这会在字面上分别转义成 `$foo` 和 `${foo}`。

示例 (解析后的结果放在`#`后)：

    FROM busybox
    ENV foo /bar
    WORKDIR ${foo}   # WORKDIR /bar
    ADD . $foo       # ADD . /bar
    COPY \$foo /quux # COPY $foo /quux

`Dockerfile`中支持以下环境变量列表中的指令：

* `ADD`
* `COPY`
* `ENV`
* `EXPOSE`
* `LABEL`
* `USER`
* `WORKDIR`
* `VOLUME`
* `STOPSIGNAL`

还有：

* `ONBUILD` (在与以上支持的指令组合时)

> **注意**:
> 1.4版以前，`ONBUILD`指令**不支持**环境变量，即使在与以上任何指令组合使用时。

环境变量的替换在整个命令行中，变量只用同一个值。举例说明：

    ENV abc=hello
    ENV abc=bye def=$abc
    ENV ghi=$abc

这会产生这样的效果，`def`得到是`hello`的值，而不是`bye`的，第二行整个命令中abc只能用一个值 。但是，`ghi`得到的是`bye`的值，因为它不是设置`abc`值`bye`命令行的一部分。

### .dockerignore文件

docker CLI 给 docker daemon 发送上下文环境前，先在上下文环境的根目录中寻找名为`.dockerignore`的文件。如果存在，CLI 会修改上下文环境描述，把匹配该文件中描述的文件和目录排除出去。可避免不必要地发送超大或敏感内容的文件和目录给 daemon，也可避免使用`ADD`或`COPY`把他们添加到镜像中的可能性。

CLI 会把`.dockerignore`文件解析成一个独立行的模式列表，模式类似于 Unix shell 的 file globs。为了模式匹配，上下文环境的根目录会被看作即是工作目录也是根目录。例如模式 `/foo/bar` 和 `foo/bar`表示，`PATH`下(或是位于`URL`的的git版本库的根目录下)`foo`子目录中名为`bar`的文件或目录被排除。

这里有个`.dockerignore`文件示例：

```
    */temp*
    */*/temp*
    temp?
```

该文件会产生下述构建操作：

| 规则           | 操作                                                                               |
|---------------|----------------------------------------------------------------------------------|
| `*/temp*`     | 排除任何根目录下任何直接以 `temp` 开始的文件和文件夹。如，文本文件 `/somedir/temporary.txt` 会被排除，因为它的开头是`/somedir/temp`。      |
| `*/*/temp*`    | 排除根目录下两层子目录下以 `temp` 起始的文件和目录。如，`/somedir/subdir/temporary.txt` 会被排除。 |
| `temp?`       | 排除根目录下对`temp`扩展了一个字符的所有文件和目录名。如，`/tempa` 和 `/tempb`都被排除。


模式匹配是通过 Go 的[filepath.Match](http://golang.org/pkg/path/filepath#Match)规则完成的。通过 Go 的 [filepath.Clean](http://golang.org/pkg/path/filepath/#Clean) 进行预处理步骤会删除前导和尾部空格，并且清除`.` 和 `..` 元素。预处理结束后的空白行会被忽略。

除了 Go 的 filepath.Match 规则，Docker也支持特殊通配符字串`**`，它可匹配任意数量(含0)的目录。如 `**/*.go` 会排除所有目录中（含构建上下文环境中的根目录）所有以 `.go` 结尾的文件。

以`!`起始的行用于设定排除规则的例外项。如下示例：

```
    *.md
    !README.md
```

*除了* `README.md`文件外，排除上下文环境中所有markdown文件。

`!`例外规则的位置造成的影响：匹配特殊文件的`.dockerignore`中最后一行会决定该文是包括还是排除。思考以下例子：

```
    *.md
    !README*.md
    README-secret.md
```

在上下文环境中除了 README 文件（不`README-secret.md`文件），排除所有 markdown 文件。

现在考虑下面例子：

```
    *.md
    README-secret.md
    !README*.md
```

包括所有 README 文件，中间行无效，因为`!README*.md`匹配`README-secret.md`规则，而且在最后一行。

即使用 `.dockerignore` 文件排除 `Dockerfile` 和 `.dockerignore` 文件，这些文件仍然会发送给 daemon，因为这是完成任务必须文件。不过 `ADD` 和 `COPY` 命令并不会把他们复制到镜像中。

最后，还需要指定哪些文件要包含而不是排除在上下文环境中。为达成目标，要指定 `*` 作第一行模式，后跟一条或多条`!`例外规则模式 。

**注意**: 因历史原因，模式 `.` 会被忽略。

## FROM

    FROM <image>

或

    FROM <image>:<tag>

或

    FROM <image>@<digest>

该指令设置 [*Base Image*](https://github.com/docker/docker/blob/master/docs/reference/glossary.md#base-image)，用于后续指令。因此，合法的 `Dockerfile` 文件第一条指令一定是 `FROM`。该镜像可以是任何合法镜像，它可以很容易从 [*Public Repositories*](https://github.com/docker/docker/blob/master/docs/userguide/containers/dockerrepos.md) 中 **pull** 下一个镜像.

- `FROM` 必须是 `Dockerfile` 中第一行非注释的指令。

- `FROM` 为了创建多个镜像，单一 `Dockerfile` 文件中可以出现多次。仅在每个新 `FROM` 命令前记下通过提交输出的最后的镜像ID。

- `tag` 或 `digest` 值是可选的。如果省略他们，构建器会把 `latest` 作默认值。如果与 `tag` 值不匹配，构建器会返回一个错误。

## MAINTAINER

    MAINTAINER <name>

该指令允许设置产生镜像的 *Author* 字段。

## RUN

它有两种形式：

- `RUN <command>` (*shell* 形式，command 运行在一个 shell 中 - `/bin/sh -c`)
- `RUN ["executable", "param1", "param2"]` (*exec* 形式)

该指令执行当前镜像顶部新层中的任意命令并提交（commit）结果。提交结果的镜像会用于`Dockerfile`文件中下一步指令。它等效于如下形式：
```
docker run <image> <command>
docker commit <container_id>
```

`RUN`指令分层和生成提交符合 Docker 的核心概念：提交方便，可从镜像历史任一点创建容器，非常类似源码版本控制。

*exec* 形式可以避免shell 字串篡改，而且可以在没有包含 `/bin/sh`的基础镜像(base image)使用  `RUN` 命令。

*shell* 形式可以使用 `\` (反斜杠)对单行 RUN 指令续行。举例如下：
```
RUN /bin/bash -c 'source $HOME/.bashrc ;\
echo $HOME'
```
上例等效于下面一行示例：
```
RUN /bin/bash -c 'source $HOME/.bashrc ; echo $HOME'
```

> **注意**：
> 要使用'/bin/sh'之外的其他 shell，就要在 *exec* 形式中传递想要shell的参数，如：
> `RUN ["/bin/bash", "-c", "echo hello"]`

> **注意**：
> *exec*形式会被解析成一个JSON数组，这意味着必须使用双引号 (") 而不是单引号(')把文字内容括起来。

> **注意**：
> 与 *shell* 形式不同，*exec*形式不调用命令的 shell。这意味着正常的 shell 操作不起作用。例如，
> `RUN [ "echo", "$HOME" ]` 其中 `$HOME` 不会做变量替换。
> 要使用 shell 操作，要么使用 *shell*形式，要么直接执行 shell ，如：`RUN [ "sh", "-c", "echo", "$HOME" ]`。

`RUN`指令的缓存在下次构建期间不会自动失效。比如象 `RUN apt-get dist-upgrade -y` 这样指令的缓存会在下次构建时重用。使用`--no-cache` 标志参数，`RUN`指令的缓存会失效，如：`docker build --no-cache`。

详情参阅 [`Dockerfile`最佳实践指南
guide](dockerfile_best-practices.md#build-cache)。

`ADD` 指令操作的 `RUN`指令的缓存也会失效。详情查看[下面](#add)。

### 已知问题 (RUN)

- [Issue 783](https://github.com/docker/docker/issues/783) 文件权限许可问题，在使用 AUFS 文件系统时会发生。比如尝试 `rm` 一个文件时。

  对于最近的aufs版本（`dirperm1`挂载项可设置）的几个系统，Docker会尝试自动通过`dirperm1`选项挂载该层来修复该问题。 `dirperm1`选项详情可查看 [`aufs` man page](http://aufs.sourceforge.net/aufs3/man.html)。

  如果系统不支持 `dirperm1`，该 issue 提供了一个解决办法。

## CMD

该指令有三种形式：

- `CMD ["executable","param1","param2"]` (*exec*形式, 这是首选形式)
- `CMD ["param1","param2"]` (用作给ENTRYPOINT提供默认参数)
- `CMD command param1 param2` (*shell*形式)

一个`Dockerfile`文件中只能有一个`CMD`指令。如何列出多个`CMD`指令，则只有最后一个`CMD`指令生效。

**`CMD`的主要用途是给要执行的容器提供默认值** ，这些默认值可包含一个可执行程序；或者也可省略这个可执行程序，在此情况下，还必须指定一个`ENTRYPOINT`指令。

> **注意**：
> `CMD`常用于给`ENTRYPOINT`指令提供默认参数，`CMD`和`ENTRYPOINT`指令应该以JSON数组格式指定。

> **注意**：
> *exec*形式会解析成JSON数组，这意味着必须使用双引号(")而不是单引号(')把文字内容括起来。

> **注意**：
> 与*shell*形式不同，*exec*形式不会调用shell命令。这意味着正常的shell操作不会生效。
> 如`CMD [ "echo", "$HOME" ]`，这不会对`$HOME`做变量替换。
> 想要shell操作，要么用*shell*形式，要么直接执行一个shell。如 `CMD [ "sh", "-c", "echo", "$HOME" ]`。

在使用shell或exec格式时，`CMD`指令会在运行镜像时设置命令去执行。

如果使用了`CMD`的*shell*形式，那么`<command>`会在`/bin/sh -c`中执行：

    FROM ubuntu
    CMD echo "This is a test." | wc -

如果想在**无shell**的环境中**运行自己的** `<command>`，就必须以JSON数组的格式表示这个命令，并且给出可执行程序的完整路径。
**数组形式是`CMD`首选格式。** 在数组中，其他任何参数都要单独表示成字符串：

    FROM ubuntu
    CMD ["/usr/bin/wc","--help"]

如果想让自己的容器每次运行相同的可执行程序，就要考虑 `ENTRYPOINT` 结合 `CMD`一起使用。可查看 [*ENTRYPOINT*](#entrypoint)。

如果用户指定了`docker run`参数，那么这些参数会覆盖`CMD`指定的默认值。

> **注意**：
> 不要混淆 `RUN`和`CMD`。`RUN`实际上运行一个命令就提交该命令的结果；`CMD`在构建时不执行任何命令，不过指明了镜像要执行的命令。

## LABEL

    LABEL <key>=<value> <key>=<value> <key>=<value> ...

该指令给镜像添加元数据。`LABEL`是一个键值对。为了在`LABEL`值中包括空格，使用引号和反斜杠可在命令行中解析。几个用法示例：

    LABEL "com.example.vendor"="ACME Incorporated"
    LABEL com.example.label-with-value="foo"
    LABEL version="1.0"
    LABEL description="This text illustrates \
    that label-values can span multiple lines."

一个镜像存在一个以上的label。要使用多个 label，Docker推荐把多个 label 结合成单一一个 `LABEL` 指令中 。每一个`LABEL`指令都会生成一个新层，如果用了许多label，那么这个新层生成的镜像会很低效。本例示范生成一个单一镜像层：

    LABEL multi.label1="value1" multi.label2="value2" other="value3"

以上也可写作：

    LABEL multi.label1="value1" \
          multi.label2="value2" \
          other="value3"

`FROM`镜像中，label是`LABEL`递增（？）。 如果 Docker 遇到一个已经存在的label键值，新值会覆盖前面所有同键名的值。

使用 `docker inspect` 命令可以查看镜像的label：

    "Labels": {
        "com.example.vendor": "ACME Incorporated"
        "com.example.label-with-value": "foo",
        "version": "1.0",
        "description": "This text illustrates that label-values can span multiple lines.",
        "multi.label1": "value1",
        "multi.label2": "value2",
        "other": "value3"
    },

## EXPOSE

    EXPOSE <port> [<port>...]

该指令用于告知 Docker 当前容器在运行时要侦听的网络端口。`EXPOSE`不让容器端口为主机所访问的话，必须使用 `-p` 标志参数发布了端口范围，或者使用 `-P` 标志参数发布所有公开的端口。可以泄露一个端口号，对外以另一个端口号发布。

关于对主机设置端口重定向，参考 [using the -P
flag](https://github.com/docker/docker/blob/master/docs/reference/run.md#expose-incoming-ports)。 Docker 网络特性支持在网络内不泄露端口的情况下创建网络，详情参阅 [overview of this
feature](https://github.com/docker/docker/blob/master/docs/userguide/networking/index.md))。

## ENV

    ENV <key> <value>
    ENV <key>=<value> ...

该指令设置环境变量 `<key>` 值为 `<value>`。该值会存在于 `Dockerfile`后续命令环境中，并会多次 [内联替换](#environment-replacement)。

`ENV`指令有两种形式。第一种形式，`ENV <key> <value>`，
给单一变量设置一个值。第一个空格后的整个字串会当作 `<value>`，包括象空格和引号这样的字符。

第二种形式，`ENV <key>=<value> ...`，允许一次给多个变量赋值。在命令行解析时，要用引号和反斜杠把值内的空格包含起来，例如：

    ENV myName="John Doe" myDog=Rex\ The\ Dog \
        myCat=fluffy

以及

    ENV myName John Doe
    ENV myDog Rex The Dog
    ENV myCat fluffy

二者在最终容器中产生相同结果，不过第一种形式是首选，因为它生成了单一一个缓存层。

当容器运行一个生成结果的镜像时，使用`ENV`设置的环境变量依然会存在。可以通过 `docker inspect`查看它的值，也可以使用`docker run --env <key>=<value>` 去修改它们。

假如你安装了JAVA程序，需要设置JAVA_HOME，那么可以在Dockerfile中这样写：`ENV JAVA_HOME /path/to/java/dirent`

> **注意**：
> 环境持久性会导致不可预料的副效应。例如，设置 `ENV DEBIAN_FRONTEND noninteractive` 可能使基于Debian镜像的apt-get
用户迷惑。要为一条命令只设置一个值，可以使用 `RUN <key>=<value> <command>`。

## ADD

它有两种形式：

- `ADD <src>... <dest>`
- `ADD ["<src>",... "<dest>"]` (对于包含空格的路径是必选形式)

该指令会复制从`<scr>`指定的新文件、目录或远程文件URL，把它们添加到`<dest>`路径指定的容器文件系统中。

可以指定多个`<src>`源，如果他们是文件或目录，那么他们必须与要构建源文件夹相关联(构建的上下文环境)。

每一个 `<src>` 都可以包含通配符，并且可用 Go 的 [filepath.Match](http://golang.org/pkg/path/filepath#Match) 规则进行匹配通过。例如：

    ADD hom* /mydir/        # 添加所有以"hom"开始的文件
    ADD hom?.txt /mydir/    # ? 可被任意一个单字符替换，如"home.txt"

`<dest>`要写成绝对路径，或是写成相对于`WORKDIR`的路径，源会被复制到目标容器内`<dest>`指定的路径下。

    ADD test relativeDir/          # 复制到 `WORKDIR`/relativeDir/
    ADD test /absoluteDir/         # 复制"test" 到 /absoluteDir/

所有新文件和目录在创建时会带一个值为0的 UID 和 GID。

如果 `<src>` 是一个远程文件 URL，目标拥有 600 许可权限。如果获取的远程文件有一个HTTP `Last-Modified` header，则该来自该header的时间戳会设置目标文件的`mtime`。不过，象任何其他在`ADD`操作期间处理的文件一样，`mtime` 不会包含在目标文件中，不论该文件改变还是缓存该更新。

> **注意**：
> 如果是通过把`Dockerfile`传递给 STDIN (`docker build - < somefile`)来构建，那么就没构建的上下文环境。因此 `Dockerfile` 仅能包含一个基于`ADD`指令的URL。也可以传递一个压缩文档给 STDIN: (`docker build - < archive.tar.gz`)，文档根目录中的 `Dockerfile`和其他文档可以用在构建的上下文环境中。

> **注意**：
> 如果 URL 文件使用了认证保护，那就需要使用 `RUN wget`、`RUN curl` 或其他容器中包含的工具来完成 `ADD` 指令不支持的认证操作。

> **注意**：
> 如果`<src>`的内容变化，第一次遇到`ADD`指令会使Dockerfile文件中后续所有指令缓存失效。这包括 `RUN` 指令的缓存失效。详情参阅 [`Dockerfile`最佳实践指南](dockerfile_best-practices.md#build-cache) for more information.


`ADD` 遵从以下规则：

- `<src>`路径一定要在构建的*上下文环境*内；不可以`ADD ../something /something`，原因是`docker build`操作的第一步就是发送上下文环境目录（或子目录）给 docker daemon。

- 如果 `<src>` 是 URL，并且 `<dest>` 尾部不是以斜杠结束，那么会从该 URL 下载文件，并把它复制给 `<dest>`。

- 如果 `<src>` 是 URL，并且 `<dest>` 尾部是以斜杠结束，那么就会根据 URL 推断出文件名<filename>，然后该文件会下载给 `<dest>/<filename>`。例如 `ADD http://example.com/foobar /` 会创建文件 `/foobar`。URL 一定要有一个普通的路径，这样就可以发现适当的文件名 (如果是 `http://example.com` 就不会有效)。

- 如果 `<src>` 是个目录，目录的整个内容会被复制，包括文件系统的元数据。

> **注意**：
> 目录自身不会被复制，仅仅复制它的内容。

- 如果 `<src>` 是个*本地*可识别压缩格式（gzip、bzip2或xz）的 tar 文档，那么它会作为一个目录拆包解压。*远程* URL的 `<src>` **不会**解压。当目录被复制或拆包解压时，其操作方式与`tar -x`相同： the result is the union of:

    1. （？无论目标路径存在什么内容） Whatever existed at the destination path and
    2. The contents of the source tree, with conflicts resolved in favor
       of "2." on a file-by-file basis.

  > **注意**：
  > 判断文件是否是可识别压缩格式，仅仅基于文件本身的内容，而不是文件名称。例如，一个以`.tar.gz`结尾的空文件并不会被当作压缩文件，并且**不会**生成任何任何解压缩报错信息。相反，该文件只会被简单地复制到目标路径中。

- 如果 `<src>` 是任意其他类型的文件，就会按和它自身的元数据一起单独复制。在这种情况下，如果 `<dest>` 尾部是斜杠 `/` 结束，这会被当作目录，并且 `<src>` 的内容会写入到 `<dest>/base(<src>)`。

- 如果指定了多个 `<src>` 资源，或是直接指定或是使用了通配符，那么 `<dest>` 必须是目录并且以斜杠 `/` 结束。

- `<dest>`不是以斜杠结束，这会被当作一个正常文件，并且 `<src>` 的内容会写入到 `<dest>`中。

- 如果 `<dest>` 不存在，在它的路径内连同所有遗失的目录一并创建。

## COPY

有两种形式：

- `COPY <src>... <dest>`
- `COPY ["<src>",... "<dest>"]` (该形式要求路径中包含空格)

该指令复制`<src>`中的新文件或目录，并把他们添加到容器`<dest>`路径指定的文件系统中。

可以指定多个 `<src>` 资源，他们必须是要构建源目录(构建的上下文环境)内的相对路径。

每个 `<src>` 都可以包含通配符，并且用 Go 的 [filepath.Match](http://golang.org/pkg/path/filepath#Match) 规则进行匹配操作。如：

    COPY hom* /mydir/        # 添加所有以"hom"开始的文件
    COPY hom?.txt /mydir/    # ? 替换任意一个单一字符，如 "home.txt"

`<dest>` 是个绝对路径，或是相对于`WORKDIR`的路径，源文件要复制进去的地方在目标容器内部。

    COPY test relativeDir/   # 添加 "test" 到 `WORKDIR`/relativeDir/
    COPY test /absoluteDir/  # 添加 "test" 到 /absoluteDir/

所有创建新文件和目录的UID和GID是0。


> **注意**：
> 如果构建使用 STDIN (`docker build - < somefile`)，那么就不会构建的上下文环境，那么`COPY`也就不能使用了。

`COPY` 遵循以下规则：

- `<src>` 路径必须存在于构建的 *上下文环境* 中；不能进行 `COPY ../something /something` 操作，原因是 `docker build` 操作的第一步就是把上下文环境目录 (以及子目录) 发送给 docker daemon 。

- 如果 `<src>` 是目录，目录的整个内容会被复制，包括文件系统元数据。

> **注意**：
> 目录自身不会被复制，仅复制内容。

- 如果 `<src>` 是任何其他类型文件，它会和它的元数据一起单独复制。在此情况下，如果 `<dest>` 以斜杠 `/` 结尾，它会当作目录，并且 `<src>` 会写入到 `<dest>/base(<src>)`。

- 如果指定了多个 `<src>` 资源，或是直接或是使用通配符，那么 `<dest>` 必须是目录，必须以斜杠 `/` 结尾。

- 如果 `<dest>` 不是以斜杠结尾，这会被当作普通文件，`<src>` 的内容会写入 `<dest>` 中。

- 如果 `<dest>` 不存在，在它的路径内连同所有遗失的目录会一并创建。

## ENTRYPOINT

ENTRYPOINT 有两种形式：

- `ENTRYPOINT ["executable", "param1", "param2"]`    (*exec*形式，首选)
- `ENTRYPOINT command param1 param2`    (*shell*形式)

`ENTRYPOINT`可以配置将运行的容器，指定容器启动时执行的命令。

例如，下面的操作会让 nginx 随它的默认内容启动，并且侦听 80 端口：

    docker run -i -t --rm -p 80:80 nginx

`docker run <image>` 的命令行参数出现在*exec*形式`ENTRYPOINT`的全部元素后面，并且这些元素会覆盖`CMD`指定的所有元素。它允许传参给进入点（entry point），如 `docker run <image> -d` 会传`-d`参数给进入点。使用`docker run --entrypoint`标志参数可以覆盖`ENTRYPOINT`指令。

*shell*形式会禁止使用任何`CMD`或`run`命令行参数，这带来的弊端就是`ENTRYPOINT`会作为`/bin/sh -c`的子命令启动，它不传递信号(signal)。这意味着该执行程序不会是容器的 `PID 1` - 并且 _不_ 会接收Unix信号，这会使该执行程序不会接收到来自`docker stop <container>`发送的`SIGTERM`信号。

`Dockerfile`文件中，只有最后一条 `ENTRYPOINT` 指令会起作用。

### Exec形式的ENTRYPOINT示例

可以使用`ENTRYPOINT`的*exec*形式设置相当稳定的默认命令和参数，然后使用`CMD`的一种形式设置其他变化频率较高的默认值。

    FROM ubuntu
    ENTRYPOINT ["top", "-b"]
    CMD ["-c"]

运行这个容器时，可以看到 `top` 命令是唯一进程：

    $ docker run -it --rm --name test  top -H
    top - 08:25:00 up  7:27,  0 users,  load average: 0.00, 0.01, 0.05
    Threads:   1 total,   1 running,   0 sleeping,   0 stopped,   0 zombie
    %Cpu(s):  0.1 us,  0.1 sy,  0.0 ni, 99.7 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
    KiB Mem:   2056668 total,  1616832 used,   439836 free,    99352 buffers
    KiB Swap:  1441840 total,        0 used,  1441840 free.  1324440 cached Mem

      PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND
        1 root      20   0   19744   2336   2080 R  0.0  0.1   0:00.04 top

想要深入检测，可以使用 `docker exec`：

    $ docker exec -it test ps aux
    USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
    root         1  2.6  0.1  19752  2352 ?        Ss+  08:24   0:00 top -b -H
    root         7  0.0  0.1  15572  2164 ?        R+   08:25   0:00 ps aux

可以优雅地使用`docker stop test` 请求 `top` 关闭。

下面的 `Dockerfile` 文件显示，使用`ENTRYPOINT`在前台运行 Apache：

```
FROM debian:stable
RUN apt-get update && apt-get install -y --force-yes apache2
EXPOSE 80 443
VOLUME ["/var/www", "/var/log/apache2", "/etc/apache2"]
ENTRYPOINT ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
```

如果要给一个执行程序写一个启动脚本，就要确保这个最终执行程序接收到由`exec`和`gosu`命令发送的Unix信号：

```bash
#!/bin/bash
set -e

if [ "$1" = 'postgres' ]; then
    chown -R postgres "$PGDATA"

    if [ -z "$(ls -A "$PGDATA")" ]; then
        gosu postgres initdb
    fi

    exec gosu postgres "$@"
fi

exec "$@"
```

最后，在shutdown时还要做一下额外清理工作（或与其他容器通信工作），或是协调多个可执行程序，确保 `ENTRYPOINT` 脚本接收到Unix信号，传递信号，然后做得更多：

```
#!/bin/sh
# Note: I've written this using sh so it works in the busybox container too

# USE the trap if you need to also do manual cleanup after the service is stopped,
#     or need to start multiple services in the one container
trap "echo TRAPed signal" HUP INT QUIT KILL TERM

# start service in background here
/usr/sbin/apachectl start

echo "[hit enter key to exit] or run 'docker stop <container>'"
read

# stop service and clean up here
echo "stopping apache"
/usr/sbin/apachectl stop

echo "exited $0"
```

If you run this image with `docker run -it --rm -p 80:80 --name test apache`,
you can then examine the container's processes with `docker exec`, or `docker top`,
and then ask the script to stop Apache:

```bash
$ docker exec -it test ps aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.1  0.0   4448   692 ?        Ss+  00:42   0:00 /bin/sh /run.sh 123 cmd cmd2
root        19  0.0  0.2  71304  4440 ?        Ss   00:42   0:00 /usr/sbin/apache2 -k start
www-data    20  0.2  0.2 360468  6004 ?        Sl   00:42   0:00 /usr/sbin/apache2 -k start
www-data    21  0.2  0.2 360468  6000 ?        Sl   00:42   0:00 /usr/sbin/apache2 -k start
root        81  0.0  0.1  15572  2140 ?        R+   00:44   0:00 ps aux
$ docker top test
PID                 USER                COMMAND
10035               root                {run.sh} /bin/sh /run.sh 123 cmd cmd2
10054               root                /usr/sbin/apache2 -k start
10055               33                  /usr/sbin/apache2 -k start
10056               33                  /usr/sbin/apache2 -k start
$ /usr/bin/time docker stop test
test
real	0m 0.27s
user	0m 0.03s
sys	0m 0.03s
```

> **注意**： 使用`--entrypoint`可以覆盖 `ENTRYPOINT` 设置，
> but this can only set the binary to *exec* (no `sh -c` will be used).

> **注意**：
> *exec*形式会被解析成JSON数组，也就是说必须使用双引号(")而非引号(')括起文字内容。

> **注意**：
> 不同于*shell*形式，*exec*形式不调用命令shell。也就是说，常规的shell操作不会起作用。如 `ENTRYPOINT [ "echo", "$HOME" ]` 不会对`$HOME`产生变量替换。要想使用shell操作，要么使用*shell*形式，要么直接执行一个shell，如 `ENTRYPOINT [ "sh", "-c", "echo", "$HOME" ]`。`Dockerfile`文件中使用`ENV`定义的变量会被`Dockerfile`解析器替换。

### Shell形式的ENTRYPOINT示例

可以在`ENTRYPOINT`中指定一个纯字符串，它会在`/bin/sh -c`中执行。该形式使用shell操作来替换shell环境变量，而且会忽视任何 `CMD` 或 `docker run` 命令行的参数。要确保`docker stop`可以正确发送信号给任何持久运行的`ENTRYPOINT`可执行程序，就要牢记以 `exec` 启动它：

    FROM ubuntu
    ENTRYPOINT exec top -b

运行该镜像可以看到一个单一 `PID 1` 进程：

    $ docker run -it --rm --name test top
    Mem: 1704520K used, 352148K free, 0K shrd, 0K buff, 140368121167873K cached
    CPU:   5% usr   0% sys   0% nic  94% idle   0% io   0% irq   0% sirq
    Load average: 0.08 0.03 0.05 2/98 6
      PID  PPID USER     STAT   VSZ %VSZ %CPU COMMAND
        1     0 root     R     3164   0%   0% top -b

执行`docker stop`，可以清洁环保地退出：

    $ /usr/bin/time docker stop test
    test
    real	0m 0.20s
    user	0m 0.02s
    sys	0m 0.04s

如果没有在`ENTRYPOINT`开始处添加`exec`：

    FROM ubuntu
    ENTRYPOINT top -b
    CMD --ignored-param1

运行它 (为下一步操作给它命名)：

    $ docker run -it --name test top --ignored-param2
    Mem: 1704184K used, 352484K free, 0K shrd, 0K buff, 140621524238337K cached
    CPU:   9% usr   2% sys   0% nic  88% idle   0% io   0% irq   0% sirq
    Load average: 0.01 0.02 0.05 2/101 7
      PID  PPID USER     STAT   VSZ %VSZ %CPU COMMAND
        1     0 root     S     3168   0%   0% /bin/sh -c top -b cmd cmd2
        7     1 root     R     3164   0%   0% top -b

可以看到`ENTRYPOINT`指定的程序 `top` 输出，它不是`PID 1`。

运行 `docker stop test`，容器就不会清洁环保地退出了 - 在一段延时后，`stop`命令会强制发送了一个 `SIGKILL` 信号：

    $ docker exec -it test ps aux
    PID   USER     COMMAND
        1 root     /bin/sh -c top -b cmd cmd2
        7 root     top -b
        8 root     ps aux
    $ /usr/bin/time docker stop test
    test
    real	0m 10.19s
    user	0m 0.04s
    sys	0m 0.03s

## VOLUME

    VOLUME ["/data"]

`VOLUME`指令创建了一个指定名称的挂载点，标记它为来自本地主机或其他容器的外部挂载卷。用于授权访问从容器内到主机上的目录。它的值可以是JSON数组（`VOLUME ["/var/log/"]`），或是带多个参数的纯字符串（`VOLUME /var/log` 或 `VOLUME /var/log
/var/db`）。更多信息、示例和挂载指令参考 [*用卷共享目录*](dockervolumes.md#挂载一个主机目录作数据卷)

`docker run`命令会用基础镜像中指定位置已经存在的数据对新创建的卷进行初始化。例如下面的 Dockerfile 片断：

    FROM ubuntu
    RUN mkdir /myvol
    RUN echo "hello world" > /myvol/greeting
    VOLUME /myvol

这个 Dockerfile 文件产生了一个镜像，它在新创建了一个`/myvol`挂载点，并且把 `greeting` 文件复制进新创建的卷中。

```
FROM base
VOLUME ["/tmp/data"]
```
运行通过该Dockerfile生成image的容器，/tmp/data目录中的数据在容器关闭后，里面的数据还存在。例如另一个容器也有持久化数据的需求，且想使用上面容器共享的 /tmp/data 目录，那么可以运行下面的命令启动一个容器：
```
docker run -t -i -rm -volumes-from container1 image2 bash
```
container1为第一个容器的ID，image2为第二个容器运行image的名字。

> **注意**：
> 如果卷已经被声明，则该卷内任何构建操作所带来的数据变化都会被丢弃。

> **注意**：
> 该列表被解析成JSON数组，也就是说必须使用双引号(")而非引号(')把文字内容括起来。

## USER

    USER daemon

`USER` 指令用于设定运行镜像时要使用的用户名或 UID，它也用于`Dockerfile`文件中任何 `RUN`、`CMD`以及`ENTRYPOINT`指令。比如指定 memcached 的运行用户，可以使用 `ENTRYPOINT` 来实现:
```
ENTRYPOINT ["memcached", "-u", "daemon"]
```

更好的方式是：
```
ENTRYPOINT ["memcached"]
USER daemon
```

## WORKDIR

    WORKDIR /path/to/workdir

`WORKDIR`指令用于设置`Dockerfile`文件中任何`RUN`、`CMD`、`ENTRYPOINT`、`COPY`以及`ADD`指令执行的工作目录。

一个`Dockerfile`文件可以使用它多次。如果提供的是相对路径，这会相对于前一个`WORKDIR`的路径，如：

    WORKDIR /a
    WORKDIR b
    WORKDIR c
    RUN pwd

最后 `pwd` 命令输出结果会是 `/a/b/c`。

`WORKDIR`指令可以解析前面使用`ENV`设置的环境变量。可以显式地使用`Dockerfile`中设置的环境变量。例如：

    ENV DIRPATH /path
    WORKDIR $DIRPATH/$DIRNAME
    RUN pwd

`pwd`命令的最终输出结果是`/path/$DIRNAME`

## ARG

    ARG <name>[=<default value>]

`ARG`指令定义了一个变量，用户在构建时可以把它传给使用`docker build`命令参数是`--build-arg <varname>=<value>`的构建者。如果用指定了一个没有在Dockerfile中定义的构建参数，则构建会输出错误信息。

```
One or more build-args were not consumed, failing build.
```

Dockerfile 的作者信息可以通过指定`ARG`一次定义一个变量，也可以指定多次定义多个变量。例如：

```
FROM busybox
ARG user1
ARG buildno
...
```

Dockerfile的作者信息可以选择用一个 `ARG` 指令指定一个默认值：

```
FROM busybox
ARG user1=someuser
ARG buildno=1
...
```

如果 `ARG` 值已经有默认的，并且在构建时没有值传递，那么构建器使用默认值。

一个`ARG`变量定义生效是从`Dockerfile`中被定义那一行开始的，并不是从命令行或其他地方使用这个参数开始。例如，思考以下Dockerfile文件：

```
1 FROM busybox
2 USER ${user:-some_user}
3 ARG user
4 USER $user
...
```
用户通过回调来构建文件：

```
$ docker build --build-arg user=what_user Dockerfile
```

第二行的 `USER` 等于 `some_user`，而`user`变量在第三行被定义。第四行`USER`等于`what_user`，而`user`已经被定义，并且`what_user`值也通过命令行传递了。如果先于`ARG`指令的定义，任何变量使用的结果值都是空字串。

> **注意**： 不建议构建时（build-time）变量用于传递象github密钥、用户证书等的数据。

可以使用 `ARG` 或 `ENV` 指令指明 `RUN` 指令可用的变量。`ENV`指令定义的环境变量总是覆盖`ARG`指令定义的同名变量。示例：

```
1 FROM ubuntu
2 ARG CONT_IMG_VER
3 ENV CONT_IMG_VER v1.0.0
4 RUN echo $CONT_IMG_VER
```
假定镜像是由以下命令构建：

```
$ docker build --build-arg CONT_IMG_VER=v2.0.1 Dockerfile
```

`RUN`指令使用的值是`v1.0.0`而不是由用户传递的`ARG`设置值`v2.0.1`。与此可作类比的是shell脚本中局部域变量，从它定义位置开始，它会覆盖作为传参或继承自环境的变量。

继续上面的示例，不过对`ENV`说明做些修改：

```
1 FROM ubuntu
2 ARG CONT_IMG_VER
3 ENV CONT_IMG_VER ${CONT_IMG_VER:-v1.0.0}
4 RUN echo $CONT_IMG_VER
```

不同于`ARG`指令，`ENV`值一直存在于构建镜像过程中。思考 Docker 构建不带`--build-arg`标志参数：

```
$ docker build Dockerfile
```

本例中，`CONT_IMG_VER`一直存在于镜像中，它的值是`v1.0.0`，该值是第三行的`ENV`指令定义的默认值。

本例中的变量扩展技巧让你可以从命令行传参，也可以通过`ENV`指令让最终镜像中变量进行持久化。变量扩展仅支持 [有限的
Dockerfile 指令集合](#environment-replacement)

Docker 有一个预定义好的`ARG`变量集合，在Dockerfile文件中不需要`ARG`指定定义直接使用：

* `HTTP_PROXY`
* `http_proxy`
* `HTTPS_PROXY`
* `https_proxy`
* `FTP_PROXY`
* `ftp_proxy`
* `NO_PROXY`
* `no_proxy`

命令行中使用 `--build-arg <varname>=<value>`标志参数即可会参使用。

### 影响构建缓存

`ARG`变量并不象`ENV`变量持久化到构建的镜像中。不过 `ARG` 变量可以类似方式对构建缓存加以影响。如果 Dockerfile 文件定义了一个 `ARG` 变量，该变量值与以前的构建不相同，这时就会在第一次使用（不是声明）时生一个 “cache miss”。例如：

```
1 FROM ubuntu
2 ARG CONT_IMG_VER
3 RUN echo $CONT_IMG_VER
```

如果在命令行中指定 `--build-arg CONT_IMG_VER=<value>`，第二行的语句并不会产生“cache miss”；第三行会生成“cache
miss”。第二行的定义并不会对结果镜像有丝毫影响。第三行`RUN`执行了一个命令，会定义一系列的环境变量，包括 `CONT_IMG_VER`。此时，`ARG`变量对结果镜像产生了影响，cache miss发生了。

同一命令行下，思考另一个例子：

```
1 FROM ubuntu
2 ARG CONT_IMG_VER
3 ENV CONT_IMG_VER $CONT_IMG_VER
4 RUN echo $CONT_IMG_VER
```
本例中，cache miss 发生在第三行。原因是`ENV`变量的值引用了`ARG`的变量，该值因命令行传参导致了变化。本例中，`ENV`命令会让镜像包含这个值。

## ONBUILD

    ONBUILD [INSTRUCTION]

`ONBUILD`指令给镜像添加了一个会在以后执行的*触发器*指令，当该镜像被当作另一个构建的基础镜像时会触发。该触发器会在下游构建（downstream build）的上下文环境中执行，就好像它已经插入到下游`Dockerfile`文件中`FROM`指令的后面了。

任何构建指定都可以注册成触发器。

如果你构建一个镜像，而它可能会被作为基础镜像而构建其他镜像，那么这个指令会很有用。例如，针对特定用户配置而定制的应用构建环境或守护进程。

For example, if your image is a reusable Python application builder, it
will require application source code to be added in a particular
directory, and it might require a build script to be called *after*
that. You can't just call `ADD` and `RUN` now, because you don't yet
have access to the application source code, and it will be different for
each application build. You could simply provide application developers
with a boilerplate `Dockerfile` to copy-paste into their application, but
that is inefficient, error-prone and difficult to update because it
mixes with application-specific code.

解决办法就是使用`ONBUILD`注册预备以后运行的指令，待下次构建场景。

以下是运行过程：

1. 碰到 `ONBUILD` 指令时，构建器会添加一个触发器到构建镜像的元数据中。指令不会影响当前构建。
2. 构建结束，所有触发器列表会存储于镜像清单（manifest）中，在键名`OnBuild`下。他们可以用`docker inspect`命令检查到。
3. 以后，该镜像可能通过`FROM`指令被用于新构建的基础镜像。在进行 `FROM` 指令处理时，下游构建器先寻找`ONBUILD`触发器，以它们注册时的顺序执行它们。如果有任何触发器运行失败，`FROM`指令终止执行引起构建失败。如果所有触发器都运行成功，`FROM`指令会完成，构建则继续。
4. 在最终镜像执行后，触发器会清除。也就是说，他们不会被隔代构建"grand-children"所继承。

示例，给构建加些东西：

    [...]
    ONBUILD ADD . /app/src
    ONBUILD RUN /usr/local/bin/python-build --dir /app/src
    [...]

> **警告**： 禁止使用 `ONBUILD ONBUILD`这样的链式`ONBUILD`指令。

> **警告**： `ONBUILD`指令不能触发`FROM`或`MAINTAINER`指令。

## STOPSIGNAL

	STOPSIGNAL signal

`STOPSIGNAL`指令设置系统呼叫信号，它会发给容器退出。该信号是合法的 unsigned 数值，与内核中syscall表中位置对应，例如信号值 9，或者在SIGNAME格式中的信号名，如 SIGKILL。

## Dockerfile实例

下面看看 Dockerfile 语法的实例。如果对更实际的例子感兴趣，可以看看这个列表 [Dockerization examples](https://github.com/docker/docker/blob/master/docs/examples/index.md).

```
# Nginx
#
# VERSION               0.0.1

FROM      ubuntu
MAINTAINER Victor Vieux <victor@docker.com>

LABEL Description="This image is used to start the foobar executable" Vendor="ACME Products" Version="1.0"
RUN apt-get update && apt-get install -y inotify-tools nginx apache2 openssh-server
```

```
# Firefox over VNC
#
# VERSION               0.3

FROM ubuntu

# Install vnc, xvfb in order to create a 'fake' display and firefox
RUN apt-get update && apt-get install -y x11vnc xvfb firefox
RUN mkdir ~/.vnc
# Setup a password
RUN x11vnc -storepasswd 1234 ~/.vnc/passwd
# Autostart firefox (might not be the best way, but it does the trick)
RUN bash -c 'echo "firefox" >> /.bashrc'

EXPOSE 5900
CMD    ["x11vnc", "-forever", "-usepw", "-create"]
```

```
# Multiple images example
#
# VERSION               0.1

FROM ubuntu
RUN echo foo > bar
# Will output something like ===> 907ad6c2736f

FROM ubuntu
RUN echo moo > oink
# Will output something like ===> 695d7793cbe4

# You᾿ll now have two images, 907ad6c2736f with /bar, and 695d7793cbe4 with
# /oink.
```