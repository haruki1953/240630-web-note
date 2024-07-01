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