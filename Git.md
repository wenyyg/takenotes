Git1

### 安装：

https://npm.taobao.org/mirrors/git-for-windows/



### 配置

右击打开Git Bash

> $git init

配置用户名、邮箱

> git config --global  user.name "wyy"     //"wyy"是自己的账户名
>
> git config --global user.email "wenyyg@163.com"   //"wenyyg@163.com"注册账户时用的邮箱

生成ssh

> ssh-keygen -t rsa

​	然后连敲三次回车键，结束后去系统盘目录下（一般在C:\Users\你的用户名.ssh）(Mac:/Users/用户/.ssh)查看是否有ssh文件夹生成，此文件夹中有两个文件

