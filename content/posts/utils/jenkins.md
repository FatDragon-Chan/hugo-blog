---
title: "Jenkins Github Webhook CI构建流程"
date: 2023-04-25T15:13:16+08:00
lastmod: 2023-04-25T15:13:16+08:00
author: ["AHone"]
keywords: 
- Jenkins Github Webhook CI
categories: 
- util
tags: 
- web
- vue
- jenkins
- debian
- github
description: ""
weight:
slug: ""
draft: false # 是否为草稿
comments: true
reward: false # 打赏
mermaid: true #是否开启mermaid
showToc: true # 显示目录
TocOpen: false # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: false # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
cover:
    image: "" #图片路径例如：posts/tech/123/123.png
    caption: "" #图片底部描述
    alt: ""
    relative: false

---

## 准备工作

- linux环境（演示环境为debian 11）
- jenkins安装包
- PushDeer



## Jenkins安装

> 新版Jenkins依赖于java11环境，记得及时切换

### 服务安装

此处开始使用的是腾讯云服务器的系统镜像 debian 11用作演示

基本上是依据文档[^1]处理，熟悉的同学可以直接跳过

```bash
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
```

执行完大概率会出现下面的问题

```bash
Job for jenkins.service failed because the control process exited with error code.
See "systemctl status jenkins.service" and "journalctl -xe" for details.
invoke-rc.d: initscript jenkins, action "start" failed.
● jenkins.service - Jenkins Continuous Integration Server
     Loaded: loaded (/lib/systemd/system/jenkins.service; enabled; vendor preset: enabled)
     Active: activating (auto-restart) (Result: exit-code) since Tue 2023-04-25 17:43:52 CST; 17ms ago
    Process: 10602 ExecStart=/usr/bin/jenkins (code=exited, status=1/FAILURE)
   Main PID: 10602 (code=exited, status=1/FAILURE)
        CPU: 4ms
dpkg: error processing package jenkins (--configure):
 installed jenkins package post-installation script subprocess returned error exit status 1
Errors were encountered while processing:
 jenkins
E: Sub-process /usr/bin/dpkg returned an error code (1)

```

此处大概率是没有安装`java-11`的问题，我们先去安装一个

```bash
sudo apt update
sudo apt install openjdk-11-jre
java -version
# 输出的版本是号是 11.x.x 就可以了
openjdk version "11.0.18" 2023-01-17
OpenJDK Runtime Environment (build 11.0.18+10-post-Debian-1deb11u1)
OpenJDK 64-Bit Server VM (build 11.0.18+10-post-Debian-1deb11u1, mixed mode, sharing)

```

 那我们重新回到`Jenkins`开始开启服务

```bash
# 服务器开启自动开启Jenkins服务
sudo systemctl enable jenkins
# 开启服务
sudo systemctl start jenkins
# 查看服务状态
sudo systemctl status jenkins
```

如果没有报错信息，此时打开 `ip:8080`即可以看到jenkins的web页面了

