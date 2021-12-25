# Windows 操作系统更改 Vagrant 的 Home 目录位置

在 Windows 操作系统下，Vagrant 安装完成以后会默认把 Home 目录设为 `C:\Users\用户名\.vagrant.d`，然后所有的相关文件如 boxes 都会放在这里。

一般不建议把该目录放在 C 盘下，有以下几个原因：

- 很占系统盘空间
- 重装系统的话需要备份下来
- 碰到中文用户名容易出现各种问题

所以我们需要把 Home 指向其它非系统分区的英文目录。其实很简单，只要设定 `VAGRANT_HOME` 环境变量就可以了。

> VAGRANT_HOME can be set to change the directory where Vagrant stores global state. By default, this is set to ~/.vagrant.d. The Vagrant home directory is where things such as boxes are stored, so it can actually become quite large on disk.

命令行模式：

环境变量添加完成以后把 `.vagrant.d` 文件夹从 `C:\Users\用户名\.vagrant.d` 移动到 `E:\Home\.vagrant.d` 就可以了。

![](img\15460614-0b0a815115beb5e1.png)