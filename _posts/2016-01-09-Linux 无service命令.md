# Linux 无service命令
>/sbin/service: line 3: /etc/init.d/functions: No such file or directory

看到以上错误，其实是由于没有 service 命令导致。

当我们想给我们自有的服务做一个镜像的时候，首先需要从 docker 官网的镜像库下载一个纯净的 linux 镜像（我们以 centos 镜像为例），然后将容器跑起来，发现里面居然连一些基本的命令都没有，比如 service，那么我们只能自己下载。
##安装包
那么，我们怎么知道 service 是通过哪一个软件包安装的呢？我们先找到一台完整的 centos 机器，然后执行

````
$ rpm -qf /sbin/service
initscripts-9.03.49-1.el6.centos.3.x86_64

````
我们可以发现 service 是通过 initscripts 这个软件包安装获得的
##安装
知道了是哪个软件包，那么只需要执行安装命令即可

```
$ yum install initscripts 
```
