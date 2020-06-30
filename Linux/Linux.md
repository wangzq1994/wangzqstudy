# LINUX

## 安装注意点

- / 主分区  文件系统类型 ext3 最大
- /boot 理论4m就可以 主要是配置文件 ext
- 文件系统类型 选择 swap 2048m

设置网络 注意要把 **网络设置为桥接网卡** 

下图主机名也可以自定义

![image-20200630101848436](E:\mianshixuexi\wangzqstudy\Linux\Linux.assets\image-20200630101848436.png)

### Contos7

#### 防火墙

1、命令行界面输入命令“systemctl status firewalld.service”并按下回车键。

2、然后在下方可度以查看得到“active（running）”，此时说明防火墙已经被打开了。

3、在命令行中输入systemctl stop firewalld.service命令，进行关闭防火墙。

4、然后再使用命令systemctl status firewalld.service，在下方出现disavtive（dead），这权样就说明防火墙已经关闭。

5、再在命令行中输入命令“systemctl disable firewalld.service”命令，即可永久关闭防火墙。

#### **selinux状态**

##### 1、查看

```
[root@dev-server ~]# getenforce
Disabled
[root@dev-server ~]# /usr/sbin/sestatus -v
SELinux status:                 disabled
```

##### 2、临时关闭

```
##设置SELinux 成为permissive模式
##setenforce 1 设置SELinux 成为enforcing模式
setenforce 0
```

##### 3、永久关闭

```
vi /etc/selinux/config
```

将SELINUX=enforcing改为SELINUX=disabled
设置后需要重启才能生效

#### 设置共享目录

前提先安装VMware ToolS工具

##### 主机操作

![image-20200630140537470](E:\mianshixuexi\wangzqstudy\Linux\Linux.assets\image-20200630140537470.png)

##### Linux中路径

![image-20200630140508174](E:\mianshixuexi\wangzqstudy\Linux\Linux.assets\image-20200630140508174.png)

#### 网络配置

VMWare设置

![image-20200630150548025](E:\mianshixuexi\wangzqstudy\Linux\Linux.assets\image-20200630150548025.png)

Linux

![image-20200630150643072](E:\mianshixuexi\wangzqstudy\Linux\Linux.assets\image-20200630150643072.png)

#### centos7操作SSH/SSHD服务(查看/启动/重启/自启)

查看状态：

```
systemctl status sshd.service
```

启动服务：

```
systemctl start sshd.service
```

重启服务：

```
systemctl restart sshd.service
```

开机自启：

```
systemctl enable sshd.service
```

## Linux命令

#### man XXX  帮助命令

![image-20200630153441926](E:\mianshixuexi\wangzqstudy\Linux\Linux.assets\image-20200630153441926.png)

![image-20200630153456942](E:\mianshixuexi\wangzqstudy\Linux\Linux.assets\image-20200630153456942.png)

#### 具体命令--help

#### info 具体命令(不常用)

#### 6个终端

Ctrl+Alt+F1/F2......F6

Ctrl+Alt+F7,回到图形化界面

#### who 查看有几个终端连接

#### cat /etc/inittab

centos7不再使用inittab方式来设置开机不自启图形界面

常用3和5 3是命令 5是图形