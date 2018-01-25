#  AWS初体验-免费搭建SS

## 1、背景

科学上网是现在很多人所需要的，尤其是需要上Google、Facebook等。新手不建议直接买VPN，个人认为购买VPS搭建自己的海外服务器比较靠谱。然而，购买VPS肯定需要从免费的入手，首推AWS和Google Cloud Platform。**前提是得有信用卡**。

- VPN：在公用网络上建立专用网络，进行加密通讯。在企业网络中有广泛应用。VPN网关通过对数据包的加密和数据包目标地址的转换实现远程访问。VPN有多种分类方式，主要是按协议进行分类。VPN可通过服务器、硬件、软件等多种方式实现。
- VPS：Virtual Private Server 虚拟专用服务器技术，将一台服务器分割成多个虚拟专享服务器的优质服务。实现VPS的技术分为容器技术，和虚拟化技术。**简单理解VPS就是一台拥有公网IP的服务器。**
- AWS：即Amazon Web Services，是亚马逊（Amazon）公司的云计算IaaS和PaaS平台服务。 

## 2、注册AWS

到[AWS](http://aws.amazon.com/cn/)进行注册。注册过程中需要绑定信用卡，会扣`$1`(预授权)。但需要注意的是，在没有用超的情况下，是免费的，作为SS服务器，只能是流量超额会导致（`15GB/月`）。注册过程中，需要填写手机号，会有国际长途（手机号默认开通）打进来，告诉你验证码。

## 3、创建AWS实例

### 1、登录AWS控制台，点击EC2（Elastic Compute Cloud）![开通AWS](http://upload-images.jianshu.io/upload_images/4324380-c71dffe71766925b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



### 2、启动虚拟机

![启动虚拟机](http://upload-images.jianshu.io/upload_images/4324380-8e05f27e6c7344f2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![启动实例](http://upload-images.jianshu.io/upload_images/4324380-2716ad552530ce18.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 3、定制化服务器类型

![AMI](http://upload-images.jianshu.io/upload_images/4324380-17a201eb78316b22.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![实例类型](http://upload-images.jianshu.io/upload_images/4324380-c04c541d52d392c8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](http://upload-images.jianshu.io/upload_images/4324380-229f82ed630dd3a9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![审核](http://upload-images.jianshu.io/upload_images/4324380-d022800585aff62e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 3、登录服务器

![连接](http://upload-images.jianshu.io/upload_images/4324380-78031132c229d1de.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![连接到实例](http://upload-images.jianshu.io/upload_images/4324380-bfa4aa4009e2fc13.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

下面主要是通过Xshell连接到EC2，具体看[教程](http://jingyan.baidu.com/article/a3a3f811d5fc338da2eb8a00.html)。**注意**：用户名那里填写**ubuntu**，否则会出现 *所选的用户密钥未在远程主机上注册 请再试一次*。

## 4、安装Shadowsocks

```
# 获取root权限
sudo -s

# 更新apt-get
apt-get update

# 安装python包管理工具
apt-get install python-setuptools
apt-get install python-pip

# 安装shadowsocks
pip install shadowsocks
```

> "or its parent directory is not owned by...", 遇到这个问题可以用命令前加上`sodu -H`

## 5、配置Shadowsocks

```
mkdir /etc/shadowsocks
vim /etc/shadowsocks/ss.json
```

配置文件内容：

```
{
    "server":"0.0.0.0",
    "server_port":443,
    "local_address":"127.0.0.1",
    "local_port":1080,
    "password":"www.jianshu.com/u/e02df63eaa87",
    "timeout":300,
    "method":"aes-256-cfb",
    "fast_open":false,
    "workers": 1
}
```

| 配置字段          | 说明                                       |
| ------------- | ---------------------------------------- |
| server        | 服务端监听地址(IPv4或IPv6)                       |
| server_port   | 服务端端口，一般为443                             |
| local_address | 本地监听地址，缺省为127.0.0.1                      |
| local_port    | 本地监听端口，一般为1080                           |
| password      | 用以加密的密匙                                  |
| timeout       | 超时时间（秒）                                  |
| method        | 加密方法，默认为aes-256-cfb，更多请查阅[Encryption](https://github.com/shadowsocks/shadowsocks/wiki/Encryption) |
| fast_open     | 是否启用[TCP-Fast-Open](https://github.com/shadowsocks/shadowsocks/wiki/TCP-Fast-Open)，true或者false |
| workers       | [worker数量](https://github.com/shadowsocks/shadowsocks/wiki/Workers) |

## 6、启动Shadowsocks

启动：`ssserver -c /etc/shadowsocks/ss.json -d start` 

停止：`ssserver -c /etc/shadowsocks/ss.json -d stop` 

重启：`ssserver -c /etc/shadowsocks/ss.json -d restart`

## 7、设置SS为开机自启动

`vi /etc/rc.local`

##  8、本地设备连接到Shadowsocks服务器

加入：`sudo ssserver -c /etc/shadowsocks/ss.json -d start`安装之后，添加服务器，地址为AWS的外网地址，登录[AWS控制台](https://ap-northeast-1.console.aws.amazon.com/ec2/)，查看正在运行中的实例，找到公有ip。 端口号为刚才配置Shadowsocks服务器时的端口号，密码也是刚才配置的(当然密码可以自行设置)，设置完之后保存。

关于Windows下使用Shadowsocks的方法，还请自行搜索。

## 其他境外服务器推荐

- DigitalOcean

- 搬瓦工(BandwagonHOST)

- Linode

  ​