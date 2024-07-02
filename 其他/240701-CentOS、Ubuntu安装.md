## 安装
下载
https://mirror.nsc.liu.se/centos-store/7.6.1810/isos/x86_64/CentOS-7-x86_64-Minimal-1810.iso

![](assets/Pasted%20image%2020240701152401.png)

最后还是重装原来的ubuntu20了

## 设置

取消更新

**网络设置**
在路由器的DHCP服务中固定ip

**安装SSH服务**
`sudo apt-get install openssh-server`

### 开机自动挂载磁盘分区
#### 1、查看分区信息
```shell
sudo blkid
```
​	该命令会列出所有分区的UUID和文件系统类型，找到欲挂载的那个分区，记录下它的的UUID和type信息，稍后会用到它们。如下：
![](assets/Pasted%20image%2020240701175713.png)

#### 2、修改配置文件
​	打开 `/etc/fstab` 文件，命令如下：
```shell
sudo vim /etc/fstab
```
​	根据上一步所得信息在文件末尾加上如下内容：(soft)
![](assets/Pasted%20image%2020240701180818.png)

- 第一项，按照上一步填入分区的UUID
- 第二项，为硬盘所要挂载到的文件夹
- 第三项，根据上一步填写文件系统类型 `ntfs`
- 第四项，填写 `defaults` 
- 第五项，填写 `0`
- 第六项，填写 `2`

​	每一项用一个或多个空格隔开，最好与上面保持对齐以便查看。

​	完成后保存并退出，然后重启系统即可生效。

## 其他
开机时按Esc键可以临时选择某个盘作为启动盘

### 配置代理
```
export http_proxy=http://192.168.2.110:10811/
export https_proxy=http://192.168.2.110:10811/

# 取消代理
unset http_proxy 
unset https_proxy

# 测试
curl -I https://www.google.com
```

### 设置root
```
# 设置密码
sudo passwd root

# 使用root
su
```

### 随便装了nginx
更新包列表
```
sudo apt update
```

查找可用的 Nginx 版本
```
apt list -a nginx

dd@dd-ubuntu:~$ apt list -a nginx
正在列表... 完成
nginx/focal-security,focal-security 1.18.0-0ubuntu1.4 all
nginx/focal,focal 1.17.10-0ubuntu1 all
```

安装
```
sudo apt install nginx=1.18.0-0ubuntu1.4
```
安装后自动启动了，并且已开机自启


如果你使用 `sudo apt install nginx=1.18.0-0ubuntu1.4` 这样的命令通过包管理器安装 nginx，那么 nginx 的文件和配置路径通常是按照 Ubuntu 发行版的标准路径来安排的。这与从源代码进行编译安装 nginx 是有所不同的。

### 编译安装与包管理器安装的区别
1. **包管理器安装（如 apt）**：
   - **安装路径**：nginx 被安装到标准的系统路径，例如：
     - 可执行文件通常位于 `/usr/sbin/nginx`。
     - 主配置文件通常位于 `/etc/nginx/nginx.conf`。
     - 日志文件通常位于 `/var/log/nginx/`。
   - **默认网站根目录**：通常位于 `/usr/share/nginx/html`。
   - **配置文件目录**：主要配置文件和站点配置文件通常位于 `/etc/nginx`。

2. **编译安装**：
   - **安装路径**：可以根据编译时的选项自定义安装路径，通常是在配置时指定的。
   - **默认路径**：默认情况下，可执行文件可能位于 `/usr/local/nginx/sbin/nginx` 或 `/usr/sbin/nginx`，具体取决于编译时的设置。
   - **配置文件和日志**：通常需要在编译时指定配置文件和日志的存放路径，可能不同于包管理器安装的默认路径。


### 确认 nginx 安装路径
```
whereis nginx
```