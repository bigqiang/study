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

[`docker build`](https://github.com/docker/docker/blob/master/docs/reference/commandline/build.md) 命令根据`Dockerfile`及 *上下文* 来构建一个镜像。构建的上下文是指该文件所在的特定位置`PATH`或`URL`。`PATH`是指本地文件系统的目录。`URL`是指 Git 版本库的位置。

上下文可以被递归处理。所以，`PATH`包含任意子目录，并且`URL`包含了版本库及其子模块。使用当前目录的上下文可以简化命令：

    $ docker build .
    Sending build context to Docker daemon  6.51 MB
    ...

该构建是由 Docker daemon 运行的，而非 CLI。构建进程做的头一件事就是把整个上下文（递归）发送给daemon。多数情况下，最好是在一个空目录中做上下文启动，把
Dockerfile 文件放在该目录中。仅添加构建该 Dockerfile 文件所必须的文件在里面。

>**警告**: 不要用目录 `/` 作 `PATH` ，它可能让构建传送硬盘所有内容给 Docker daemon。

为了使用在构建上下文中的某个文件，`Dockerfile` 文件中的指令要具体地指定该文件。例如 `COPY` 指令。为提升构建性能，在上下文目录中可以添加一个 `.dockerignore` 文件以排除不要的文件和目录 。 详情查看本文档的如何创建一个[`.dockerignore`文件](#dockerignore文件)。

习惯上，`Dockerfile`被称作`Dockerfile`，并且位于上下文的根路径中。`docker build`用`-f`标志参数指向文件系统中任意位置的 Dockerfile 文件。

    $ docker build -f /path/to/a/Dockerfile .

如果构建成功，可以指定一个版本库并标记存储这个新镜像：

    $ docker build -t shykes/myapp .

构建完成后，如果想标记该镜像到多个版本库中，可以在运行`build`命令时添加多个 `-t` 参数：

    $ docker build -t shykes/myapp:1.0.2 -t shykes/myapp:latest .

在最终生成新镜像的ID前，Docker daemon 是按顺序依次执行 `Dockerfile` 文件中指令的，同时在必要的情况下，会把每条指令的执行结果提交到新镜像中。Docker daemon 会自动清理干净发送过来的上下文描述信息。

注意，每条指令都是独立运行的，都会产生一个新镜像，所以`RUN cd /tmp`不会对下一条指令有任何影响。

只要有可能，Docker就会重用中间镜像(cache)，这可以极大加速`docker build`进程。可通过控制台中的`Using cache`信息显示这一点。
(详情参阅 [Build cache section](https://github.com/docker/docker/blob/master/docs/userguide/eng-image/dockerfile_best-practices.md#build-cache)) `Dockerfile`文件最佳实践指南：

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

docker CLI 给 docker daemon 发送上下文前，先在上下文的根目录中寻找名为`.dockerignore`的文件。如果存在，CLI 会修改上下文描述，把匹配该文件中描述的文件和目录排除出去。可避免不必要地发送超大或敏感内容的文件和目录给 daemon，也可避免使用`ADD`或`COPY`把他们添加到镜像中的可能性。

CLI 会把`.dockerignore`文件解析成一个独立行的模式列表，模式类似于 Unix shell 的 file globs。为了模式匹配，上下文的根目录会被看作即是工作目录也是根目录。例如模式 `/foo/bar` 和 `foo/bar`表示，`PATH`下(或是位于`URL`的的git版本库的根目录下)`foo`子目录中名为`bar`的文件或目录被排除。

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

除了 Go 的 filepath.Match 规则，Docker也支持特殊通配符字串`**`，它可匹配任意数量(含0)的目录。如 `**/*.go` 会排除所有目录中（含构建上下文中的根目录）所有以 `.go` 结尾的文件。

以`!`起始的行用于设定排除规则的例外项。如下示例：

```
    *.md
    !README.md
```

*除了* `README.md`文件外，排除上下文中所有markdown文件。

`!`例外规则的位置造成的影响：匹配特殊文件的`.dockerignore`中最后一行会决定该文是包括还是排除。思考以下例子：

```
    *.md
    !README*.md
    README-secret.md
```

在上下文中除了 README 文件（不`README-secret.md`文件），排除所有 markdown 文件。

现在考虑下面例子：

```
    *.md
    README-secret.md
    !README*.md
```

包括所有 README 文件，中间行无效，因为`!README*.md`匹配`README-secret.md`规则，而且在最后一行。

即使用 `.dockerignore` 文件排除 `Dockerfile` 和 `.dockerignore` 文件，这些文件仍然会发送给 daemon，因为这是完成任务必须文件。不过 `ADD` 和 `COPY` 命令并不会把他们复制到镜像中。

最后，还需要指定哪些文件要包含而不是排除在上下文中。为达成目标，要指定 `*` 作第一行模式，后跟一条或多条`!`例外规则模式 。

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

该指令执行当前镜像顶部新层中的任意命令并提交（commit）结果。提交结果的镜像会用于`Dockerfile`文件中下一步指令。

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
guide](https://github.com/docker/docker/blob/master/docs/userguide/eng-image/dockerfile_best-practices.md#build-cache)。

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

> **注意**：
> 环境持久性会导致不可预料的副效应。例如，设置 `ENV DEBIAN_FRONTEND noninteractive` 可能使基于Debian镜像的apt-get
用户迷惑。要为一条命令只设置一个值，可以使用 `RUN <key>=<value> <command>`。

## ADD

它有两种形式：

- `ADD <src>... <dest>`
- `ADD ["<src>",... "<dest>"]` (对于包含空格的路径是必选形式)

该指令会复制从`<scr>`指定的新文件、目录或远程文件URL，把它们添加到`<dest>`路径指定的容器文件系统中。

可以指定多个`<src>`源，如果他们是文件或目录，那么他们必须与要构建源文件夹相关联(构建的上下文)。

每一个 `<src>` 都可以包含通配符，并且可用 Go 的 [filepath.Match](http://golang.org/pkg/path/filepath#Match) 规则进行匹配通过。例如：

    ADD hom* /mydir/        # 添加所有以"hom"开始的文件
    ADD hom?.txt /mydir/    # ? 可被任意一个单字符替换，如"home.txt"

`<dest>`要写成绝对路径，或是写成相对于`WORKDIR`的路径，源会被复制到目标容器内`<dest>`指定的路径下。

    ADD test relativeDir/          # 复制到 `WORKDIR`/relativeDir/
    ADD test /absoluteDir/         # 复制"test" 到 /absoluteDir/

所有新文件和目录在创建时会带一个值为0的 UID 和 GID。

如果 `<src>` 是一个远程文件 URL，目标拥有 600 许可权限。如果获取的远程文件有一个HTTP `Last-Modified` header，则该来自该header的时间戳会设置目标文件的`mtime`。不过，象任何其他在`ADD`操作期间处理的文件一样，`mtime` 不会包含在目标文件中，不论该文件改变还是缓存该更新。

> **注意**：
> 如果是通过把`Dockerfile`传递给 STDIN (`docker build - < somefile`)来构建，那么就没构建的上下文。因此 `Dockerfile` 仅能包含一个基于`ADD`指令的URL。也可以传递一个压缩文档给 STDIN: (`docker build - < archive.tar.gz`)，文档根目录中的 `Dockerfile`和其他文档可以用在构建的上下文中。

> **注意**：
> 如果 URL 文件使用了认证保护，那就需要使用 `RUN wget`、`RUN curl` 或其他容器中包含的工具来完成 `ADD` 指令不支持的认证操作。

> **注意**：
> 如果`<src>`的内容变化，第一次遇到`ADD`指令会使Dockerfile文件中后续所有指令缓存失效。这包括 `RUN` 指令的缓存失效。详情参阅 [`Dockerfile`最佳实践指南](https://github.com/docker/docker/blob/master/docs/userguide/eng-image/dockerfile_best-practices.md#build-cache) for more information.


`ADD` obeys the following rules:

- The `<src>` path must be inside the *context* of the build;
  you cannot `ADD ../something /something`, because the first step of a
  `docker build` is to send the context directory (and subdirectories) to the
  docker daemon.

- If `<src>` is a URL and `<dest>` does not end with a trailing slash, then a
  file is downloaded from the URL and copied to `<dest>`.

- If `<src>` is a URL and `<dest>` does end with a trailing slash, then the
  filename is inferred from the URL and the file is downloaded to
  `<dest>/<filename>`. For instance, `ADD http://example.com/foobar /` would
  create the file `/foobar`. The URL must have a nontrivial path so that an
  appropriate filename can be discovered in this case (`http://example.com`
  will not work).

- If `<src>` is a directory, the entire contents of the directory are copied,
  including filesystem metadata.

> **注意**：
> The directory itself is not copied, just its contents.

- If `<src>` is a *local* tar archive in a recognized compression format
  (identity, gzip, bzip2 or xz) then it is unpacked as a directory. Resources
  from *remote* URLs are **not** decompressed. When a directory is copied or
  unpacked, it has the same behavior as `tar -x`: the result is the union of:

    1. Whatever existed at the destination path and
    2. The contents of the source tree, with conflicts resolved in favor
       of "2." on a file-by-file basis.

  > **注意**：
  > Whether a file is identified as a recognized compression format or not
  > is done solely based on the contents of the file, not the name of the file.
  > For example, if an empty file happens to end with `.tar.gz` this will not
  > be recognized as a compressed file and **will not** generate any kind of
  > decompression error message, rather the file will simply be copied to the
  > destination.

- If `<src>` is any other kind of file, it is copied individually along with
  its metadata. In this case, if `<dest>` ends with a trailing slash `/`, it
  will be considered a directory and the contents of `<src>` will be written
  at `<dest>/base(<src>)`.

- If multiple `<src>` resources are specified, either directly or due to the
  use of a wildcard, then `<dest>` must be a directory, and it must end with
  a slash `/`.

- If `<dest>` does not end with a trailing slash, it will be considered a
  regular file and the contents of `<src>` will be written at `<dest>`.

- If `<dest>` doesn't exist, it is created along with all missing directories
  in its path.

## COPY

COPY has two forms:

- `COPY <src>... <dest>`
- `COPY ["<src>",... "<dest>"]` (this form is required for paths containing
whitespace)

The `COPY` instruction copies new files or directories from `<src>`
and adds them to the filesystem of the container at the path `<dest>`.

Multiple `<src>` resource may be specified but they must be relative
to the source directory that is being built (the context of the build).

Each `<src>` may contain wildcards and matching will be done using Go's
[filepath.Match](http://golang.org/pkg/path/filepath#Match) rules. For example:

    COPY hom* /mydir/        # adds all files starting with "hom"
    COPY hom?.txt /mydir/    # ? is replaced with any single character, e.g., "home.txt"

The `<dest>` is an absolute path, or a path relative to `WORKDIR`, into which
the source will be copied inside the destination container.

    COPY test relativeDir/   # adds "test" to `WORKDIR`/relativeDir/
    COPY test /absoluteDir/  # adds "test" to /absoluteDir/

All new files and directories are created with a UID and GID of 0.

> **注意**：
> If you build using STDIN (`docker build - < somefile`), there is no
> build context, so `COPY` can't be used.

`COPY` obeys the following rules:

- The `<src>` path must be inside the *context* of the build;
  you cannot `COPY ../something /something`, because the first step of a
  `docker build` is to send the context directory (and subdirectories) to the
  docker daemon.

- If `<src>` is a directory, the entire contents of the directory are copied,
  including filesystem metadata.

> **注意**：
> The directory itself is not copied, just its contents.

- If `<src>` is any other kind of file, it is copied individually along with
  its metadata. In this case, if `<dest>` ends with a trailing slash `/`, it
  will be considered a directory and the contents of `<src>` will be written
  at `<dest>/base(<src>)`.

- If multiple `<src>` resources are specified, either directly or due to the
  use of a wildcard, then `<dest>` must be a directory, and it must end with
  a slash `/`.

- If `<dest>` does not end with a trailing slash, it will be considered a
  regular file and the contents of `<src>` will be written at `<dest>`.

- If `<dest>` doesn't exist, it is created along with all missing directories
  in its path.

## ENTRYPOINT

ENTRYPOINT has two forms:

- `ENTRYPOINT ["executable", "param1", "param2"]`
  (*exec* form, preferred)
- `ENTRYPOINT command param1 param2`
  (*shell* form)

An `ENTRYPOINT` allows you to configure a container that will run as an executable.

For example, the following will start nginx with its default content, listening
on port 80:

    docker run -i -t --rm -p 80:80 nginx

Command line arguments to `docker run <image>` will be appended after all
elements in an *exec* form `ENTRYPOINT`, and will override all elements specified
using `CMD`.
This allows arguments to be passed to the entry point, i.e., `docker run <image> -d`
will pass the `-d` argument to the entry point.
You can override the `ENTRYPOINT` instruction using the `docker run --entrypoint`
flag.

The *shell* form prevents any `CMD` or `run` command line arguments from being
used, but has the disadvantage that your `ENTRYPOINT` will be started as a
subcommand of `/bin/sh -c`, which does not pass signals.
This means that the executable will not be the container's `PID 1` - and
will _not_ receive Unix signals - so your executable will not receive a
`SIGTERM` from `docker stop <container>`.

Only the last `ENTRYPOINT` instruction in the `Dockerfile` will have an effect.

### Exec form ENTRYPOINT example

You can use the *exec* form of `ENTRYPOINT` to set fairly stable default commands
and arguments and then use either form of `CMD` to set additional defaults that
are more likely to be changed.

    FROM ubuntu
    ENTRYPOINT ["top", "-b"]
    CMD ["-c"]

When you run the container, you can see that `top` is the only process:

    $ docker run -it --rm --name test  top -H
    top - 08:25:00 up  7:27,  0 users,  load average: 0.00, 0.01, 0.05
    Threads:   1 total,   1 running,   0 sleeping,   0 stopped,   0 zombie
    %Cpu(s):  0.1 us,  0.1 sy,  0.0 ni, 99.7 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
    KiB Mem:   2056668 total,  1616832 used,   439836 free,    99352 buffers
    KiB Swap:  1441840 total,        0 used,  1441840 free.  1324440 cached Mem

      PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND
        1 root      20   0   19744   2336   2080 R  0.0  0.1   0:00.04 top

To examine the result further, you can use `docker exec`:

    $ docker exec -it test ps aux
    USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
    root         1  2.6  0.1  19752  2352 ?        Ss+  08:24   0:00 top -b -H
    root         7  0.0  0.1  15572  2164 ?        R+   08:25   0:00 ps aux

And you can gracefully request `top` to shut down using `docker stop test`.

The following `Dockerfile` shows using the `ENTRYPOINT` to run Apache in the
foreground (i.e., as `PID 1`):

```
FROM debian:stable
RUN apt-get update && apt-get install -y --force-yes apache2
EXPOSE 80 443
VOLUME ["/var/www", "/var/log/apache2", "/etc/apache2"]
ENTRYPOINT ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
```

If you need to write a starter script for a single executable, you can ensure that
the final executable receives the Unix signals by using `exec` and `gosu`
commands:

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

Lastly, if you need to do some extra cleanup (or communicate with other containers)
on shutdown, or are co-ordinating more than one executable, you may need to ensure
that the `ENTRYPOINT` script receives the Unix signals, passes them on, and then
does some more work:

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

> **Note:** you can over ride the `ENTRYPOINT` setting using `--entrypoint`,
> but this can only set the binary to *exec* (no `sh -c` will be used).

> **注意**：
> The *exec* form is parsed as a JSON array, which means that
> you must use double-quotes (") around words not single-quotes (').

> **注意**：
> Unlike the *shell* form, the *exec* form does not invoke a command shell.
> This means that normal shell processing does not happen. For example,
> `ENTRYPOINT [ "echo", "$HOME" ]` will not do variable substitution on `$HOME`.
> If you want shell processing then either use the *shell* form or execute
> a shell directly, for example: `ENTRYPOINT [ "sh", "-c", "echo", "$HOME" ]`.
> Variables that are defined in the `Dockerfile`using `ENV`, will be substituted by
> the `Dockerfile` parser.

### Shell form ENTRYPOINT example

You can specify a plain string for the `ENTRYPOINT` and it will execute in `/bin/sh -c`.
This form will use shell processing to substitute shell environment variables,
and will ignore any `CMD` or `docker run` command line arguments.
To ensure that `docker stop` will signal any long running `ENTRYPOINT` executable
correctly, you need to remember to start it with `exec`:

    FROM ubuntu
    ENTRYPOINT exec top -b

When you run this image, you'll see the single `PID 1` process:

    $ docker run -it --rm --name test top
    Mem: 1704520K used, 352148K free, 0K shrd, 0K buff, 140368121167873K cached
    CPU:   5% usr   0% sys   0% nic  94% idle   0% io   0% irq   0% sirq
    Load average: 0.08 0.03 0.05 2/98 6
      PID  PPID USER     STAT   VSZ %VSZ %CPU COMMAND
        1     0 root     R     3164   0%   0% top -b

Which will exit cleanly on `docker stop`:

    $ /usr/bin/time docker stop test
    test
    real	0m 0.20s
    user	0m 0.02s
    sys	0m 0.04s

If you forget to add `exec` to the beginning of your `ENTRYPOINT`:

    FROM ubuntu
    ENTRYPOINT top -b
    CMD --ignored-param1

You can then run it (giving it a name for the next step):

    $ docker run -it --name test top --ignored-param2
    Mem: 1704184K used, 352484K free, 0K shrd, 0K buff, 140621524238337K cached
    CPU:   9% usr   2% sys   0% nic  88% idle   0% io   0% irq   0% sirq
    Load average: 0.01 0.02 0.05 2/101 7
      PID  PPID USER     STAT   VSZ %VSZ %CPU COMMAND
        1     0 root     S     3168   0%   0% /bin/sh -c top -b cmd cmd2
        7     1 root     R     3164   0%   0% top -b

You can see from the output of `top` that the specified `ENTRYPOINT` is not `PID 1`.

If you then run `docker stop test`, the container will not exit cleanly - the
`stop` command will be forced to send a `SIGKILL` after the timeout:

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

The `VOLUME` instruction creates a mount point with the specified name
and marks it as holding externally mounted volumes from native host or other
containers. The value can be a JSON array, `VOLUME ["/var/log/"]`, or a plain
string with multiple arguments, such as `VOLUME /var/log` or `VOLUME /var/log
/var/db`. For more information/examples and mounting instructions via the
Docker client, refer to
[*Share Directories via Volumes*](https://github.com/docker/docker/blob/master/docs/userguide/containers/dockervolumes.md#mount-a-host-directory-as-a-data-volume)
documentation.

The `docker run` command initializes the newly created volume with any data
that exists at the specified location within the base image. For example,
consider the following Dockerfile snippet:

    FROM ubuntu
    RUN mkdir /myvol
    RUN echo "hello world" > /myvol/greeting
    VOLUME /myvol

This Dockerfile results in an image that causes `docker run`, to
create a new mount point at `/myvol` and copy the  `greeting` file
into the newly created volume.

> **注意**：
> If any build steps change the data within the volume after it has been
> declared, those changes will be discarded.

> **注意**：
> The list is parsed as a JSON array, which means that
> you must use double-quotes (") around words not single-quotes (').

## USER

    USER daemon

The `USER` instruction sets the user name or UID to use when running the image
and for any `RUN`, `CMD` and `ENTRYPOINT` instructions that follow it in the
`Dockerfile`.

## WORKDIR

    WORKDIR /path/to/workdir

The `WORKDIR` instruction sets the working directory for any `RUN`, `CMD`,
`ENTRYPOINT`, `COPY` and `ADD` instructions that follow it in the `Dockerfile`.

It can be used multiple times in the one `Dockerfile`. If a relative path
is provided, it will be relative to the path of the previous `WORKDIR`
instruction. For example:

    WORKDIR /a
    WORKDIR b
    WORKDIR c
    RUN pwd

The output of the final `pwd` command in this `Dockerfile` would be
`/a/b/c`.

The `WORKDIR` instruction can resolve environment variables previously set using
`ENV`. You can only use environment variables explicitly set in the `Dockerfile`.
For example:

    ENV DIRPATH /path
    WORKDIR $DIRPATH/$DIRNAME
    RUN pwd

The output of the final `pwd` command in this `Dockerfile` would be
`/path/$DIRNAME`

## ARG

    ARG <name>[=<default value>]

The `ARG` instruction defines a variable that users can pass at build-time to
the builder with the `docker build` command using the `--build-arg
<varname>=<value>` flag. If a user specifies a build argument that was not
defined in the Dockerfile, the build outputs an error.

```
One or more build-args were not consumed, failing build.
```

The Dockerfile author can define a single variable by specifying `ARG` once or many
variables by specifying `ARG` more than once. For example, a valid Dockerfile:

```
FROM busybox
ARG user1
ARG buildno
...
```

A Dockerfile author may optionally specify a default value for an `ARG` instruction:

```
FROM busybox
ARG user1=someuser
ARG buildno=1
...
```

If an `ARG` value has a default and if there is no value passed at build-time, the
builder uses the default.

An `ARG` variable definition comes into effect from the line on which it is
defined in the `Dockerfile` not from the argument's use on the command-line or
elsewhere.  For example, consider this Dockerfile:

```
1 FROM busybox
2 USER ${user:-some_user}
3 ARG user
4 USER $user
...
```
A user builds this file by calling:

```
$ docker build --build-arg user=what_user Dockerfile
```

The `USER` at line 2 evaluates to `some_user` as the `user` variable is defined on the
subsequent line 3. The `USER` at line 4 evaluates to `what_user` as `user` is
defined and the `what_user` value was passed on the command line. Prior to its definition by an
`ARG` instruction, any use of a variable results in an empty string.

> **Note:** It is not recommended to use build-time variables for
>  passing secrets like github keys, user credentials etc.

You can use an `ARG` or an `ENV` instruction to specify variables that are
available to the `RUN` instruction. Environment variables defined using the
`ENV` instruction always override an `ARG` instruction of the same name. Consider
this Dockerfile with an `ENV` and `ARG` instruction.

```
1 FROM ubuntu
2 ARG CONT_IMG_VER
3 ENV CONT_IMG_VER v1.0.0
4 RUN echo $CONT_IMG_VER
```
Then, assume this image is built with this command:

```
$ docker build --build-arg CONT_IMG_VER=v2.0.1 Dockerfile
```

In this case, the `RUN` instruction uses `v1.0.0` instead of the `ARG` setting
passed by the user:`v2.0.1` This behavior is similar to a shell
script where a locally scoped variable overrides the variables passed as
arguments or inherited from environment, from its point of definition.

Using the example above but a different `ENV` specification you can create more
useful interactions between `ARG` and `ENV` instructions:

```
1 FROM ubuntu
2 ARG CONT_IMG_VER
3 ENV CONT_IMG_VER ${CONT_IMG_VER:-v1.0.0}
4 RUN echo $CONT_IMG_VER
```

Unlike an `ARG` instruction, `ENV` values are always persisted in the built
image. Consider a docker build without the --build-arg flag:

```
$ docker build Dockerfile
```

Using this Dockerfile example, `CONT_IMG_VER` is still persisted in the image but
its value would be `v1.0.0` as it is the default set in line 3 by the `ENV` instruction.

The variable expansion technique in this example allows you to pass arguments
from the command line and persist them in the final image by leveraging the
`ENV` instruction. Variable expansion is only supported for [a limited set of
Dockerfile instructions.](#environment-replacement)

Docker has a set of predefined `ARG` variables that you can use without a
corresponding `ARG` instruction in the Dockerfile.

* `HTTP_PROXY`
* `http_proxy`
* `HTTPS_PROXY`
* `https_proxy`
* `FTP_PROXY`
* `ftp_proxy`
* `NO_PROXY`
* `no_proxy`

To use these, simply pass them on the command line using the `--build-arg
<varname>=<value>` flag.

### Impact on build caching

`ARG` variables are not persisted into the built image as `ENV` variables are.
However, `ARG` variables do impact the build cache in similar ways. If a
Dockerfile defines an `ARG` variable whose value is different from a previous
build, then a "cache miss" occurs upon its first usage, not its declaration.
For example, consider this Dockerfile:

```
1 FROM ubuntu
2 ARG CONT_IMG_VER
3 RUN echo $CONT_IMG_VER
```

If you specify `--build-arg CONT_IMG_VER=<value>` on the command line the
specification on line 2 does not cause a cache miss; line 3 does cause a cache
miss. The definition on line 2 has no impact on the resulting image. The `RUN`
on line 3 executes a command and in doing so defines a set of environment
variables, including `CONT_IMG_VER`. At that point, the `ARG` variable may
impact the resulting image, so a cache miss occurs.

Consider another example under the same command line:

```
1 FROM ubuntu
2 ARG CONT_IMG_VER
3 ENV CONT_IMG_VER $CONT_IMG_VER
4 RUN echo $CONT_IMG_VER
```
In this example, the cache miss occurs on line 3. The miss happens because
the variable's value in the `ENV` references the `ARG` variable and that
variable is changed through the command line. In this example, the `ENV`
command causes the image to include the value.

## ONBUILD

    ONBUILD [INSTRUCTION]

The `ONBUILD` instruction adds to the image a *trigger* instruction to
be executed at a later time, when the image is used as the base for
another build. The trigger will be executed in the context of the
downstream build, as if it had been inserted immediately after the
`FROM` instruction in the downstream `Dockerfile`.

Any build instruction can be registered as a trigger.

This is useful if you are building an image which will be used as a base
to build other images, for example an application build environment or a
daemon which may be customized with user-specific configuration.

For example, if your image is a reusable Python application builder, it
will require application source code to be added in a particular
directory, and it might require a build script to be called *after*
that. You can't just call `ADD` and `RUN` now, because you don't yet
have access to the application source code, and it will be different for
each application build. You could simply provide application developers
with a boilerplate `Dockerfile` to copy-paste into their application, but
that is inefficient, error-prone and difficult to update because it
mixes with application-specific code.

The solution is to use `ONBUILD` to register advance instructions to
run later, during the next build stage.

Here's how it works:

1. When it encounters an `ONBUILD` instruction, the builder adds a
   trigger to the metadata of the image being built. The instruction
   does not otherwise affect the current build.
2. At the end of the build, a list of all triggers is stored in the
   image manifest, under the key `OnBuild`. They can be inspected with
   the `docker inspect` command.
3. Later the image may be used as a base for a new build, using the
   `FROM` instruction. As part of processing the `FROM` instruction,
   the downstream builder looks for `ONBUILD` triggers, and executes
   them in the same order they were registered. If any of the triggers
   fail, the `FROM` instruction is aborted which in turn causes the
   build to fail. If all triggers succeed, the `FROM` instruction
   completes and the build continues as usual.
4. Triggers are cleared from the final image after being executed. In
   other words they are not inherited by "grand-children" builds.

For example you might add something like this:

    [...]
    ONBUILD ADD . /app/src
    ONBUILD RUN /usr/local/bin/python-build --dir /app/src
    [...]

> **Warning**: Chaining `ONBUILD` instructions using `ONBUILD ONBUILD` isn't allowed.

> **Warning**: The `ONBUILD` instruction may not trigger `FROM` or `MAINTAINER` instructions.

## STOPSIGNAL

	STOPSIGNAL signal

The `STOPSIGNAL` instruction sets the system call signal that will be sent to the container to exit.
This signal can be a valid unsigned number that matches a position in the kernel's syscall table, for instance 9,
or a signal name in the format SIGNAME, for instance SIGKILL.

## Dockerfile examples

Below you can see some examples of Dockerfile syntax. If you're interested in
something more realistic, take a look at the list of [Dockerization examples](https://github.com/docker/docker/blob/master/docs/examples/index.md).

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
