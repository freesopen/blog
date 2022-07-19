安装

```powershell
sudo apt-get install postgresql
```

```powershell
若安装
sudo apt-get install postgresql-12
```

安装完成后，PostgreSQL 服务将启动，要验证安装，使用psql工具打印服务器版本

```powershell
sudo -u postgres psql -c "SELECT version();"
```

输出应类似于以下内容：

```powershell
PostgreSQL 11.14 (Debian 11.14-0+deb10u1) on x86_64-pc-linux-gnu, compiled by gcc (Debian 8.3.0-6) 8.3.0, 64-bit
```

启用对PostgreSQL服务器的远程访问：

 打开配置文件`postgresql.conf`

```powershell
sudo vi /etc/postgresql/11/main/postgresql.conf
```

将 `listen_addresses`的值改为：

```
listen_addresses = '*'   # what IP address(es) to listen on;

# 用于测试所以就全开，如果生产环境，请指定IP地址
```

打开配置文件`pg_hba.conf`

```
sudo vi /etc/postgresql/11/main/pg_hba.conf
```

将服务器配置为接受远程登录,找到`# IPv4 local connections:`,如果不需要密码连接，需要改为trust，需要则md5验证