# Raspberry Pi4b 集群基于Raspbian Buster搭建K3S集群同时开启KubernetesDashboard和TraefikDashboard 

## 写在前面
设备使用的是4台树莓派4B8G版<br>
<img width="800" alt="image" src="https://user-images.githubusercontent.com/1888585/123137313-68916f80-d486-11eb-9b67-f62ebca2df54.png"><br><br>
原本是想在树莓派中直接烧录**rancherOS**，但是烧录之后无法启动。<br>
google了一下发现截止到当前版本**1.5.8**，**官方的镜像并不能支持树莓派4**。<br>
rancher os git官方 [https://github.com/rancher/os](https://github.com/rancher/os)<br>

**1.5.5**版提供了**rancheros-raspberry-pi64的镜像支持树莓派3**，但是同样**不支持树莓派4**.<br>
手里有树莓派3的朋友可以去尝试。<br>
地址在这里 [https://github.com/rancher/os/releases/download/v1.5.5/rancheros-raspberry-pi64.zip](https://github.com/rancher/os/releases/download/v1.5.5/rancheros-raspberry-pi64.zip)<br>

关于树莓派4玩家关于这个问题的讨论与挣扎在这里 [Cannot boot RancherOS on Raspberry Pi 4 #2875](https://github.com/rancher/os/issues/2875)<br>

期待未来能够早日在树莓派4上玩到rancherOS。<br>

另外，git评论中有人提到**基于K3S在Pi和其他IOT设备**上运行的另一个Rancher的项目——**k3OS**<br>
有兴致的朋友可以自己去尝试。<br>
<img width="800" alt="image" src="https://user-images.githubusercontent.com/1888585/122863172-ea817b80-d354-11eb-9525-2960fa34e266.png">
<br><br>

k3OS的中文官网<br>
[https://www.rancher.cn/k3os/](https://www.rancher.cn/k3os/)<br>
关于rancherOS和k3OS区别的一些讨论可以看这里<br>
[https://github.com/rancher/k3os/issues/201](https://github.com/rancher/k3os/issues/201)<br>
<img width="800" alt="image" src="https://user-images.githubusercontent.com/1888585/122863406-44824100-d355-11eb-8861-391111c5cad8.png">
<br><br>


## 正式开始

# 往SD卡上刷系统
首先，我在树莓派4上最终烧录的系统是 Raspbian Buster <br>
通过这个链接进去找最新的镜像来下载<br>
[http://downloads.raspberrypi.org/raspios_arm64/images/](http://downloads.raspberrypi.org/raspios_arm64/images/)<br>
我们这里用到的是这个 [2021-05-07-raspios-buster-arm64](http://downloads.raspberrypi.org/raspios_arm64/images/raspios_arm64-2021-05-28/2021-05-07-raspios-buster-arm64.zip)<br><br>
<img width="500" alt="image" src="https://user-images.githubusercontent.com/1888585/122952907-e4b98380-d3b0-11eb-9c1e-bfc8c64a4e19.png">
<br><br>
烧录工具使用官方的imager或者balena的etcher都可以，etcher会快一些<br>

下载地址<br>

<img height ="50" alt="image" src="https://user-images.githubusercontent.com/1888585/122865252-a7c1a280-d358-11eb-9b66-92eb3ee3c910.png"> [Raspberry Pi Imager ](https://www.raspberrypi.org/software/)
<br>
<img height ="50" alt="image" src="https://user-images.githubusercontent.com/1888585/122865289-b740eb80-d358-11eb-9f76-0840a05a8f7f.png"> [balenaEtcher](https://www.balena.io/etcher/)
<br><br>


烧录过程就非常简单
<br>
1、选择操作系统——选择"擦除"
<br>
2、选择SD卡
<br>
3、烧录 (这一步就是格式化一下这张SD卡)
<br>
<img width="600" alt="image" src="https://user-images.githubusercontent.com/1888585/122865792-87deae80-d359-11eb-83b7-648388efd021.png">

接下来继续
<br>
1、选择操作系统——选择"使用自定义镜像" 选取刚才下载到的镜像
<br>
2、选择SD卡
<br>
3、烧录 (确认后会要求输入一次密码)
<br>
<img width="600" alt="image" src="https://user-images.githubusercontent.com/1888585/122953843-850fa800-d3b1-11eb-9b79-b40b90f420cc.png">
<br><br>
等几分钟就好 
<br><br>
<img width="600" alt="image" src="https://user-images.githubusercontent.com/1888585/122954005-a83a5780-d3b1-11eb-893e-742da2bff429.png">
<br><br>
这就算弄好了 
<br><br>
<img width="600" alt="image" src="https://user-images.githubusercontent.com/1888585/122955761-f69c2600-d3b2-11eb-967f-006af791e863.png">
<br><br>
别急着拔掉SD卡，还差一步<br>
在SD卡的根目录下创建一个命名为ssh的空文件<br>
只要有这个文件，系统启动的时候就会自动开启ssh<br>
<img width="600" alt="image" src="https://user-images.githubusercontent.com/1888585/122956223-63172500-d3b3-11eb-80fc-4ca7d29acb56.png">
<br><br>

# 进入Raspbian Buster

系统刷好之后，给树莓派插上SD卡。<br>
无屏幕的情况下，有了刚才在系统boot目录下放进去的ssh空文件，这时候直接插上电源和网线就可以直接通过终端SSH访问了 <br><br>

但是首先我们要知道它自动获取的IP地址<br>
最简单的方式可以直接从路由器里找到这台新接入的设备<br>
<img width="600" alt="image" src="https://user-images.githubusercontent.com/1888585/122957073-27308f80-d3b4-11eb-8b1a-dde2c1853804.png">
<br><br>
以上面从路由里看到的IP为例<br>

通过终端直接远程登录 默认的用户名就是pi <br>
```ssh -p22 pi@192.168.31.178```

<br>
顺便一提，如果这个IP之前使用SSH登陆过的话，会出现错误（接下来第一次登入另外三台树莓派的时候默认分配到了同一个地址就会遇到） <br>
ECDSA host key for 192.168.31.178 has changed and you have requested strict checking.
Host key verification failed.<br>
这是因为当前这台机器保留了上一次登录的目标服务器的缓存和公钥，清除就好<br>

终端执行命令  ```ssh-keygen -R 192.168.31.178```<br>

回到 ```ssh -p22 pi@192.168.31.178``` 这里<br>

登录时会给你一个新的 ECDSA key 输入 **yes** 继续<br>
首次登陆的默认密码是 raspberry<br>

成功登陆后，会提示建议修改当前密码<br>
输入当前密码(raspberry) 再输入两次新密码就可以了。<br>
<img width="600" alt="image" src="https://user-images.githubusercontent.com/1888585/122959107-9490f000-d3b5-11eb-98c4-02641a65d792.png"> 
<br><br><br>


# 开启root用户登录

首先要说的是，Debian(Raspdebian)和Ubuntu一样。在发行版默认锁定了root用户，普通用户也可以通过**sudo + 命令**来获取到root权限去执行命令。<br>
如果希望继续使用普通用户账号的话，后面如果遇到权限不足的情况，命令前面请加上**sudo**<br>
说一千道一万最终目的就是怕你把系统给玩儿坏了捅娄子。<br>
但是在家自己折腾树莓派，就不用那么谨慎了。<br>

总之，先看看这篇文章，接受一下**生产安全教育**。<br>
[Ubuntu 中的 root 用户：你应该知道的重要事情](https://zhuanlan.zhihu.com/p/104469251)<br>
<br>
三思之后，我们最终走到了这一步。<br>

```sudo su``` 切换到 root 用户<br>
<img width="300" alt="image" src="https://user-images.githubusercontent.com/1888585/122960974-87c0cc00-d3b6-11eb-97fa-e8e9e2e1fbbc.png">
<br><br>
接着先要给root用户设置一个密码
```passwd```
<br>
<img width="290" alt="image" src="https://user-images.githubusercontent.com/1888585/122961086-a4f59a80-d3b6-11eb-8504-c2aab1f3af2c.png">
<br><br>
先随手装个vim<br>
```sudo apt install vim```
<br><br>

然后去修改sshd配置<br>
```vim /etc/ssh/sshd_config```
<br>

找到 ```PermitRootLogin prohibit-password```
<br>

修改为 ```PermitRootLogin yes```
<br>
<img width="300" alt="image" src="https://user-images.githubusercontent.com/1888585/122962916-6f51b100-d3b8-11eb-9bdc-c0efc6c1d420.png">
<br><br>

重启sshd<br>

```systemctl restart sshd```
<br>

退出登录，使用root账号密码重新登入<br>
```ssh -p22 root@192.168.31.178```
<br>

这下root用户就开启成功了 <br>



# 设置静态IP<br>
这一步我们继续给这台树莓派设置一个静态IP<br>
去编辑 dhcpcd.conf 配置文件<br>
```vim /etc/dhcpcd.conf```
<br>
找到下面这部分<br>
```
# Example static IP configuration:
#interface eth0
#static ip_address=192.168.0.10/24
#static ip6_address=fd51:42f8:caae:d92e::ff/64
#static routers=192.168.0.1
#static domain_name_servers=192.168.0.1 8.8.8.8 fd51:42f8:caae:d92e::1
```
```.```
<br>

放开注释<br>
修改 static ip_address=192.168.31.81/24  (IP/24是子网掩码255.255.255.0)<br>
删掉 static ip6_address<br>
修改 static routers=192.168.31.1 (这是我的路由器地址 也就是网关)<br>
修改 static domain_name_servers=114.114.114.114 (也可以用 8.8.8.8)<br>
<br>
这段改完了是这样<br>
```
interface eth0
static ip_address=192.168.31.81/24
static routers=192.168.31.1
static domain_name_servers=114.114.114.114
```
```.```
<br>
<img width="480" alt="image" src="https://user-images.githubusercontent.com/1888585/122966489-f8b6b280-d3bb-11eb-8b28-631a8efdfc37.png">
<br>
<br><br>
# 设置时区
通过 ```date```  查看一下系统时间<br>
<img width="256" alt="image" src="https://user-images.githubusercontent.com/1888585/122966195-9bbafc80-d3bb-11eb-904b-1719ba39e3eb.png"><br>
目前是默认的 英国夏令时(BST) 我们需要回到亚洲来<br>
执行```tzselect``` 
<br>
选择 4(Asia) 9(China) 1(Beijing) 1(Yes)<br> 
<img width="508" alt="image" src="https://user-images.githubusercontent.com/1888585/122966740-403d3e80-d3bc-11eb-9613-ccfd38ced95d.png">
<br><br>
还差一步<br>
复制文件到/etc/localtime目录下<br>
```cp /usr/share/zoneinfo/Asia/Shanghai  /etc/localtime```
<br>
再看一下 **date**<br>
<img width="228" alt="image" src="https://user-images.githubusercontent.com/1888585/122966845-64991b00-d3bc-11eb-96a2-92b2941852c8.png">
<br>
这就对了 <br><br>


# 切换下载源<br><br>
因为众所周知的网络原因，默认的下载源速度会很慢或无法访问。因此我们需要切换为国内的下载源。<br>
**需要注意的是**，我们在树莓派4上装的Raspbian Buster系统需要的是 **64位（arm64版本）** 的源<br>
这里使用**清华**的镜像地址<br>
[https://mirrors.tuna.tsinghua.edu.cn](https://mirrors.tuna.tsinghua.edu.cn)<br><br>
步骤如下：<br>
```cd /etc/apt```
<br>
先备份一下<br>
```cp sources.list ./sources.list.backup```
<br>
然后编辑 sources.list <br>
```vim sources.list```<br>
<br>
<img width="600" alt="image" src="https://user-images.githubusercontent.com/1888585/122968036-aececc00-d3bd-11eb-9c22-fbf36f1ccae4.png">
<br>
全换掉<br>
替换内容如下 <br><br>
```
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
<br>
<img width="700" alt="image" src="https://user-images.githubusercontent.com/1888585/122968181-d6259900-d3bd-11eb-92d9-dcdcdf98e845.png">
<br><br>







<br>还有一处<br>
```vim /etc/apt/sources.list.d/raspi.list```
<br><br>
内容替换为<br> `deb http://mirrors.tuna.tsinghua.edu.cn/raspberrypi/ buster main ui`
<br><br>

更改完 sources.list 之后执行 <br>

```sudo apt update```
<br>
```sudo apt upgrade -y```
<br><br>

更新索引使其生效<br>

<br><br><br>
# 安装Docker

Docker官网 **Debian**相关链接如下<br> 
[https://docs.docker.com/engine/install/debian/](https://docs.docker.com/engine/install/debian/)
<br>
我们需要的 **Raspbian Buster** 这个版本也在这里了<br>
<img width="746" alt="image" src="https://user-images.githubusercontent.com/1888585/122969915-cd35c700-d3bf-11eb-98f3-ce8b29ce1719.png">
<br><br>
如果机器上有旧版本，可以通过下面命令来删除<br>
```sudo apt-get remove docker docker-engine docker.io containerd runc```<br>
<br><br>
下面我们来用官方推荐的 Install using the repository 进行安装 <br>
先更新一下现有的package <br>
```sudo apt update``` <br>
再安装一些依赖的包 HTTPS相关<br>
```sudo apt-get install  apt-transport-https  ca-certificates  curl  gnupg  lsb-release```<br>
<img width="800" alt="image" src="https://user-images.githubusercontent.com/1888585/122970259-40d7d400-d3c0-11eb-822c-a6974ccb43e6.png">
<br><br>

下一步来添加Docker官方的GPG key<br>
```curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg```<br>
<br>
还没完，继续将源导到入系统中 <br>
```echo "deb [arch=arm64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null```
<br><br>

接下来就可以开始安装 Docker Engine 了 <br>
<br>
先执行 
```sudo apt-get update``` 
把刚才添加的源更新 <br>
<br>
开始安装docker-ce<br>
```sudo apt-get install docker-ce docker-ce-cli containerd.io```
<br><br><br>
# 替换Docker镜像源
同样原因，我们需要把docker的镜像源换成国内镜像。<br>
这里使用科大的镜像源 [https://docker.mirrors.ustc.edu.cn](https://docker.mirrors.ustc.edu.cn)<br>

操作方法<br>
进入/etc/docker 去查找 daemon.json 并进行修改，如果没有这个文件就直接新建一个<br>
```vim /etc/docker/daemon.json```<br>
内容如下<br>
```
{
"registry-mirrors":["https://docker.mirrors.ustc.edu.cn"]
}
```
<br>
<img width="400" alt="image" src="https://user-images.githubusercontent.com/1888585/122892892-65a85900-d378-11eb-8d94-6e8f30f657d8.png">
<br><br>

顺便设置一下docker随开机自启动<br>
```systemctl enable docker```<br>
重启docker<br>
```systemctl restart docker```<br>
查看状态<br>
```systemctl status docker```<br>
<br><br><br>
对4台树莓派进行同样的操作，静态IP按顺序设置好。<br>
至此，我们的系统环境就搭建完成了。<br>
下面开始进入容器服务集群的搭建。<br><br><br>


## 搭建K3S集群及Dashboard 
这里我们正式开始搭建K3S集群，并将部署kubernetes dashboard实现可视化操作，同时开放Traefik的dashboard(WEB UI)，最后会通过Nginx和Ingress将两个Dashboard暴露给集群外网络环境。<br>
<br>
### 部署K3S
这里有两个选择。<br>
1.通过K3S官方的安装脚本，分别在每一台主机上执行。<br>
2.使用AutoK3s来协助安装。<br>
我们分别来看<br><br>

### 使用K3S官方脚本进行安装
官方脚本地址在这里<br>
[https://docs.rancher.cn/docs/k3s/installation/install-options/_index/](https://docs.rancher.cn/docs/k3s/installation/install-options/_index/)<br><br>
<img width="600" alt="image" src="https://user-images.githubusercontent.com/1888585/123514497-e68f8980-d6c5-11eb-91ec-82e161d78412.png">
<br><br>
官方脚本分别安装在各节点主机上之后，可以根据实际需要进行配置后各自启动为Server(Master)或Agent(Worker)。<br>
这里我们为了省事，直接在脚本后加上简单的参数，分别在各台主机上通过脚本直接安装和启动。<br><br>

**启动 K3S Server(Master)**<br>
在用来做Master的主机上直接执行<br>
```curl -sfL http://rancher-mirror.cnrancher.com/k3s/k3s-install.sh | INSTALL_K3S_MIRROR=cn K3S_NODE_NAME=pi1 sh -```
<br>
这里使用的是官方的国内地址<br>
参数中加了个K3S_NODE_NAME,也就是这台节点的名字<br><br>
<img width="700" alt="image" src="https://user-images.githubusercontent.com/1888585/123514832-6bc76e00-d6c7-11eb-8634-27d02cbfc2e5.png">
<br><br>
Server启动之后，需要获取任意一台Server的Token给后面的Agent来使用<br>
执行命令<br>
```cat /var/lib/rancher/k3s/server/node-token```
<br>
<img width="700" alt="image" src="https://user-images.githubusercontent.com/1888585/123514890-b0530980-d6c7-11eb-8414-0ec5ba60cee9.png">
<br><br>
记住这个token<br>
<br><br>
**启动 K3S Agent(Worker)**
在Agent主机上执行<br>
```curl -sfL http://rancher-mirror.cnrancher.com/k3s/k3s-install.sh | INSTALL_K3S_MIRROR=cn K3S_NODE_NAME=pi2 K3S_URL=https://192.168.31.81:6443 K3S_TOKEN=K104765c8f2ec3fd560d5594adec956de7d5d841cafdd88cde6270ebffc4a138067::server:7a29c75fedf9ceca1ec863b8b028f38f sh -```
<br><br>
其中K3S_NODE_NAME是节点名称，K3S_URL是Server的地址。<br>
K3S_TOKEN就是上面拿到的token，需要完整复制过来。<br><br>
<img width="1000" alt="image" src="https://user-images.githubusercontent.com/1888585/123514997-5c94f000-d6c8-11eb-8efd-d5079daacd4a.png">
<br><br>
都执行完毕后，来到Server主机上<br>
```kubectl get nodes```<br>
可以看到server和agent都启动好了<br>
```kubectl get pods -A``` <br>
能查看到pods也都运行起来了 <br>
<img width="700" alt="image" src="https://user-images.githubusercontent.com/1888585/123515406-d4174f00-d6c9-11eb-861d-311b96bc4de2.png">
<br><br>
这里我们使用了1台Server，3台Agent的方式搭建了一个单节点的K3S集群，这种方式相对简单，比较适合实验学习和测试环境。
如果用做生产环境，请参考高可用的方式，部署多台Server，同时要使用外部数据存储一类的东西。
至此使用K3S官方脚本进行安装的过程就告一段落了 <br><br>

### 使用AutoK3s工具进行安装

我们还可以用另一种方式来部署K3S集群。<br><br>
AutoK3s 是用于Rancher新出的一款用于简化K3s集群管理的轻量级工具。可以非常简便的在各大云厂商上通过API、CLI和UI等方式快速创建K3s。<br>
由于目前来看它实在是太新了(目前的版本是0.4.3)，功能尚未完善，同时存在着一些问题。 <br>
官方地址如下<br>
[什么是 AutoK3s](https://docs.rancher.cn/docs/k3s/autok3s/_index/)
<br><br>
首先我们可以在本地电脑上的Docker环境下安装AutoK3s。<br>
```docker run -itd --restart=unless-stopped -p 8080:8080 cnrancher/autok3s:v0.4.3```
<br><br>
当然直接在MacOS或者Windows系统中安装也没问题，上面的官网页面上有安装脚本和对应的程序。<br>
安装好之后，浏览器访问 localhost:8080 就可以看到了<br><br>

<img width="1000" alt="image" src="https://user-images.githubusercontent.com/1888585/123515789-6ec45d80-d6cb-11eb-94ea-5a037c3c3b3d.png">
<br><br>
选择左侧的Cluster，去点击页面右上的Create<br>

<img width="1000" alt="image" src="https://user-images.githubusercontent.com/1888585/123515918-09bd3780-d6cc-11eb-850a-a55f2341a0e9.png">
<br><br>

我们使用的是自己的服务器集群，所以Provider这里选择native<br>
给集群起个名字 RaspberryPiCluster<br>
分别配置了1台Master主机和3台Worker主机的IP<br>
下面的Advance这里点击show把详情显示出来<br><br>
由于我们的树莓派都是使用账号密码登录，所以SSH Key Path这里的内容需要直接删掉，否则会报错 can't get user home directory <br>
PS:此处报错的内容还被反馈了个issue作为了bug，相关内容在这里 [fix(native): fix get user home directory for native provider](https://github.com/cnrancher/autok3s/pull/333) <br><br>
继续，SSH Password 就是root的密码 <br>
<img width="1000" alt="image" src="https://user-images.githubusercontent.com/1888585/123516330-193d8000-d6ce-11eb-94f3-8834ffc56175.png"> <br><br>
然后点开K3s Option这里，把 K3s Install Script 选成 http://rancher-mirror.cnrancher.com/k3s/k3s-install.sh 这个。和脚本安装一样，用的是国内源。<br>
<br>
<img width="1000" alt="image" src="https://user-images.githubusercontent.com/1888585/123516353-3a9e6c00-d6ce-11eb-8746-e21f2085e3ba.png">
<br><br>
然后点开Additional Options，可以看到UI这个选项默认是Disable。<br><br>
这个UI就是kubernetes的**dashboard**了。<br>
**注意此处有坑** <br>
**如果这个时候把UI给Enable的话，那么在一会儿开始自动部署各节点执行命令的时候，会默认对外开放kubernetes的外部端口433。**<br>
**这样一来，Traefik在启动时，会因为主机的433端口被占用而启动失败。**<br><br>
```Warning FailedScheduling default-scheduler 0/1 nodes are available: 1 node(s) didn't have free ports for the requested pod ports.```<br><br>
因此这里就保持UI选项是Disable就可以了。我们后面再手动部署一下kubernetes dashboard即可。<br><br>
都设置好之后，点击 Create 按钮开始执行部署。**中途可能会各种报错** -_-<br>
所以我们还可以通过边上的 Ganerate CLI Command 按钮来直接获取完整的执行命令 <br>
<br>
<img width="1000" alt="image" src="https://user-images.githubusercontent.com/1888585/123516742-34a98a80-d6d0-11eb-9a37-a7338b53d828.png">
<br><br>
其实生成的命令中，也是缺点东西的... <br>
其实就是最后缺了个name变量 加上就好  --name 集群的名字<br>
完整的命令如下<br><br>

```autok3s create --provider native --k3s-channel stable --k3s-install-mirror INSTALL_K3S_MIRROR=cn --k3s-install-script http://rancher-mirror.cnrancher.com/k3s/k3s-install.sh --master 0 --ssh-password xxxxxxx --ssh-port 22 --ssh-user root --worker 0 --master-ips 192.168.31.81 --worker-ips 192.168.31.82,192.168.31.83,192.168.31.84 --name raspberrypicluster```
<br><br>
本地(就是装autok3s的电脑)执行一下命令，就能看到4个节点陆续都部署上去了
<br><br>
<img width="800" alt="image" src="https://user-images.githubusercontent.com/1888585/123516963-60794000-d6d1-11eb-8235-1c99409fc50c.png">
<br><br>
来到Master这台主机上<br>
```kubectl get nodes```<br>
可以看到server和agent都启动好了<br>
```kubectl get pods -A``` <br>
能查看到pods也都运行起来了 ```.```<br><br>
<img width="700" alt="image" src="https://user-images.githubusercontent.com/1888585/123517006-856db300-d6d1-11eb-9390-4bf169468773.png">
<br><br>
至此我们使用AutoK3s在4台树莓派上(又)部署好了一个 1 Master 3 Worker 的K3S集群。<br><br>
如需卸载节点可以参考下面的命令<br><br>
卸载K3s server <br><br>
```/usr/local/bin/k3s-uninstall.sh```
卸载k3s agent<br><br>
```/usr/local/bin/k3s-agent-uninstall.sh```
<br><br>

### 部署Kubernetes Dashboard (WEB UI) 
官网页面在这里<br><br>
[Web UI (Dashboard)](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/)<br><br><br>
我们可以在Master这台机器上直接执行官方的recommended.yaml<br>
命令行如下<br>
```kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.2.0/aio/deploy/recommended.yaml```<br>
执行完之后<br>
```kubectl get pods -A```<br>
查看一下<br>
<img width="700" alt="image" src="https://user-images.githubusercontent.com/1888585/123532518-488cd500-d740-11eb-8b5f-08cbba6c8603.png">
<br><br>
Kubernetes Dashboard已经装好了<br>
在Master上执行命令<br>
```kubectl proxy```<br>
然后就可以在这台机器上访问了，地址如下<br>
http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/<br>
<br><br>
目前默认状态下，kubernetes-dashboard并没有对外暴露<br>
通过命令可查看kubernetes-dashboard相关的service<br>
```kubectl -n kubernetes-dashboard get svc```<br>
<img width="700" alt="image" src="https://user-images.githubusercontent.com/1888585/123532688-e634d400-d741-11eb-868b-dbaad2d84624.png">
<br>
能看到目前只可内部访问<br><br>

接下来我们用以Nodeport的形式将dashboard服务暴露出去<br>
最简单的办法可以使用kubectl命令直接修改刚才已经部署好的kubernetes-dashboard服务<br>
```kubectl  patch svc kubernetes-dashboard -n kubernetes-dashboard -p '{"spec":{"type":"NodePort","ports":[{"port":443,"targetPort":8443,"nodePort":30443}]}}'```
<br><br><br>
还有一种方式是把上面的recommended.yaml先wget下来，然后修改<br>
找到文件中kind: Service这部分<br>
<img width="308" alt="image" src="https://user-images.githubusercontent.com/1888585/123532780-c3ef8600-d742-11eb-891f-acf9c1c4b3c3.png">
<br>
在 spec: 下加上 type: NodePort ，并在 ports: 这里面加上 nodePort: 30443<br>
修改后的这部分内容如下<br>

```
---

kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
spec:
  type: NodePort
  ports:
    - port: 443
      targetPort: 8443
      nodePort: 30443
  selector:
    k8s-app: kubernetes-dashboard

---
```
<br><br>
修改好文件之后再更新一下就可以了<br>
```kubectl apply -f recommended.yaml```
<br>
<br>
我们再来查看kubernetes-dashboard的service<br>
```kubectl -n kubernetes-dashboard get svc```
<br>

<img width="1200" alt="image" src="https://user-images.githubusercontent.com/1888585/123532895-d28a6d00-d743-11eb-9dce-1319911e7da4.png">
<br><br>
这时候就可以通过浏览器来访问dashboard了<br>

<img width="500" alt="image" src="https://user-images.githubusercontent.com/1888585/123532921-0d8ca080-d744-11eb-8bef-be454d170169.png">
<br><br>
这里如果是Firefox的话，可以通过配置例外来正常访问。<br>
我们用的是Chrome，这里有一个骚操作<br>
鼠标点一下这个页面的空白处，然后直接输入 thisisunsafe <br>
这个过程什么也看不到，就是盲敲键盘就可以，像极了小时候玩游戏时输入秘籍的场景<br>
然后就可以访问了<br>
<br>
<img width="500" alt="image" src="https://user-images.githubusercontent.com/1888585/123533020-b1764c00-d744-11eb-9e51-4eec58122538.png">
<br><br><br><br>
接下来我们来到了登录环节<br><br>
首先回到Master机器的终端<br><br>

```cd /var/lib/rancher/k3s/server/manifests/```<br><br>


来到manifests目录创建dashboard-adminuser.yaml文件<br><br>
```
cat > dashboard-adminuser.yaml << EOF
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard  
EOF
```

<br><br>
然后就可以查找到token了<br><br>
```kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')```
<br><br>
<img width="1000" alt="image" src="https://user-images.githubusercontent.com/1888585/123533199-f5b61c00-d745-11eb-8c4b-99ccb4ced0f3.png">
<br><br>
复制到刚才的登录页面中，登录<br><br>

<img width="1000" alt="image" src="https://user-images.githubusercontent.com/1888585/123533223-24cc8d80-d746-11eb-8b86-9b48d55fa20c.png">
<br><br>

### 开启Traefik UI
traefik UI 在通过官方脚本部署K3S的时候就已经安装好了。<br>
但是默认状态下并未对外暴露。<br><br><br>
这里**需要注意**的是当前安装的K3S版本是v1.21.1-k3s1，其中对应安装好的**traefik版本是2.4**<br>
因此原有的1.7和1.8版本的修改yaml文件开启UI的方式(dashboard:enable)就不适用了，我们需要创建dashboard.yaml文件为traefik配置。<br>
<br><br>
traefik在Github上关于开启Traefik Dashboard相关页面在这里<br><br>
[traefik / traefik-helm-chart](https://github.com/traefik/traefik-helm-chart)
<br>
参考 Exposing the Traefik dashboard 这一段<br>
<br><br>
我们回到manifests目录<br>
```cd /var/lib/rancher/k3s/server/manifests/```<br><br>
创建dashboard-adminuser.yaml文件<br>
内容如下<br><br>
```
# dashboard.yaml
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: dashboard
spec:
  entryPoints:
    - web
  routes:
    - match: Host(`traefik.k3s`) && (PathPrefix(`/dashboard`) || PathPrefix(`/api`))
      kind: Rule
      services:
        - name: api@internal
          kind: TraefikService
```
<br><br>
这里我们设置了一个traefik的访问域名是 traefik.k3s <br><br>
```kubectl apply -f dashboard.yaml```<br>
更新一下<br>
(以上内容也可以直接在kubernetes dashboard中输入YAML内容那里来执行)<br><br>
配置一下本地电脑上的host <br>
```192.168.31.81   traefik.k3s```<br>
<br>
现在我们可以通过 http://traefik.k3s/dashboard/ 来访问Traefik的WEB UI了。<br>
<img width="1000" alt="image" src="https://user-images.githubusercontent.com/1888585/123536149-1dfc4580-d75b-11eb-96b8-f29765ecd489.png">
<br><br>

### 部署一个Nginx
部署一个Nginx
接下来我们在kubernetes dashboard部署一个Nginx服务并通过Ingress为它配置一个域名并暴露在集群服务器的网络下<br><br>
首先进入到kubernetes dashboard 页面，点击右上角的加号<br>
输入YAML内容并上传，启动一个Nginx服务。<br>
YAML内容如下<br><br>
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.18-alpine
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: NodePort
```
<br><br>
<img width="1000" alt="image" src="https://user-images.githubusercontent.com/1888585/123534574-b5a86680-d750-11eb-9aed-6717316a33cf.png">
<br><br>
上传好之后，我们在Deployments中可以看到部署好的Nginx，同时在Services里面可以找到Nginx的service<br>
<img width="1000" alt="image" src="https://user-images.githubusercontent.com/1888585/123534798-3fa4ff00-d752-11eb-8076-a3de880abd80.png">
<br><br>
接下来还是点击加号，继续输入YAML来配置一个Ingress，host配置traefik.dracula.io域名<br><br>
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: traefik-nginx
  namespace: default
  annotations:
    kubernetes.io/ingress.class: traefik
spec:
  rules:
  - host: traefik.dracula.io
    http:
      paths:
      - backend:
          serviceName: nginx-service
          servicePort: 80
```
<br><br>
```.```再来配置一下本地电脑上的host <br>
```192.168.31.81   traefik.dracula.io```<br>
<br>
现在通过浏览器访问 traefik.dracula.io 可以看到Nginx页面。<br>
<br>
<img width="800" alt="image" src="https://user-images.githubusercontent.com/1888585/123536213-8fd48f00-d75b-11eb-8954-7a99dadfb33b.png">
<br><br>
至此，4台树莓派组成的K3S集群搭建完成。<br>
后来我买了乐高拼了个壳<br>
<img width="1000" alt="image" src="https://user-images.githubusercontent.com/1888585/123536424-d37bc880-d75c-11eb-831a-94a30c27a6bd.png">

<br><br>
