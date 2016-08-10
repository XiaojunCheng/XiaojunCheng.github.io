# 如何在Mac上使用多个Github账号

## 背景说明

  很多开发者可能碰到这个场景：白天在公司工作使用公司分配的Github账号，晚上回家使用个人Github账号进行项目开发

  需要如何做？
  
1. 打开终端，使用如下命令生成

```
ssh-keygen -t rsa -b 4096 -C "email_1@example.com"
```

2. 输入文件名

```
Enter a file in which to save the key (/Users/you/.ssh/id_rsa): ~/.ssh/github_user_1
```

```
ssh-add ~/.ssh/github_user_1
```

配置.ssh/config

```
Host user1
  HostName github.com
  IdentityFile ~/.ssh/github_user_1

Host user2
  HostName github.com
  IdentityFile ~/.ssh/github_user_2
```

## 参考文献

- [SSH原理与运用（一）：远程登录](http://www.ruanyifeng.com/blog/2011/12/ssh_remote_login.html)
- [SSH原理与运用（二）：远程操作与端口转发](http://www.ruanyifeng.com/blog/2011/12/ssh_port_forwarding.html)
- [Generating a new SSH key and adding it to the ssh-agent](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/)
- [如何在同一台电脑上使用两个github账户](http://www.cnblogs.com/foxracle/archive/2012/07/20/2599830.html?utm_source=tuicool&utm_medium=referral)
- [mac下如何彻底切换github账号](https://segmentfault.com/q/1010000004179810)
