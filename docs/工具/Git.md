# 简介
> Git 是⼀种分布式版本控制系统，它可以不受⽹络连接的限制，加上其它众多优点，⽬前已经成为程序开发⼈员做项⽬版本管理
时的⾸选，⾮开发⼈员也可以⽤ Git 来做⾃⼰的⽂档版本管理⼯具。
# 使用
## 步骤
1. 下载安装包安装
2. 进入工作文件夹从github上克隆项目 | git init初始化本地仓库

## 常用操作
### git clone
> 从服务器拉取代码

``` bash
git clone https://github.com/yks43/yks-blog.git
```

### git config
> 配置开发者用户名和邮箱
``` git
git config user.name yks43
git config user.email 2714277030@qq.com
```
每次代码提交的时候都会⽣成⼀条提交记录，其中会包含当前配置的用户名和邮箱。

### git branch
> 创建、重命名、查看、删除项⽬分⽀，通过 Git 做项⽬开发时，⼀般都是在开发分⽀中进⾏，开发完成后合并分⽀到主干。

```git
git branch daily/0.0.0
```
创建⼀个名为 daily/0.0.0 的⽇常开发分⽀，分⽀名只要不包括特殊字符即可。
```git
git branch -m daily/0.0.0 daily/0.0.1
```
如果觉得之前的分⽀名不合适，可以为新建的分⽀重命名，重命名分⽀名为 daily/0.0.1
```git
git branch
```
通过不带参数的branch命令可以查看当前项⽬分⽀列表
```git
git branch -d daily/0.0.1
```
如果分⽀已经完成使命则可以通过 -d 参数将分⽀删除
### git checkout
> 切换分支
```git
git checkout daily/0.0.1
```
切换到daily/0.0.1分支，后续的操作将在这个分支上进行
### git status
> 查看文件变动状态
```git
git status
```
### git add
> 添加文件到暂存区
```git
git add REAMDE.md
```
通过指定⽂件名 README.md 可以将该⽂件添加到暂存区，如果想添加所有⽂件可⽤ git add . 命令，这时候可通过 git status 看
到⽂件当前状态 Changes to be committed:（⽂件已提交到暂存区）
### git commit
> 提交文件变动到版本库
```git
git commit -m '提交原因'
```
通过 -m 参数可直接在命令⾏⾥输⼊提交描述⽂本
### git push
> 将本地代码改动推送到服务器
```bash
git push origin daily/0.0.1
```
origin 指代的是当前的git服务器地址，这⾏命令的意思是把 daily/0.0.1 分⽀推送到服务器
### git pull
> 将服务器上最新的代码拉取到本地
```bash
git pull origin daily/0.0.1
```
如果其它项⽬成员对项⽬做了改动并推送到服务器，我们需要将最新的改动更新到本地，这⾥我们来模拟⼀下这种情况。
进⼊Github⽹站的项⽬⾸⻚，再进⼊ daily/0.0.1 分⽀，在线对 README.md ⽂件做⼀些修改并保存，然后在命令中执⾏以上命
令，它将把刚才在线修改的部分拉取到本地，⽤编辑器打开 README.md ，你会发现⽂件已经跟线上的内容同步了。
如果线上代码做了变动，⽽你本地的代码也有变动，拉取的代码就有可能会跟你本地的改动冲突，⼀般情况下 Git 会⾃动处理这
种冲突合并，但如果改动的是同⼀⾏，那就需要⼿动来合并代码，编辑⽂件，保存最新的改动，再通过 git add . 和 git commit -
m 'xxx' 来提交合并。
### git log
> 查看版本提交记录
```bash
git log 
```
查看整个项⽬的版本提交记录，它⾥⾯包含了提交⼈、⽇期、提交原因等信息；
提交记录可能会⾮常多，按 J 键往下翻，按 K 键往上翻，按 Q 键退出查看
### git tag
> 为项目标记里程碑
```bash
git tag publish/0.0.1
git push origin publish/0.0.1
```
当我们完成某个功能需求准备发布上线时，应该将此次完整的项⽬代码做个标记，并将这个标记好的版本发布到线上，这⾥我们以 publish/0.0.1 为标记名并发布，
### .gitignore
> 设置哪些内容不需要推送到服务器，这是一个配置文件
.gitignore 不是 Git 命令，⽽在项⽬中的⼀个⽂件，通过设置 .gitignore 的内容告诉 Git 哪些⽂件应该被忽略不需要推送到服务
器，通过以上命令可以创建⼀个 .gitignore ⽂件，并在编辑器中打开⽂件，每⼀⾏代表⼀个要忽略的⽂件或⽬录，如：
```bash
demo.html
build/
```
以上内容的意思是 Git 将忽略 demo.html ⽂件 和build/ ⽬录，这些内容不会被推送到服务器上


# doctify
文档：https://docsify.js.org/#/zh-cn/quickstart

第一次执行
``` bash
git init
git add .
git commit -m "first-commmit"
git remote add origin https://github.com/yks43/yks-blog.git
git push origin master
```

更新
``` bash
git add .
git commit -m "se-commmit"
git push origin master
```
# bug
## Failed to connect to 127.0.0.1 port 1080: Connection refused

查询是否使用代理：
git config --global http.proxy
取消代理：
git config --global --unset http.proxy

# 公司使用
## 经验
### 前期准备
> 配置好本机git，拿到公司分配的gitlab账号，配置好ssh key，配置好maven和公司的setting文件，把对应的项目拉下来，问清楚启动流程以及需要修改的参数

正常来说，一个项目有一个master分支，但是大多数会分出很多分支，一个test分支用于测试环境，其他各个小组可能根据需求分了很多分支。作为一个小组成员，我所在的分支比如说是sky，那么我在进行日常开发时
1. 切换到sky，pull下最新代码
2. 本地coding，需求写一个commit一个
3. commit完，再pull一下最新代码，如果在你coding的这段时间有其他同学更改了同一份文件，那么它就会提示你代码有冲突，需要你手动去合并一下，idea合并冲突很简单，正常来讲，分三列，左边是你本地的，中间是合并后的最终成果，右边是刚pull下来的代码，手动根据指引把你写的和和最新的合在一起就行了
4. 合并好，直接push到远程仓库
5. 然后再切换到test分支，pull下最新代码
6. 再合并sky分支，有冲突继续解决，然后直接push就可以了，代码就提到test分支了
7. 后续就是部署，测试，该bug，部署，测试，验收，发布生产了
8. 生产上出bug，直接拉一个新的分支hotfix1，修复bug，提交测试，然后直接把hotfix1合到master(生产)部署上线