![image-20230425175430246](http://img.chenzian.com/uPic/image-20230425175430246_2023_04_25_17_54_30.png) 



如果看不到或者服务不可用，可以去服务器的防火墙配置里看下是否开放了8080端口，比如我演示使用的轻量云服务器

![image-20230425175504590](http://img.chenzian.com/uPic/image-20230425175504590_2023_04_25_17_55_04.png)

### Jenkins配置初始化

跟随引导前往`/var/lib/jenkins/secrets/initialAdminPassword`查看当前初始化的密钥

```bash
vim /var/lib/jenkins/secrets/initialAdminPassword

# 此时会返回初始密码，复制好后填入即可
```

  此处建议安装推荐即可，有额外需要的插件我们可以在之后单独安装

![image-20230425175840543](http://img.chenzian.com/uPic/image-20230425175840543_2023_04_25_17_58_40.png)

等待安装完成并填写账户信息及地址域名信息后即来到了主要页面

![image-20230425180739020](http://img.chenzian.com/uPic/image-20230425180739020_2023_04_25_18_07_39.png)



### Jenkins 构建

#### 项目配置

此时我们新建个新的项目来示例构建打包的过程，此处使用的是vue2的示例项目， 实际的构建项目打包可以根据需要或者项目的不同自行调整。

此处选用自由风格

![image-20230505101739530](http://img.chenzian.com/uPic/image-20230505101739530_2023_05_05_10_17_39.png)

如果是公共项目，则直接填入Git Repository URL 即可，此处是私有项目。 因此需要增加一个凭据拉取

![image-20230505101943288](http://img.chenzian.com/uPic/image-20230505101943288_2023_05_05_10_19_43.png)

### Jenkins SSH Private Key

> 仅在第一次配置使用ssh拉取仓库代码时会出现，如无ssh需求可直接跳过

#### 错误相关



##### 错误 1：Error performing git command: git ls-remote -h

一般这种情况是属于服务器没有安装git

```bash
git --version

# 如果输出这些则代表没有安装
# -bash: git: command not found

# 以示例中的 debian 11 举例
sudo apt install git

```

##### 错误 2： code 128

![image-20230506152910834](http://img.chenzian.com/uPic/image-20230506152910834_2023_05_06_15_29_10.png)

这种问题一般是没有通过ssh链接过github出现的，解决方法如下：[jenkins-host-key-verification-failed](https://stackoverflow.com/questions/15174194/jenkins-host-key-verification-failed)

```bash
# 切换jenkins
sudo su jenkins

# ssh获取
git ls-remote -h 项目地址.git HEAD
# 此时返回
# The authenticity of host 'github.com (207.223.240.181)' can't be established.
# RSA key fingerprint is 97:8c:1b:f2:6f:14:6b:5c:3b:ec:aa:46:46:74:7c:40.
# Are you sure you want to continue connecting (yes/no)?

# 确认即可 输入yes 回车
yes

# 此时返回Jenkins测试就可以了

```



### 构建配置

构建触发器选择github hook trigger for GITScm polling即可
build Steps 选择 执行shell 此处输入命令即可

![image-20230505102256547](http://img.chenzian.com/uPic/image-20230505102256547_2023_05_05_10_22_56.png)

```bash
# 此处是我的shell相关
node -v
yarn -v
yarn install
yarn run build
pwd
cd ./dist
tar -zcvf ./ts-web-warehouse.tar.gz ./
echo "打包完成"
```

jenkins构建的部分到这里就完成了

###  Publish over SSH

#### 配置 Publish over SSH

前往 Dashboard > Manage Jenkins  > Configure System  >  Publish over SSH

1、安装jenkinsf服务和Publish over SSH插件。
2、Jenkins SSH Key不需要填写
3、添加SSH Servers
Name：随便写
Hostname：服务器IP地址
Username：用户名
Remote Directory：远程目录
重要：点击Use password authentication, or use a different key
在Passphrase / Password填写用户密码

![在这里插入图片描述](http://img.chenzian.com/uPic/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3lhX3NoeQ==,size_16,color_FFFFFF,t_70_2023_05_06_16_16_04.png)

填写完毕记得测试下是否链接成功
![image-20230506162204650](http://img.chenzian.com/uPic/image-20230506162204650_2023_05_06_16_22_04.png)

#### 构建后配置

![image-20230506162258941](http://img.chenzian.com/uPic/image-20230506162258941_2023_05_06_16_22_59.png)

![image-20230506165528999](http://img.chenzian.com/uPic/image-20230506165528999_2023_05_06_16_55_29.png)

```bash
# 清理之前的文件夹
rm -rf /usr/local/nginx/html

# 创建新的文件夹
mkdir /usr/local/nginx/html

# 移动新的文件到指定目录
mv /temp/ts-web-warehouse.tar.gz /usr/local/nginx/html/ts-web-warehouse.tar.gz

# 解压文件
cd /usr/local/nginx/html
tar -zxvf ./ts-web-warehouse.tar.gz

# 重命名包带上时间日期
mv ./ts-web-warehouse.tar.gz ./ts-web-warehouse-$(date +%Y%m%d-%H%M).tar.gz

```



## 配置webhook

> 关于什么是webhook[^2]建议查看相关文档，此处推荐RedHat的文章解释，简单可以理解为通知服务端的请求

### 调试webhook

Jenkins的默认webhook地址是 ip:端口/github-webhook/   比如我此处的webhook地址则是 http://192.168.0.1:8080/github-webhook/

此时打开github项目的设置 Settings => webhooks => Add webhook

把jenkins的webhook地址填入Payload URL即可

![image-20230426100611735](http://img.chenzian.com/uPic/image-20230426100611735_2023_04_26_10_06_11.png)

配置成功后会自动ping一下webhook，当返回成功，配置即算成功了

![image-20230426100808924](http://img.chenzian.com/uPic/image-20230426100808924_2023_04_26_10_08_09.png)

### 通知推送

此处使用pushDeer推送构建成功的消息

打开pushDeer[^3]，此处使用的是pushDeer官方在线版，有意折腾的可以搞个自架服务端

安装http-request插件，步骤如下：

- 插件下载地址：http://updates.jenkins-ci.org/download/plugins/http_request/
- 按照下图所示上传插件并部署重启
- ![image-20230506171147407](http://img.chenzian.com/uPic/image-20230506171147407_2023_05_06_17_11_47.png)



此处省略了pushDeer的过程，可参考官方文档处理， 拿到了自己的key之后。在Jenkins新建一个Item

![image-20230506170420992](http://img.chenzian.com/uPic/image-20230506170420992_2023_05_06_17_04_21.png)

![image-20230506170504717](http://img.chenzian.com/uPic/image-20230506170504717_2023_05_06_17_05_04.png)

![image-20230506171552770](http://img.chenzian.com/uPic/image-20230506171552770_2023_05_06_17_15_52.png)

替换key即可

```bash
# 相关的更详细的使用可以参阅官网 https://www.pushdeer.com/official.html
https://api2.pushdeer.com/message/push?pushkey=key&text=ts-web-warehouse构建成功

```

![image-20230506171618518](http://img.chenzian.com/uPic/image-20230506171618518_2023_05_06_17_16_18.png)

### 结语

> 主要是经常会忘，这部分的知识，每次寻找的资料都是层次不齐，因此整理了一次从零的入门手册，方便后续自己寻找资料。如有错漏，欢迎指出～



[^1]: [Jenkins  官方文档](https://www.jenkins.io/doc/book/installing/linux/)
[^2]: [什么是 Webhook？](https://www.redhat.com/zh/topics/automation/what-is-a-webhook)
[^3]: [PushDeer官方在线版](https://www.pushdeer.com/)
