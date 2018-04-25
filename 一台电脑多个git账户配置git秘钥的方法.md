## 1、清除git的全局设置
新安装git跳过这一步。如果对git设置过的user.name和user.email，类似这种设置过：

`$ git config --global user.name "your_name"  `

`$ git config --global user.email  "your_email"`

首先删除该设置， 不然会冲突的。取消全局设置方法：

`$ git config --global --unset user.name "your_name" ` 

`$ git config --global --unset user.email "your_email"`

如果一台电脑两个账户，自己亲测了下，不清除git的全局设置也可，关键步骤在添加并识别新的SSH key秘钥，也就是下面的3，请往下面读

## 2、生成新的SSH keys
生成ssh keys命令：

`$ ssh-keygen -t rsa -C "your_email"`
一般直接回车，默认生成`id_rsa`和`id_rsa.pub`，`id_rsa`是私钥，`id_rsa.pub`是公钥。
如果是多个git账户，需要注意的是：出现提示输入文件名的时候(`Enter file in which to save the key (~/.ssh/id_rsa): id_rsa_chen`)要输入与默认配置不一样的文件名，比如：我这里填的是 `id_rsa_personage`，另一个是 `id_rsa_company`

查看生成的ssh keys：`$ open ~/.ssh` 或者 `$ code ~/.ssh`      


## 3、添加并识别新的SSH keys私钥
<b>因为默认只读取`id_rsa`，为了让SSH识别新的私钥，需将其添加到SSH agent中 
命令：</b>
- start the ssh-agent in the background
1. `$ eval $(ssh-agent -s)` 
  <font color=#0099ff>Agent pid 59566</font>  

2. `$ ssh-agent bash`  

3. `$ ssh-add ~/.ssh/id_rsa_personage`  

4. `$ ssh-add ~/.ssh/id_rsa_company`  

<b>比如:需要分别添加`id_rsa_personage`和`id_rsa_company`。特别注意，如果后边出行权限问题：Permission denied（Publickey),很可能是私钥没有导入ssh-agent中</b>

## 4、添加新的SSH keys到Git账号的SSH设置中
将新生成的公钥`id_rsa_*.pub`添加到你的另一个github帐号(或者公司的gitlab)下的SSH Key中。 

## 5、配置~/.ssh/config文件
创建config文件，如果没有的话
`$ touch ~/.ssh/config  `      # 创建config文件
配置config文件信息
```config
#personage account  
Host github.com  
Hostname github.com  
PreferredAuthentications publickey
User acmwln  
IdentityFile ~/.ssh/id_rsa_personage
  
#company account  
Host github.com  
Hostname github.com  
PreferredAuthentications publickey
User wang_ln  
IdentityFile ~/.ssh/id_rsa_company
```

## 6、验证连接Git
连接git命令：

`$ ssh -T git@github.com`  

Hi you user name! You've successfully authenticated, but GitHub does not provide shell access.  

 上面是github的成功返回语句

## 7.使用git仓库的时候使用ssh地址：git@github.com:acmwln/timeFormat.git 
每次都需要输入用户名和密码是因为你采用的是 https 方式提交代码，第一次操作就遇到这个问题，怎么push都需要输入用户名和密码，于是百度得知需要使用ssh协议就不需要; 如果采用的是 ssh 方式只需要在版本库中添加用户的rsa 的key就可以实现提交时无需输入用户名和密码。

- 如果你的版本库已经用https 方式创建好了，那么就需要先删除原来的提交方式。在终端执行以下指令：  

1. `$ git remote rm origin`
2. `$ git remote add origin git@github.com:(用户名)/版本库名`  eg:`git@github.com:acmwln/timeFormat.git`

- 然后这个时候你使用下面指令提交代码：  

1.`git push -u origin master`

- 此时系统会提示你没有权限巴拉巴拉的，此时你需要在本地创建自己的SSH KEY

1.`ssh-keygen -t rsa -C "用户名"`


下面附上一个超链接---[git 配置多个SSH-Key](https://blog.csdn.net/dqchouyang/article/details/54898910)

tips:我发现每次在github上创建一个新的仓库，就需要重新加入私钥,这个我也不知道为什么，暂时先这样操作吧：（疑惑脸） 

1. `$ ssh-add ~/.ssh/id_rsa_personage`

2. `git remote add origin git@github.com:(用户名)/版本库名`  

3. `git remote -v `     [^_^]:查看远程连接库

4. `git push -u origin master`

....同上