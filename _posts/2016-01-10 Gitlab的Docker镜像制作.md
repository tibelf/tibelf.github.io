#Gitlab 的 docker 镜像制作
今天我这里来介绍下，如果制作 Gitlab 的容器镜像。
##基础镜像
首先需要下载一个基础镜像，这里我选择了 centos6.7 作为我的基础镜像，当然你也可以选择更高版本的 centos ，或者是选择 ubuntu 作为你的基础镜像，镜像下载地址 [centos docker镜像](https://hub.docker.com/_/centos/)
##Gitlab 下载
基础镜像下载好后，就需要下载 Gitlab 的rpm包了，因为受到GFW的影响，下载官方的rpm包会很慢，所以我建议大家使用国内的源来进行下载，我这里使用的是：[清华的源](http://mirror.tuna.tsinghua.edu.cn/)，我选择下载了版本为：7.13.0 的 gitlab 
##安装Gitlab
软件下载好后，就可以开始启动容器，然后在容器中安装gitlab了。
###启动容器
第一步，就是启动容器，如果执行如下命令：

```
docker run --name gitlab -P -p 8080:80 -d -it centos:latest /bin/bash
```
>-p：由于 gitlab 暴露的端口是80，但是在容器外80端口往往有可能会被占用，所以我们一般需要进行端口映射
>-d：daemon

然后你会发现你在安装 gitlab 的时候，会报一个错误

```
Unable To Connect To Upstart: Failed To Connect To Socket /Com/Ubuntu/Upstart: Connection Refused
```
这是由于你启动容器的时候，执行的 CMD 是 `/bin/bash`，所以启动的PID 1 的程序是 bash，但是 gitlab 在安装过程中会将 chef，redis 等程序作为 daemon 启动，错误显示的是 upstart 没有启动，这是因为你的 PID 1 不是 /sbin/init 导致。

>我们来分析下，其实在启docker容器的时候，其实就是启了一个进程，如果这个进程退出了，那么这个容器也就停止了。比如我们使用`/bin/bash`这个指令执行的时候，其实就是在容器中启动了一个 `bash` 进程，我们执行`exit`命令的同时，容器也停止了。所以也就意味着，你不能简单的在容器中通过指令`/etc/init.d/nginx start`启动一个后台程序，因为当你执行了后台程序后，原先的程序就会退出，原因如上。
>那么如果我们想实现在容器中跑多个服务怎么办呢，有很多种方式，但是前提条件就是，你的PID 1必须是`init`，作为进程管理者。

接下来，我们更新我们的启动命令：
```
docker run --name gitlab -P -p 8080:80 -d -it centos:latest /sbin/init
```

恭喜你，刚才的错误不再出现了，但是又出现了新的错误

```
STDERR: error: "Read-only file system" setting key "net.ipv4.ip_forward"
```
这是由于 docker 容器默认启动模式是 unprivileged，所以我们需要加一个 `--privileged` 参数，从而给容器赋予更多的权限

最后，我们的启动命令应该为：
```
docker run --privileged --name gitlab -P -p 8080:80 -d -it centos:latest /sbin/init
```
###安装Gitlab
其实官网有 Gitlab 的详细安装文档可以参见：[gitlab安装文档](https://about.gitlab.com/downloads)，只需要选择你的基础环境，然后就可以根据上面的指令执行安装即可。
