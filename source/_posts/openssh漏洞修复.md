---
title: OpenSSH权限提升漏洞修复过程
date: 2022-10-01
tags:
  - OpenSSH
  - 漏洞
categories:
  - 实战
  - 漏洞修复


---
# OpenSSH权限提升漏洞(CVE-2021-41617)

> 升级ssh最好服务器现场有人支持，避免升级失败后无法恢复问题


## 漏洞详情

OpenSSH是SSH（Secure SHell）协议的免费开源实现。
 
OpenSSH项目发布了OpenSSH 8.8安全更新，修复了OpenSSH 6.2 到 8.7版本中的 sshd(8)中的一个权限提升漏洞（CVE-2021-41617）。
 
当sshd(8)在执行AuthorizedKeysCommand或AuthorizedPrincipalsCommand时，未能正确地初始化，其中AuthorizedKeysCommandUser或AuthorizedPrincipalsCommandUser指令被设置为以非root用户身份运行。相反，这些命令将继承 sshd(8) 启动时的组的权限，根据系统配置的不同，继承的组可能会让辅助程序获得意外的权限，导致权限提升。在sshd_config(5)中，AuthorizedKeysCommand和AuthorizedPrincipalsCommand都没有被默认启用。
 
建议受影响用户做好资产自查以及预防工作，以免遭受黑客攻击。

## 影响范围
OpenSSH版本6.2-8.7


## 解决方案

注意！！！

升级之前另外开启一个窗口，保持窗口不关闭，防止升级出错时恢复服务。

tail -300f /var/log/messages

1. 查看当前ssh版本
```sh
ssh -V
```

2. 下载openssh并安装

```sh

cd /usr/local/src

# http://www.openssh.com/releasenotes.html
wget https://cdn.openbsd.org/pub/OpenBSD/OpenSSH/portable/openssh-8.8p1.tar.gz


tar -zxvf openssh-8.8p1.tar.gz

cd openssh-8.8p1

./configure && make -j 2 && make install

```

3. 备份并替换原文件

```sh

# 注：执行目录为openssh-8.8p1目录下
cp -r  /etc/ssh/sshd_config /etc/ssh/sshd_config_bak
\cp sshd_config /etc/ssh/sshd_config

```

4. 修改配置文件

```sh
# 配置一：
sed -ri 's/^#(PermitRootLogin).*/\1 yes/g' /etc/ssh/sshd_config

# 配置二：
##注意“/usr/local/libexec/sftp-server”的地址需根据实际环境来，一般为“/usr/local/libexec/sftp-server”
##也有的为“/usr/local/libexec/openssh/sftp-server”
sed -ri 's/^(Subsystem).*/\1 sftp \/usr\/local\/libexec\/sftp-server/g' /etc/ssh/sshd_config 
echo "Ciphers aes128-ctr,aes192-ctr,aes256-ctr" >> /etc/ssh/sshd_config 
echo 'OPTIONS="-f /etc/ssh/sshd_config"' >> /etc/sysconfig/sshd

# 配置三：
###以下命令物理机执行，ECS服务器不存在sshd.service配置文件 ，可跳过
sed -ri 's/^(ExecStart=).*/\1\/usr\/local\/sbin\/sshd -D $OPTIONS/g' /usr/lib/systemd/system/sshd.service 
sed -ri 's/^(Type=).*/\1simple/g' /usr/lib/systemd/system/sshd.service
```

5. 重新加载
```sh
\cp /usr/bin/ssh{,.bak}
\cp /usr/local/bin/ssh /usr/bin/ssh
systemctl daemon-reload
systemctl restart sshd

```
6. 验证结果
```sh
# 验证安装结果

ssh -V

```

