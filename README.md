# repo_usage

通过下面实验了解android SDK源码是如何通过repo来管理的

## 搭建repo服务器

假设服务器IP : 192.168.1.100

### SSH KEY配置

客户端配置ssh key后将~/.ssh/id_rsa.pub里的内容拷贝到服务器的~/.ssh/authorized_keys

	client $ ssh -t rsa

### MANIFEST

在服务器端为项目创建manifest.git(假设目录是/android_sdk)

	server $ mkdir android_sdk
	server $ cd android_sdk
	server $ git init --bare manifest.git

在客户端向服务端的manifest中添加default.xml

	client $ git clone ssh://192.168.1.100/android_sdk/manifest.git
	client $ cd manifest.git

添加default.xml内容如下
```xml
<?xml version="1.0" encoding="UTF-8"?>
<manifest>
	<remote name="origin" fetch=".." />
	<default revision="master" remote="origin" />
	<project name="android_sdk/kernel" path="kernel" />
</manifest>
```

该xml文件表示这个sdk里有一个名为kernel的项目

在客户端将default.xml更新到服务器上

	client $ git add default.xml
	client $ git commit -m 'add default xml file'
	client $ git push

在服务器上建立default.xml里的项目(android_sdk目录下)

	server $ git init --bare kernel

### 第一个客户端下载代码

在客户端初始化repo

	client $ repo init -u ssh://192.168.1.100/android_sdk/manifest.git

上面的命令有异常的话尝试使用清华大学的镜像站点

	client $ repo init -u ssh://192.168.1.100/android_sdk/manifest.git --repo-url https://mirrors.tuna.tsinghua.edu.cn/git/git-repo

第一次使用时由于server端的git库是空的,没有分支信息,可以将每个git库clone下来,push一个提交到server

	client $ git clone ssh://192.168.1.100/android_sdk/kernel
	client $ cd kernel
	client $ touch README
	client $ git add .
	client $ git commit -m 'init'
	client $ git push

## 客户端下载代码

配置好后其它客户端操作流程

下载manifest

	new_client $ git clone ssh://192.168.1.100/android_sdk/manifest.git

初始化repo

	new_client $ repo init -u ssh://192.168.1.100/android_sdk/manifest.git --repo-url https://mirrors.tuna.tsinghua.edu.cn/git/git-repo

同步代码

	new_client $ repo sync

## 客户端修改代码提交流程

repo sync后的各个项目都是处在detached的状态的

工作流程是先创建分支,在分支上修改,后提交代码,删除分支

假设创建一个名位experimental的分支

	client $ repo start experimental --all
	假设这里修改kernel里的代码
	client $ cd kernel
	client $ git add .
	client $ git commit -m 'experiment fix bug'
	client $ git push origin experimental:master
