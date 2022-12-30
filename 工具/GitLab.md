



配置文件位置

```
/etc/gitlab/gitlab.rb

/var/opt/gitlab/gitlab-rails/etc/gitlab.yml
```







### 什么是GitLab？

> ​    GitLab是由GitLabInc.开发，使用MIT许可证的基于网络的Git仓库管理工具，且具有wiki和issue跟踪功能。使用Git作为代码管理工具，并在此基础上搭建起来的web服务。

> ​    GitLab由乌克兰程序员DmitriyZaporozhets和ValerySizov开发，它使用Ruby语言写成。后来，一些部分用Go语言重写。截止2018年5月，该公司约有290名团队成员，以及2000多名开源贡献者。GitLab被IBM，Sony，JülichResearchCenter，NASA，Alibaba，Invincea，O’ReillyMedia，Leibniz-Rechenzentrum(LRZ)，CERN，SpaceX等组织使用。

在我们开始之前我们先来更新下我们系统，这个可有可无，我这个是最小安装





# 安装流程

## 更新系统(可选)

```shell
yum update -y
```

## 安装ssh

远程链接服务

### 安装sshd

```shell
yum install -y curl policycoreutils-python openssh-server
```

### 启动sshd

```shell
# 设置开机自启
systemctl enable sshd
# 启动sshd
systemctl start sshd
```

### 配置防火墙



#### 2.3 接下来我们配置下防火墙：

​    打开 /etc/sysctl.conf 文件，在文件最后添加新的一行

```powershell
net.ipv4.ip_forward = 1
复制代码
```

