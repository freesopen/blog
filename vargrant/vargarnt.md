# Vagrant设置默认目录

​			[2016年2月17日](http://blog.rohochan.com/?p=4)[rohochan](http://blog.rohochan.com/?author=1) 					 [Leave a Comment](http://blog.rohochan.com/?p=4#respond) 				

# 更改VirtualBox虚拟机映像文件的位置

- 打开 VirtualBox 程序，点击`管理/全局设定`菜单项(Ctrl+G), 将`常规`栏里的`默认虚拟电脑位置(M)`改为其他磁盘下的路径
- 将原路径 `C:\Users\user_name\.VirtualBox\VirtualBox VMs` 下的文件移动到新路径下。
- 重新启动VirtualBox程序，在虚拟机列表里，以前建立的虚拟机虽然都还在，但已经不可用了，将他们全部删除。
- 双击打开新路径各个文件夹里的vbox文件，将建立的虚拟机重新导入。

# 更改vagrant配置文件的位置

- 将 `C:\Users\user_name\.vagrant.d` 移动到新的位置
- 新建环境变量`VAGRANT_HOME`，并指向新路径