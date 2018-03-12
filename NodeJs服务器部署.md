## Nodejs 项目的线上服务器部署与发布

### 1.生产环境所需要素

1.购买自己的域名

2.购买自己的服务器

3.域名备案

4.配置服务器应用环境

5.安装配置数据库

6.项目远程部署发布与更新



### 2.涉及技术点

ssh、nginx、node、mongoDB、PM2、IPtables、github



### 3.SSH登录服务器

普通ssh登录

```shell
ssh ubuntu@119.29.93.219 #登录系统
df -h #查看磁盘使用情况
```

使用mac的zsh中的软连接来建立快捷命令：

```shell
code ~/.zshrc
#在.zshrc最后添加
alias ssh_t='ssh ubuntu@119.29.93.219'
#保存，回到命令行重新加载环境变量
source ~/.zshrc
```

在腾讯云中默认不是用root用户进行登录，配置允许root进行ssh登录

```shell
#使用ubuntu用户登录到服务器上
sudo passwd root 
#设置root的登录密码
sudo vi /etc/ssh/sshd_config
#修改 PermitRootLogin without-password 为 PermitRootLogin yes
#按 esc 然后输入 :wq 保存退出
#重启ssh服务
sudo service ssh restart
```

一般不使用root/ubuntu这种超级用户进行服务器操作，危险性太大。下面为添加用户流程

用户名：jiamiao_xie

密码：[password]

```shell
#添加用户
adduser jiamiao_xie  #设置密码和基础信息等，确定Y即可
gpasswd -a jiamiao_xie sudo  #给用户添加权限，让该用户可以使用sudo命令

sudo visudo ##编辑文件
# # User privilege specification
# 新增一行
jiamiao_xie ALL=(ALL:ALL) ALL
```

本地电脑初始化.ssh

```shell
#在~目录下查看是否有.ssh文件夹，文件夹是否有 id_rsa 和 id_rsa.pub两个文件，如果有跳过下面步骤，如果没有执行下面的命令
ssh-keygen -t rsa -b 4096 -C "xiejiamiao0326@hotmail.com"   #一路回车即可
```

 本地电脑开启ssh代理

```shell
eval "$(ssh-agent -s)" #开启SSH代理
ssh-add ~/.ssh/id_rsa  #把ssh秘钥加入到代理中
```

服务端初始化.ssh并开启ssh代理

```shell
ssh-keygen -t rsa -b 4096 -C "xiejiamiao0326@hotmail.com"
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_rsa
```

创建并编辑 ~/.ssh/authorized_key 文件

```shell
cd ~/.ssh
vi authorized_keys
#↓↓↓↓本地电脑操作↓↓↓↓
cat ~/.ssh/id_rsa.pub
#拷贝本地公钥
#↑↑↑↑本地电脑完毕↑↑↑↑
#↓↓↓↓服务器↓↓↓↓
# 按i进入vi编辑模式，拷贝本地的公钥到 authorized_keys 文件中 
# 按esc回到vi命令模式，输入 :wq! 保存文件
```

修改authorized_key文件权限

```shell
chmod 600 authorized_keys
sudo service ssh restart
```

修改ssh默认登录端口

```shell
sudo vi /etc/ssh/sshd_config
```

修改 Port 22 为 Port 38888

在 UsePAM yes 下面新增 UseDNS no (已存在则不用加)

最下面添加 AllowUsers jiamiao_xie

保存，重启ssh服务

```shell
sudo service ssh restart
```

ssh进一步安全配置

```
PermitRootLogin no # 禁止root用户登录
PermitEmptyPasswords no # 禁止空密码
PasswordAuthentication no # 禁止密码登录，需要配置好秘钥登录之后才可进行此操作
```

升级apt

```shell
sudo apt-get update && sudo apt-get upgrade
sudo iptables -F # 清除本地iptable所有规则
sudo vi /etc/iptables.up.rules
```

拷贝下面内容

```
*filter

# allow all connection
-A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# allow out traffic
-A OUTPUT -j ACCEPT

# allow http https
-A INPUT -p tcp --dport 443 -j ACCEPT
-A INPUT -p tcp --dport 80 -j ACCEPT

# allow ssh port login
-A INPUT -p tcp -m state --state NEW --dport 38888 -j ACCEPT

# ping
-A INPUT -p icmp -m icmp --icmp-type 8 -j ACCEPT

# log denied calls
-A INPUT -m limit --limit 5/min -j LOG --log-prefix "iptables denied:" --log-level 7

# drop incoming sentive connections
-A INPUT -p tcp --dport 80 -i eth0 -m state --state NEW -m recent --set
-A INPUT -p tcp --dport 80 -i eth0 -m state --state NEW -m recent --update --seconds 60 --hitcount 150 -j DROP
```

执行命令：

```shell
sudo iptables-restore < /etc/iptables.up.rules
sudo ufw enable
```

创建自启动脚本

```shell
sudo vi /etc/network/if-up.d/iptables
```

拷贝下面内容

```
#!/bin/sh
iptables-restore /etc/iptables.up.rules
```

为脚本添加权限

```shell
sudo chmod +x /etc/network/if-up.d/iptables
```

Fila2Ban：检测可疑行为做对应的操作

```shell
sudo apt-get install fail2ban
sudo vi /etc/fail2ban/jail.conf
```

做如下修改：

```
# "bantime" is the number of seconds that a host is banned.
bantime  = 600
改为
bantime  = 3600

destemail = root@localhost
改为
destemail = xiejiamiao0326@hotmail.com

action = %(action_)s
改为
action = %(action_mw)s
```

开启关闭fail2ban

```shell
sudo service fail2ban status #查看状态
sudo service fail2ban stop #关闭
sudo service fail2ban start #开启
```

哈哈哈哈