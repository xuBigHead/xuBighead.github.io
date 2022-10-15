# 目录
[TOC]

# 概述

# 仓库管理

# 代码管理
## pull语法
fatal: refusing to merge unrelated histories

```
$ git pull gitee master --allow-unrelated-histories
```

## push语法

# 配置
## 配置SSH

- 首先在本地生成id_rsa和id_rsa.pub文件
    
本地安装完成git后打开bash.exe执行下述命令，邮箱是在github上注册的账号绑定的邮箱，执行该命令中输入数据时直接按`enter`键跳过即可；
```
$ ssh-keygen -t rsa -C "*******@qq.com"
Generating public/private rsa key pair.
Enter file in which to save the key (/c/Users/Administrator/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /c/Users/Administrator/.ssh/id_rsa.
Your public key has been saved in /c/Users/Administrator/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:sYdBnHEU/HhBkbRX87eXg4jffxCyAWl+c2ZOdSgwDUU *******@qq.com
The key's randomart image is:
+---[RSA 2048]----+
|       .o=B@Eo o.|
|       .o.=.=...=|
|        oo +.o..+|
|         =+ O.B +|
|        S..+ % =.|
|         .. o o o|
|           . . . |
|              . .|
|               ..|
+----[SHA256]-----+
```

2
``` 
$ eval "ssh-agent -s"
SSH_AUTH_SOCK=/tmp/ssh-U3PTzE21YO3O/agent.5760; export SSH_AUTH_SOCK;
SSH_AGENT_PID=5764; export SSH_AGENT_PID;
echo Agent pid 5764;
```

3
```
$ ssh-add ~/.ssh/id_rsa
Could not open a connection to your authentication agent.
```

4
```
$ ssh-agent bash
```

5
``` 
$ ssh-add ~/.ssh/id_rsa
Identity added: /c/Users/Administrator/.ssh/id_rsa (2985742776@qq.com)
```

- 然后将本地生成的id_rsa.pub中的ssh的密钥key配置到github中

Github => Settings => SSH and GPG keys => New SSH key

## 全局配置
- 查看所有全局配置
```
git config --global -l
```
- 添加全局配置

`setting_key`表示配置名称，`setting_value`表示配置的值。
```
git config --global setting_key setting_value
```
- 取消指定全局配置

`setting_key`表示配置名称。

```
git config --global --unset setting_key
```

# 连接问题
## port 22
## port 443
```
ssh: connect to host ssh.github.com port 443: Connection refused
fatal: Could not read from remote repository.
Please make sure you have the correct access rights
and the repository exists.
```

### 域名解析污染
通过 https://ipaddress.com/website/ssh.github.com 获取 ssh.github.com 的IP地址，直接通过IP地址连接 Github。

修改 .ssh 目录下的config文件中的 Hostname 的配置 ssh.github.com 为上述获取到的IP地址。

> config file
```
Host github.com
User xuBighead
Hostname 140.82.113.36
PreferredAuthentications publickey
IdentityFile ~/.ssh/id_rsa
Port 443
```


# 参考资料
- []()
- []()
- []()
- []()
- [github push代码老被墙，有没有招儿？ ](https://www.sohu.com/a/550844893_121124361)
- [git 用命令下载代码到本地](https://www.cnblogs.com/aspirant/p/13071624.html)
