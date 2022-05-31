# Linux-ssh
Linux—修改ssh远程登录信息


http://t.zoukankan.com/liuhaidon-p-11870625.html
修改ssh远程登录端口

1.修改ssh服务的配置文件：/etc/ssh/sshd_config ，将 Port 22 改为 Port 3120 保存退出。

[root@localhost ~]# vi /etc/ssh/sshd_config
2.修改防火墙规则

# 打开防火墙配置文件
[root@localhost ~]# vi /etc/sysconfig/iptables

# 同意3120端口，在文件里面添加下面一行，保存退出
-A INPUT -p tcp -m state --state NEW -m tcp --dport 3120 -j ACCEPT
3.重启防火墙与ssh服务

# 重启防火墙 
[root@localhost ~]# service iptables restart

# 重启ssh服务 
[root@localhost ~]# service sshd restart
[root@localhost ~]# /etc/init.d/ssh restart
现在可以使用 3120 端口远程登录了。建议在修改过程中在本地 tcping -t ip 端口，实时检测下修改的远程端口通没通。

禁止root远程登录

1.修改ssh服务的配置文件：/etc/ssh/sshd_config ，找到PermitRootLogin，将后面的yes改为no，这样root就不能远程登录了，保存退出。

[root@localhost ~]# vi /etc/ssh/sshd_config
2.重启ssh服务

[root@localhost ~]# service sshd restart
值得一提的是，如果你的Linux中只有root用户，在关闭root远程登录之前，请一定要建立一个新用户，否则会导致无法使用ssh远程登录服务器！

允许或禁止指定用户或IP进行SSH登录

限制用户 SSH 登录

1.只允许指定用户进行登录（白名单）：

在 /etc/ssh/sshd_config 配置文件中设置 AllowUsers 选项，（配置完成需要重启 SSHD 服务）格式如下：

AllowUsers    aliyun test@192.168.1.1   # 允许 aliyun 和从 192.168.1.1 登录的 test 帐户通过 SSH 登录系统。
2.只拒绝指定用户进行登录（黑名单）：

在 /etc/ssh/sshd_config 配置文件中设置 DenyUsers  选项，（配置完成需要重启 SSHD 服务）格式如下：

DenyUsers zhangsan aliyun     # 拒绝 zhangsan、aliyun 帐户通过 SSH 登录系统
DenyUsers mayun@35.12.15.2    # 拒绝 mayun 帐户且IP为25.12.15.2 通过 SSH 登录系统
限制IP SSH 登录

除了可以禁止某个用户登录，我们还可以针对固定的IP进行禁止登录，linux 服务器通过设置 /etc/hosts.allow 和 /etc/hosts.deny 这两个文件，可以实现限制或者允许某个或者某段IP地址远程 SSH 登录服务器。

hosts.allow 和hosts.deny 两个文件同时设置规则的时候，hosts.allow 文件中的规则优先级高，按照此方法设置后服务器只允许 192.168.0.1 这个 IP 地址的 ssh 登录，其它的 IP 都会拒绝。

1.vim /etc/hosts.allow， 添加

sshd:ALL # 允许全部的 ssh 登录 
sshd:192.168.0.1:allow    # 允许 192.168.0.1 这个 IP 地址 ssh 登录
sshd:192.168.0.1/24:allow # 允许 192.168.0.1/24 这段 IP 地址的用户登录
2.vim /etc/hosts.deny，添加

sshd:ALL   # 拒绝全部的 ssh 登录
ssh密钥登录Linux
一、生成ssh秘钥对

[root@localhost ~]# ssh-keygen -t rsa
[root@localhost ~]# ssh-keygen -t rsa -C "公钥内容注释" -f "生成的文件名"
说明：命令执行后会有提示，输入三次回车即可，执行完成后会在当前用户的.ssh目录下生成两个文件：id_rsa、id_rsa.pub文件，前者时私钥文件，后者是公钥文件（拷贝到其他主机只需要拷贝公钥文件的内容即可）

二、配置公钥

[root@localhost ~]# echo -e '#this is keys_root' >> ~/.ssh/authorized_keys ;    # 这一步可以忽略
[root@localhost ~]# cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys             # 将生成的公钥写入到用户的authorized_keys文件中。
[root@localhost ~]# cat ~/.ssh/authorized_keys                                  # 这里放在了/root/.ssh下面，就是root用户登陆。想用什么用户登陆，就放在该用户家目录下的 .ssh 目录下的authorized_keys文件中。
注：每个用户都拥有自己的 authorized_keys文件。建议该文件权限对拥有者为读写权限，其他用户无权限，即600权限。

三、配置私钥（xshell使用私钥登录）

[root@localhost ~]# sz id_rsa    # 下载私钥到本地机器
ssh免密登录Linux
ssh使用私钥登录大致步骤就是：主机A（客户端）创建公钥私钥，并将公钥复制到主机B（被登陆机）的指定用户下，然后主机A使用保存私钥的用户登录到主机B对应保存公钥的用户。这里介绍主机A免密登录主机B。

一、在需要免密登陆的主机（主机A）下生成公钥和私钥。

[root@localhost ~]# ssh-keygen -t rsa     # -t rsa可以省略，默认就是生成rsa类型的密钥
二、将客户机的公钥复制到被登陆的主机（主机B）上的 ~/.ssh/authorized_keys 文件中，在主机A上面执行。

[root@localhost ~]# ssh-copy-id -i ~/.ssh/id_rsa.pub ubuntu@192.168.1.142
使用 ssh-copy-id 进行拷贝公钥非常方便，只需要指定目标主机和目标主机的用户即可。

执行上面这条命令后会自动将登录主机（A）的公钥文件内容追加至目标主机（B）中指定用户（root或ubuntu或其他用户）的.ssh目录下的authorized_keys文件中。这个过程是全自动的，非常方便。当然也可以手动复制到目标主机上。

三、免密登录

[root@localhost ~]# ssh ubuntu@192.168.1.142
上例只能实现主机A免密登陆到主机B的ubuntu用户，如果想让主机B也免密登录到主机A，创建密钥和拷贝步骤相同。

密钥登陆的方式只能登录被登录机中 .ssh 目录下有对应公钥的用户，如果想让所有用户都可以被登录则需要将authorized_keys文件的内容追加到其他用户的 ~/.ssh/authorized_keys 文件中。

如果使用自己复制的方法，一定要注意.ssh目录和authorized_keys文件的权限，前者是700，后者是600。

https://www.cnblogs.com/henkeyi/p/10487553.html

https://www.onqc.com/25.html

