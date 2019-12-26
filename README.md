# 一台电脑配置两个SSH key的踩坑记

需求：有时候我们的代码托管在多个平台上，这就需要为每个托管平台设置SSH-key

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
> a. 生成一个公司用的SSH-Key
`$ ssh-keygen -t rsa -C "yourCompany_email"`
一般直接回车，默认生成`id_rsa`和`id_rsa.pub`，`id_rsa`是私钥，`id_rsa.pub`是公钥。

> b. 生成一个github用的SSH-Key
`$ ssh-keygen -t rsa -C "yourPersonal_email"`
如果是多个git账户，需要注意的是：出现提示输入文件名的时候(`Enter file in which to save the key (~/.ssh/id_rsa): id_rsa_github`)要输入与默认配置不一样的文件名，比如：我这里填的是 `id_rsa_github`，另一个是 `id_rsa`默认的,这个默认的代表是公司。

> c. 查看生成的ssh keys：`$ open ~/.ssh` 或者 `$ code ~/.ssh` 

> d. 此时，.ssh目录下应该有4个文件：id_rsa和id_rsa.pub，id_rsa_github和id_rsa_github.pub，分别将他们的公钥文件（id_rsa.pub，id_rsa_github.pub）内容配置到对应的code仓库上

![img](https://img-blog.csdnimg.cn/20181127170638531.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2FjbV93bG4=,size_16,color_FFFFFF,t_70)


## 3、添加并识别新的SSH keys私钥
<b>因为默认只读取`id_rsa`，为了让SSH识别新的私钥，需将其添加到SSH agent中 
命令：</b>
- start the ssh-agent in the background
> a. `$ eval $(ssh-agent -s)` 
  <font color=#0099ff>Agent pid 59566</font>  

> b. `$ ssh-agent bash`  

> c. `$ ssh-add ~/.ssh/id_rsa_github`  

> d. `$ ssh-add ~/.ssh/id_rsa`  

<b>比如:需要分别添加`id_rsa_github`和`id_rsa`。特别注意，如果后边出行权限问题：Permission denied（Publickey),很可能是私钥没有导入ssh-agent中</b>

## 4、添加新的SSH keys到Git账号的SSH设置中
将新生成的公钥`id_rsa_*.pub`添加到你的另一个github帐号(或者公司的gitlab)下的SSH Key中。 

## 5、配置~/.ssh/config文件（坑就在这，欲哭无泪，整了好久）
创建config文件，如果没有的话
`$ touch ~/.ssh/config  `      # 创建config文件
配置config文件信息

![img text](https://img-blog.csdnimg.cn/20181127172702402.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2FjbV93bG4=,size_16,color_FFFFFF,t_70)
# config
```
#gitlab
Host gitlab地址 
Hostname gitlab地址  
User 公司注册的邮箱  
IdentityFile /Users/xxx/.ssh/id_rsa_company

#github
Host github.com  
Hostname ssh.github.com  
User github注册的邮箱  
IdentityFile /Users/xxx/.ssh/id_rsa_personage
  
```

## 6、验证连接Git
连接git命令：

`$ ssh -T git@github.com`  

Hi you user name! You've successfully authenticated, but GitHub does not provide shell access.  

 上面是github的成功返回语句

 有可能不成功：
 <b>坑就在这个config文件里</b>

- 刚开始我也是网上百度把config文件里的内容拷贝到我自己新创建的config文件里，此时我觉的key配置好了，私钥也加成功了，应该没问题了，于是我开始测试是否已经有github的权限了，执行了
![img](https://img-blog.csdnimg.cn/20181127173011195.png)
- 但是结果不是我想看到的，ssh连不通，我开始百度谷歌，尝试各种方案，还是不可以。但是好奇新又让我去知道为什么，所以各种尝试，包括重新生成ssh key等等，后来看到这篇文章https://www.jianshu.com/p/83fbb1828453
- 又各种尝试：比如在config里加如下内容

![text](https://img-blog.csdnimg.cn/20181127173345388.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2FjbV93bG4=,size_16,color_FFFFFF,t_70)

- 再次执行
![img](https://img-blog.csdnimg.cn/20181127173606632.png)

- 如果只添加Port 443 然后会出现
![img](https://img-blog.csdnimg.cn/20181127173756555.png)

- 如果箭头处都添加又会出现
![img](https://img-blog.csdnimg.cn/20181127174244310.png)

- 又根据这个错误去google了一把，发现是语法错误，于是又开始到config文件里去检查语法，可是找来找去怎么都找不出来语法问题，就各种试。。。。。。此时有个念头就是重新格式化（有可能mac的回车和之前从网上拷贝不一样）然后再次试

- ssh -T git@github.com，哇塞，成功了！！！这个错误。。。希望记录下来以后可以回头翻翻，也提供给像我一样遇到这个坑的人，以免再次走弯路。

## 7.使用git仓库的时候使用ssh地址：git@github.com:acmwln/timeFormat.git 
每次都需要输入用户名和密码是因为你采用的是 https 方式提交代码，第一次操作就遇到这个问题，怎么push都需要输入用户名和密码，于是百度得知需要使用ssh协议就不需要; 如果采用的是 ssh 方式只需要在版本库中添加用户的rsa 的key就可以实现提交时无需输入用户名和密码。

- 如果你的版本库已经用https 方式创建好了，那么就需要先删除原来的提交方式。在终端执行以下指令：  

1. `$ git remote rm origin`
2. `$ git remote add origin https://github.com/:(用户名)/版本库名`  eg:`https://github.com/acmwln/timeFormat.git`

- 然后这个时候你使用下面指令提交代码：  

1.`git push -u origin master`

- 此时系统会提示你没有权限balabala的，此时你需要在本地创建自己的SSH KEY

1.`ssh-keygen -t rsa -C "用户名"`


下面附上一个超链接---[git 配置多个SSH-Key](https://blog.csdn.net/dqchouyang/article/details/54898910)

## 每次在github上创建一个新的仓库，就需要重新加入私钥

1. `$ ssh-add ~/.ssh/id_rsa_github`

2. `git remote add origin git@github.com:(用户名)/版本库名`  

3. `git remote -v `     [^_^]:查看远程连接库

4. `git push -u origin master`

....同上