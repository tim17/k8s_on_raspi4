# 4台RaspberryPi4B搭建K8s(K3s)容器集群(一)安装系统及docker环境

## 写在前面

设备使用的是4台树莓派4B8G版
![image.png](https://raw.githubusercontent.com/tim17/gallery/main/paWdlD1OAsoJfCz.png)



原本是想在树莓派中直接烧录**rancherOS**，但是烧录之后无法启动。
google了一下发现截止到当前版本**1.5.8**，**官方的镜像并不能支持树莓派4**。
rancher os git官方 [https://github.com/rancher/os](https://github.com/rancher/os)

**1.5.5**版提供了**rancheros-raspberry-pi64的镜像支持树莓派3**，但是同样**不支持树莓派4**.
手里有树莓派3的朋友可以去尝试。
地址在这里 [https://github.com/rancher/os/releases/download/v1.5.5/rancheros-raspberry-pi64.zip](https://github.com/rancher/os/releases/download/v1.5.5/rancheros-raspberry-pi64.zip)

关于树莓派4玩家关于这个问题的讨论与挣扎在这里 [Cannot boot RancherOS on Raspberry Pi 4 #2875](https://github.com/rancher/os/issues/2875)

期待未来能够早日在树莓派4上玩到rancherOS。

另外，git评论中有人提到**基于K3S在Pi和其他IOT设备**上运行的另一个Rancher的项目——**k3OS**
有兴致的朋友可以自己去尝试。
![image.png](https://raw.githubusercontent.com/tim17/gallery/main/GYP2tDK4nVRpM53.png)

k3OS的中文官网
[https://www.rancher.cn/k3os/](https://www.rancher.cn/k3os/)
关于rancherOS和k3OS区别的一些讨论可以看这里
[https://github.com/rancher/k3os/issues/201](https://github.com/rancher/k3os/issues/201)
![image.png](https://raw.githubusercontent.com/tim17/gallery/main/Y4Ng5y3BzUkltDq.png)



## 正式开始

### 往SD卡上刷系统

首先，我在树莓派4上最终烧录的系统是 Raspbian Buster 
通过这个链接进去找最新的镜像来下载
[http://downloads.raspberrypi.org/raspios_arm64/images/](http://downloads.raspberrypi.org/raspios_arm64/images/)
我们这里用到的是这个 [2021-05-07-raspios-buster-arm64](http://downloads.raspberrypi.org/raspios_arm64/images/raspios_arm64-2021-05-28/2021-05-07-raspios-buster-arm64.zip)
![image.png](https://raw.githubusercontent.com/tim17/gallery/main/spcKkaMOf9oI2G1.png)

烧录工具使用官方的imager或者balena的etcher都可以，etcher会快一些

下载地址

<img width ="50" alt="image" src="https://raw.githubusercontent.com/tim17/gallery/main/B5iQnIW7NlzP1Ge.png"> [Raspberry Pi Imager ](https://www.raspberrypi.org/software/)

<img width ="50" alt="image" src="https://raw.githubusercontent.com/tim17/gallery/main/l5CPrGZtsi17Ycd.png"> [balenaEtcher](https://www.balena.io/etcher/)



烧录过程就非常简单

1、选择操作系统——选择"擦除"

2、选择SD卡

3、烧录 (这一步就是格式化一下这张SD卡)

<img width="600" alt="image" src="https://raw.githubusercontent.com/tim17/gallery/main/HaMmesrWKwNVO2d.png">

接下来继续

1、选择操作系统——选择"使用自定义镜像" 选取刚才下载到的镜像

2、选择SD卡

3、烧录 (确认后会要求输入一次密码)

<img width="600" alt="image" src="https://raw.githubusercontent.com/tim17/gallery/main/M2vsrxduiWQGAZS.png">

等几分钟就好 

<img width="600" alt="image" src="https://raw.githubusercontent.com/tim17/gallery/main/shRKnyQYr3bcDF6.png">

这就算弄好了 

<img width="600" alt="image" src="https://raw.githubusercontent.com/tim17/gallery/main/AckOSiE8vqfpysl.png">

别急着拔掉SD卡，还差一步
在SD卡的根目录下创建一个命名为ssh的空文件
只要有这个文件，系统启动的时候就会自动开启ssh
<img width="600" alt="image" src="https://raw.githubusercontent.com/tim17/gallery/main/zFVYIUux3kJRHDt.png">


### 进入Raspbian Buster

系统刷好之后，给树莓派插上SD卡。
无屏幕的情况下，有了刚才在系统boot目录下放进去的ssh空文件，这时候直接插上电源和网线就可以直接通过终端SSH访问了 

但是首先我们要知道它自动获取的IP地址
最简单的方式可以直接从路由器里找到这台新接入的设备
<img width="600" alt="image" src="https://raw.githubusercontent.com/tim17/gallery/main/FepfrxG96ZONoyQ.png">

以上面从路由里看到的IP为例

通过终端直接远程登录 默认的用户名就是pi 

```shell
ssh -p22 pi@192.168.31.178
```

顺便一提，如果这个IP之前使用SSH登陆过的话，会出现错误（接下来第一次登入另外三台树莓派的时候默认分配到了同一个地址就会遇到） 
ECDSA host key for 192.168.31.178 has changed and you have requested strict checking.
Host key verification failed.
这是因为当前这台机器保留了上一次登录的目标服务器的缓存和公钥，清除就好

终端执行命令  

```shell
ssh-keygen -R 192.168.31.178
```


回到  

```shell
ssh -p22 pi@192.168.31.178
```

这里

登录时会给你一个新的 ECDSA key 输入 **yes** 继续
首次登陆的默认密码是 raspberry

成功登陆后，会提示建议修改当前密码
输入当前密码(raspberry) 再输入两次新密码就可以了。
<img width="600" alt="image" src="https://raw.githubusercontent.com/tim17/gallery/main/YkVMg2IlwqPiT19.png"> 



### 开启root用户登录

首先要说的是，Debian(Raspdebian)和Ubuntu一样。在发行版默认锁定了root用户，普通用户也可以通过**sudo + 命令**来获取到root权限去执行命令。
如果希望继续使用普通用户账号的话，后面如果遇到权限不足的情况，命令前面请加上**sudo**
说一千道一万最终目的就是怕你把系统给玩儿坏了捅娄子。
但是在家自己折腾树莓派，就不用那么谨慎了。

总之，先看看这篇文章，接受一下**生产安全教育**。
[Ubuntu 中的 root 用户：你应该知道的重要事情](https://zhuanlan.zhihu.com/p/104469251)

三思之后，我们最终走到了这一步。

```shell
sudo su
```

切换到 root 用户
<img width="300" alt="image" src="https://raw.githubusercontent.com/tim17/gallery/main/LazySV5TDwKsN8j.png">

接着先要给root用户设置一个密码

```shell
passwd
```

<img width="290" alt="image" src="https://raw.githubusercontent.com/tim17/gallery/main/dPrWhax7i8XCf5o.png">

先随手装个vim

```shell
sudo apt install vim
```



然后去修改sshd配置

```shell
vim /etc/ssh/sshd_config
```


找到 `PermitRootLogin prohibit-password`


修改为 `PermitRootLogin yes`

<img width="300" alt="image" src="https://raw.githubusercontent.com/tim17/gallery/main/um5tondpIJz1wb7.png">


重启sshd

```shell
systemctl restart sshd
```


退出登录，使用root账号密码重新登入

```shell
ssh -p22 root@192.168.31.178
```


这下root用户就开启成功了 



### 设置静态IP

这一步我们继续给这台树莓派设置一个静态IP
去编辑 dhcpcd.conf 配置文件

```shell
vim /etc/dhcpcd.conf
```

找到下面这部分

```yaml
# Example static IP configuration:
#interface eth0
#static ip_address=192.168.0.10/24
#static ip6_address=fd51:42f8:caae:d92e::ff/64
#static routers=192.168.0.1
#static domain_name_servers=192.168.0.1 8.8.8.8 fd51:42f8:caae:d92e::1

```


放开注释
修改 static ip_address=192.168.31.81/24  (IP/24是子网掩码255.255.255.0)
删掉 static ip6_address
修改 static routers=192.168.31.1 (这是我的路由器地址 也就是网关)
修改 static domain_name_servers=114.114.114.114 (也可以用 8.8.8.8)

这段改完了是这样

```yaml
interface eth0
static ip_address=192.168.31.81/24
static routers=192.168.31.1
static domain_name_servers=114.114.114.114
```

<img width="480" alt="image" src="https://raw.githubusercontent.com/tim17/gallery/main/DmhAOQEt7Jugj9G.png">



### 设置时区

通过 `date`  查看一下系统时间
<img width="256" alt="image" src="https://raw.githubusercontent.com/tim17/gallery/main/Hw5nScIhBJqLMFV.png">
目前是默认的 英国夏令时(BST) 我们需要回到亚洲来
执行`tzselect`

选择 4(Asia) 9(China) 1(Beijing) 1(Yes) 
<img width="508" alt="image" src="https://raw.githubusercontent.com/tim17/gallery/main/eFGNYPWhC73Vl9I.png">

还差一步
复制文件到/etc/localtime目录下

```shell
cp /usr/share/zoneinfo/Asia/Shanghai  /etc/localtime
```

再看一下 **date**
<img width="228" alt="image" src="https://raw.githubusercontent.com/tim17/gallery/main/Tdk7IFJcYwry6ls.png">

这就对了 


### 切换下载源

因为众所周知的网络原因，默认的下载源速度会很慢或无法访问。因此我们需要切换为国内的下载源。
**需要注意的是**，我们在树莓派4上装的Raspbian Buster系统需要的是 **64位（arm64版本）** 的源
这里使用**清华**的镜像地址
[https://mirrors.tuna.tsinghua.edu.cn](https://mirrors.tuna.tsinghua.edu.cn)
步骤如下：

```shell
cd /etc/apt
```

先备份一下

```shell
cp sources.list ./sources.list.backup
```

然后编辑 sources.list 

```shell
vim sources.list
```

<img width="600" alt="image" src="https://raw.githubusercontent.com/tim17/gallery/main/YNJsayKbcHpV1G6.png">

全换掉
替换内容如下 

```yaml
# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb https://mirrors.tuna.tsinghua.edu.cn/debian/ buster main contrib non-free
# deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ buster main contrib non-free
deb https://mirrors.tuna.tsinghua.edu.cn/debian/ buster-updates main contrib non-free
# deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ buster-updates main contrib non-free
deb https://mirrors.tuna.tsinghua.edu.cn/debian/ buster-backports main contrib non-free
# deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ buster-backports main contrib non-free
deb https://mirrors.tuna.tsinghua.edu.cn/debian-security buster/updates main contrib non-free
# deb-src https://mirrors.tuna.tsinghua.edu.cn/debian-security buster/updates main contrib non-free
```

<img width="700" alt="image" src="https://raw.githubusercontent.com/tim17/gallery/main/1zjGwDgda9ArcEL.png">


还有一处

```shell
vim /etc/apt/sources.list.d/raspi.list
```

内容替换为

```yaml
deb http://mirrors.tuna.tsinghua.edu.cn/raspberrypi/ buster main ui
```


更改完 sources.list 之后执行 

```shell
sudo apt update
```


```shell
sudo apt upgrade -y
```



更新索引使其生效


### 安装Docker

Docker官网 **Debian**相关链接如下 
[https://docs.docker.com/engine/install/debian/](https://docs.docker.com/engine/install/debian/)

我们需要的 **Raspbian Buster** 这个版本也在这里了
<img width="746" alt="image" src="https://raw.githubusercontent.com/tim17/gallery/main/PgO2sHEYGw6DZjy.png">

如果机器上有旧版本，可以通过下面命令来删除

```shell
sudo apt-get remove docker docker-engine docker.io containerd runc
```

下面我们来用官方推荐的 Install using the repository 进行安装 
先更新一下现有的package 

```shell
sudo apt update
```

再安装一些依赖的包 HTTPS相关

```shell
sudo apt-get install  apt-transport-https  ca-certificates  curl  gnupg  lsb-release
```

<img width="800" alt="image" src="https://raw.githubusercontent.com/tim17/gallery/main/Jhl7RFPcMqbUyka.png">


下一步来添加Docker官方的GPG key

```shell
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```


还没完，继续将源导到入系统中 

```shell
echo "deb [arch=arm64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

接下来就可以开始安装 Docker Engine 了 

先执行 

```shell
sudo apt-get update
```

把刚才添加的源更新 

开始安装docker-ce

```shell
sudo apt-get install docker-ce docker-ce-cli containerd.io
```


### 替换Docker镜像源

同样原因，我们需要把docker的镜像源换成国内镜像。
这里使用科大的镜像源 [https://docker.mirrors.ustc.edu.cn](https://docker.mirrors.ustc.edu.cn)

操作方法
进入/etc/docker 去查找 daemon.json 并进行修改，如果没有这个文件就直接新建一个

```shell
vim /etc/docker/daemon.json
```

内容如下

```yaml
{
"registry-mirrors":["https://docker.mirrors.ustc.edu.cn"]
}
```

<img width="400" alt="image" src="https://raw.githubusercontent.com/tim17/gallery/main/z3LAbdjYNJX7TW2.png">


顺便设置一下docker随开机自启动

```shell
systemctl enable docker
```

重启docker

```shell
systemctl restart docker
```

查看状态

```shell
systemctl status docker
```

对4台树莓派进行同样的操作，静态IP按顺序设置好。
至此，我们的系统环境就搭建完成了。
下面开始进入容器服务集群的搭建。


------------