![在这里插入图片描述](https:////p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/36a8efff0f324150acf15754b327bb8a~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

#####     我们只需要sysctl.conf在最后添加一行，按下esc 加:wq 保存即可

#### 2.4 启用并启动防火墙：

```powershell
systemctl enable firewalld
systemctl start firewalld
复制代码
```

#####   这里由于是演示，我这里就把http放行

```powershell
firewall-cmd --permanent --add-service=http
复制代码
```

#### 2.5 重启防火墙：

```powershell
systemctl reload firewalld
复制代码
```

##### 以上操作步骤：

![在这里插入图片描述](https:////p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e156a979929a4f3c866128f7852607ea~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

### 安装 postfix

#####     GitLab 需要使用 postfix 来发送邮件。当然，也可以使用 SMTP 服务器。

#### 3.1 安装postfix

```powershell
yum install -y postfix

```

#####    打开 /etc/postfix/main.cf 文件，在第 119 行附近找到 inet_protocols = all，将 all 改为 ipv4

```powershell
inet_protocols = ipv4

```

![在这里插入图片描述](https:////p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ebb675ea7a7b4c7692f469558bd9d9c6~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

#### 3.2 启用并启动 postfix：

```powershell
systemctl enable postfix 
systemctl start postfix

```

### 3.3 配置 swap 交换分区

> ​    由于 GitLab 较为消耗资源，我们需要先创建交换分区，以降低物理内存的压力。 在实际生产环境中，如果服务器配置够高，则不必配置交换分区。

### 3.4 新建 2 GB 大小的交换分区：

```powershell
dd if=/dev/zero of=/root/swapfile bs=1M count=2048
复制代码
```

### 3.5 接下来我们对其格式化

```powershell
mkswap /root/swapfile
swapon /root/swapfile
复制代码
```

![在这里插入图片描述](https:////p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2cfc850de13f42ebb9786b386a5aef31~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

#####    添加自启用。打开 /etc/fstab 文件，在文件最后添加新的一行

```powershell
/root/swapfile swap swap defaults 0 0
复制代码
```

![在这里插入图片描述](https:////p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/42558515d80144f18a50cc25efa374c4~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

### 四，接下里我们安装git

#### 4.1 安装 GitLab

> 将软件源修改为国内源 由于网络环境的原因，将 repo 源修改为清华大学 。

##### 在 /etc/yum.repos.d 目录下新建 gitlab-ce.repo 文件并保存。内容如下：

```powershell
[gitlab-ce]
name=Gitlab CE Repository
baseurl=https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el$releasever/
gpgcheck=0
enabled=1
复制代码
```

#### 4.2 修改完 yum 源，因此先重新生成缓存：

（此步骤执行时间较长，一般需要 3~5 分钟左右，请耐心等待）

```powershell
yum makecache
复制代码
```

#### 4.3 安装 GitLab：

（此步骤执行时间较长，一般需要 3~5 分钟左右，请耐心等待）

```powershell
yum install -y gitlab-ce
复制代码
```

![在这里插入图片描述](https:////p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/597c47be90c2460d9c167c6b9c9660d7~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp) ![在这里插入图片描述](https:////p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b7bc9b0a8a6242448624f3d45d10d4a8~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

#### 五， 初始化 GitLab

##### 5.1 配置 GitLab 的域名（非必需）

######     打开 /etc/gitlab/gitlab.rb 文件，在第 13 行附近找到 external_url '[gitlab.example.com'，将单引号中的内容改为自己的域名（带上协议头，末尾无斜杠）](https://link.juejin.cn?target=http%3A%2F%2Fgitlab.example.com'%EF%BC%8C%E5%B0%86%E5%8D%95%E5%BC%95%E5%8F%B7%E4%B8%AD%E7%9A%84%E5%86%85%E5%AE%B9%E6%94%B9%E4%B8%BA%E8%87%AA%E5%B7%B1%E7%9A%84%E5%9F%9F%E5%90%8D%EF%BC%88%E5%B8%A6%E4%B8%8A%E5%8D%8F%E8%AE%AE%E5%A4%B4%EF%BC%8C%E6%9C%AB%E5%B0%BE%E6%97%A0%E6%96%9C%E6%9D%A0%EF%BC%89)

```powershell
external_url 'http://119.29.102.85'
复制代码
```

![在这里插入图片描述](https:////p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/89df5a9a926c4a019f2064b451549bdc~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

#### 5.2 初始化 GitLab ==特别重要！==

###### 使用如下命令初始化 GitLab：

（此步骤执行时间较长，一般需要 5~10 分钟左右，请耐心等待）

```powershell
sudo gitlab-ctl reconfigure
复制代码
```

![在这里插入图片描述](https:////p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9bbb14a3b0a64ebb95db0ad2bf8af315~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp) ![在这里插入图片描述](https:////p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6608764925604340a75d331a4e0ce5ec~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

###### 当看到这个就说明我们gitlab已经安装成功了。

##### 5.3 启动成功之后我们通过浏览器访问下

![在这里插入图片描述](https:////p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f16f2536dc5643e1b3ec1982ce08f388~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp) ![在这里插入图片描述](https:////p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/99b4f30d7ad7442499d2c3f9032ec9b9~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

##### 当我们看到进入我们就可以对我们代码进行管理了。

回到我们开始的话题，有些朋友安装成功后看到的界面可能是这个 ![在这里插入图片描述](https:////p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/68fdb26ed25c4ce2a7688416adf3b78e~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

#### 这种情况出现的原因：

#####   原因1、8080端口被tomcat占用

######     解决办法：更换端口

  安装tomcat默认的是8080端口，netstat -ntpl查看端口情况 ![在这里插入图片描述](https:////p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5928bce0c28240fca7667b6d49427990~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)    最简单的方式我们就是把8080端口kill掉，然后改下端口号 为了避免8080端口冲突问题，可以修改下的默认端口，vim打开/etc/gitlab/gitlab.rb配置文件

#### 执行重新启动

```powershell
sudo gitlab-ctl reconfigure
sudo gitlab-ctl stop
sudo gitlab-ctl start
复制代码
```

相关操作

> 启动服务：gitlab -ctl start
>  查看状态：gitlab -ctl status
>  停掉服务：gitlab -ctl stop
>  重启服务：gitlab -ctl restart
>  让配置生效：gitlab -ctl reconfigure

####     原因2、gitlab占用内存太多，导致服务器崩溃。尤其是使用阿里云服务器最容易出现502

#####      解决办法：默认情况下，主机的swap功能是没有启用的，解决办法是启动swap分区，就是我们上面启用的这里就不再过多解释了

###     以上就是我们今天的教程，如果本文对你有所帮助，欢迎关注点赞，分享给您身边的朋友。您的鼓励就是对我的最大动力。


作者：Somnus_小凯
链接：https://juejin.cn/post/6871249593749733383
来源：稀土掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。