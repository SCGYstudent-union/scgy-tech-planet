# git 快速入门教程

*注：本教程供SCGY技术部同学参考，用于快速熟悉需要使用的git技能。*

author： loliw@SCGYTech

## 一、什么是git

git是一个开源的版本控制系统，无论是小到自己的c语言上机作业，大到各种项目都可以用git进行方便的版本控制。

关于git如何实现版本控制的原理，我暂且不细讲，有兴趣的同学请看[官方文档](https://git-scm.com/book/en/v2/Getting-Started-About-Version-Control)  (不要害怕看不懂英文文档，开个词典在边上查生词，重要的是理解内容)

总之，git使用SHA-1生成每个版本的版本号（长这样：463dc4f），只要你对文件做一个小小的改动，新的版本号就会和原来的截然不同，也无需担心新的版本号和以前的某个版本相撞，保证了版本号和版本一一对应。

## 二、git有什么用？

e.g.我的c语言程序

误删？误改文件？没关系，放弃修改回到上一个版本甚至更早的版本，文件的状态都可以恢复。

e.g.多人协作

A和B一起写网页，A和B同时修改了网页，那么是A发给B,B重新改，还是B发给A,A 重新改呢？

有了git的智能合并和一个远程仓库（例如github，gitee，gitlab……）同步完全不是问题：

## 三、怎么用git

### 准备工作

#### 1.安装

- Windows 系统：从 https://git-scm.com/downloads 获取 Git for Windows，按默认选项安装即可
- macOS 系统：从 Homebrew 安装
- Linux：从默认软件源安装即可（CentOS 7 建议从 EPEL 安装）                  
  - Debian / Ubuntu: `sudo apt-get install git`
  - CentOS 7: `sudo yum install epel-repo; sudo yum install git`
  - CentOS 8 / Fedora: `sudo dnf install git`

#### 2.账号注册

需要一个远程仓库，建议注册 [GitHub](https://github.com/) 账号

#### 3.命令行

mac/linux下，打开terminal，cd进入目标文件夹即可。

windows下不熟悉命令行的同学可以先打开目标文件夹，空白处右击打开Git Bash Here

![1570868659606](.\material_git\git_bash.png)

#### 4.初始配置

``` bash
git config --global user.name "your name" 
git config --global user.email "your_email@example.com"
```

p.s.不小心打错了重输一遍对的就好了

### (1)本地仓库

#### 新建仓库：

``` bash
mkdir my_repo	//新建文件夹
cd my_repo		//进入文件夹
git init		//在这个文件夹里新建仓库,仓库名就是my_repo
```

#### 提交新版本（commit）：

在仓库文件夹中进行一些添加、修改、删除操作之后，都需要git add

``` bash
//在仓库文件夹里加入你的helloworld.c之类的文件
git add helloworld.c
//如再进行其他修改,还需要git add
git add yourfile
//这个操作可以将当前文件夹下所有修改全部add 适合懒惰的我:D
git add .
```

commit 之前需要将所有修改用git add添加

这个命令可以查看当前状态，很常用


```bash
git status
```

显示出的红色文件为修改但是未add，绿色文件为add但是还没commit

这个命令可以查看即将提交的修改内容

```bash 
git diff helloworld.c
```

确认所有修改都add之后，输入：

``` bash
git commit -m "your comment"
```


将会产生一个新的提交（commit），其中 your comment 建议填写有意义注释，如”add ReadMe","change font size","fix color bugs"等

#### 版本回退

查看提交日志：

```bash
git log
```

退回提交：

```bash
git reset --hard HEAD~  # 退回一次提交
git reset --hard HEAD~n	# 退回n次提交
git reset --hard 7F23a0	# 推回指定版本号 (至少输入前四位)
```

#### 撤销修改

`git checkout -- helloworld.c` 可以让文件回到上一次提交时状态

`git reset helloworld.c` 撤销 `git add` 的作用

### (2)远程仓库

#### 添加SSH密钥

之前config的时候只输入了username和email，那么远程仓库（github）怎么知道是你呢？显然需要一个密码，但是每次都输入密码太麻烦了，于是我们使用ssh密钥。*如果想知道SSH密钥为什么可以代替输入密码，自行使用搜索引擎:D*

##### 第一步：生成一对 SSH 密钥

输入：

```bash
ssh-keygen -C your_email@example.com
```

第一次生成ssh密钥的话，建议3次回车，不必设置密码。

##### 第二步：找到密钥

查看第一次回车前屏幕上打印的信息,找到`id_rsa.pub`的位置，默认通常是 “*你的个人文件夹*/.ssh/id_rsa.pub"

 用记事本打开`id_rsa.pub`，复制里面内容:以 `ssh-rsa` 开头，你的邮箱结尾的内容

##### 第三步：在Github上添加密钥

登陆 GitHub，点击右上角自己头像，打开 Account settings → SSH Keys 页面，点 Add SSH Key

 Title随意填写，方便自己辨认即可。在 Key 文本框里粘贴 `id_rsa.pub `的内容 

##### 第四步：确认 SSH 密钥添加成功

输入：

```bash
ssh -T git@github.com
```

如果看到你的用户名提示认证成功，那么ssh key就配置完成了。

#### 添加远程库

##### 1.新建自己的仓库

登陆 GitHub，左上角点 New repository，仓库名填 `my_repo`（填自己的仓库名，与之前保持一致即可），其他默认，点击创建完成。

找到Clone or download 的绿色按钮，选到Use SSH，点击地址右侧按钮即可复制项目地址

![1570874177412](.\material_git\clone_or_download.png)

输入以下命令：

```
$ git remote add origin [你复制的地址]
```

即可添加名为”origin“的远程仓库地址

删除地址的操作：`git remote remove origin`

查看当前地址列表的操作：`git remote -v`

 push本地内容到远程库：  

- 初次推送：`git push -u origin master`
- 后面推送就简单了：`git push` ，（origin master 为默认）

将远程库同步到本地：`git pull`

##### 2.向已有项目贡献代码

在进行多人合作的时候，假如每个人都有直接向远程仓库写的权限，那么在管理的时候会非常不方便，更不要说还会有可能的安全问题 (删库跑路？) 。

那么在没有直接写的权限时改如何贡献代码呢？

以少院学生会网站的repo为例:

###### 1）fork仓库

登陆Github打开[website-of-SCGY](https://github.com/SCGYstudent-union/website-of-SCGY)，点击fork

![img](.\material_git\fork.png) 

然后你会获得一份一模一样内容的仓库，但是owner是你自己。显然这个仓库你有写入的权限，将这个仓库作为origin

###### 2）clone远程仓库

首先，在你想要复制原始仓库的地方打开命令行，输入

```bash
git clone [复制fork之后的项目地址]//请在网络环境良好的地方进行，否则会clone失败或耗时很久
```

同时该项目地址会被自动添加为origin

```bash
cd website-of-SCGY
```

在git bash中可以看到蓝色的`master`字样

回到[website-of-SCGY](https://github.com/SCGYstudent-union/website-of-SCGY)，复制其项目地址，并用

```bash
git remote add upstream [fork之前的原始项目地址]//upstream可改成自己取的名
```

###### 3）push

进行了新的commit之后，运行`git push origin master`

再登陆Github，设法发送pull requests

![1570887096175](.\material_git\pull_request.png)

联系原始仓库的管理员进行合并（merge），新的commit就进了原始仓库

4）pull

输入`git pull upstream master`即可

## 四、尾声

本教程是为SCGY技术部同学设计的，包含了很有可能用到的的git相关知识。关于git更加详尽的介绍请移步[Pro Git book](https://git-scm.com/book/en/v2)或其他教程。关于Github的用途也绝不仅限于本教程所述，敬请自行探索：[Github Guide](https://guides.github.com/)