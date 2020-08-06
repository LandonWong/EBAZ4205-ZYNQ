# 制作可以在OpenStack上运行的KVM虚拟机镜像(以Ubuntu 20.04为例)
## wangsongyue18@mails.ucas.ac.cn
# 简介
实验内容: 制作一个可以在OpenStack上正确运行的KVM虚拟机镜像.\
本次实验使用Linux发行版Ubuntu 20.04桌面版为系统镜像进行配置(`ubuntu-20.04-desktop-amd64.iso`), 配置的宿主机环境为Ubuntu 18.04. \
另外, 您需要保证您的计算机支持KVM虚拟化. 您可通过以下命令查询.
```
grep -E -o '(vmx|svm)' /proc/cpuinfo
```
若输出`vmx`则支持.
# 具体步骤

## 基本开发环境配置
在制作镜像前, 您首先需要配置qemu, KVM虚拟机管理工具virt以及VNC Client. 以下是安装前两者的命令.
```
$ sudo apt install qemu
$ sudo apt install virt
```
## 创建磁盘
在OpenStack上运行的KVM需要镜像的磁盘格式为`qcow2`. 下面首先利用`qemu-img`工具创建一块`qcow2`格式的虚拟磁盘. 命令格式如下.
```
qemu-img create -f <dist_format> <disk_img_path> <size>
```
下面是例子, 在本目录下创建了一块40G大小的, 名称为`ubuntu.qcow2`的虚拟磁盘.
```
$ qemu-img create -f qcow2 ./ubuntu.qcow2 40G
```
注意: 此虚拟磁盘在创建之初并不会真正耗费40G空间, 其实际大小根据后期写入弹性伸缩.
## 挂载安装盘启动
下面我们将Ubuntu安装到这个虚拟磁盘上. 
##　创建并启动虚拟机
首先我们挂载`ubuntu-20.04-desktop-amd64.iso`和这块虚拟磁盘创建并启动一个KVM虚拟机.
```
$ virt-install --virt-type kvm --name Ubuntu \
> --ram 8192 --disk ./ubuntu.qcow2,format=qcow2 \
> --graphics vnc,listen=0.0.0.0 --noautoconsole \
> --os-type=linux --os-variant=ubuntu16.04 \
> --network network=default \
> --cdrom=/home/landon/Downloads/ubuntu-20.04-desktop-amd64.iso 
```
其中各选项根据您的实际选择. 成功启动后, 您将看到以下提示.
```
Starting install
Domain installation still in progress. You can reconnect to the console to complete the installation process.
```
此时根据提示可知, 虚拟机已经在运行. 下面我们将通过VNC连接到虚拟机以完成安装部署过程.
## 连接到KVM虚拟机
前面创建虚拟机时, 我们指定了网络为默认. 而virt工具默认会创建一个网桥`virbr0`, 您可以通过此网桥连接至虚拟机. IP地址可通过`ifconfig`查询. \
查询到ip后, 可直接打开VNC Client输入并连接, 无需端口号.
## 配置虚拟机内环境

### 系统安装
现在已经可以对虚拟机进行操作了. 首先先根据提示完成系统的安装.\
关于磁盘的分区方式, 推荐使用Ubuntu安装引导程序的默认选项, 其包括了一个启动分区, 一个swap分区和主分区等.
### 系统配置
在完成了系统安装后, 需要我们继续对此虚拟机进行配置
