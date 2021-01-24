
@[TOC](git )
# git 再次学习

>之前几个月学习过git当时事情比较多，未重视，想着先用起来，现在好久没有使用，之后要进行一些自己的项目开发迭代，又想重新学习，现在感觉但是git工具用的并不很懂，现重新学习

## 引言
git是一个不同于其他集中式的版本管理工具，git采用的是一种分布式的版本管理工具

每个客户端同时又是一台服务器保证项目安全，内置的压缩算法解决了不想只保存快照，还保存大量数据镜像的空间问题

总之：  
git每次commit都是整个项目的快照，可在本地，数据安全

集中式，只保存修改的快照，在一个总服务器，服务器宕机，玩完

## 目前使用的git 版本

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210117105726143.png)

## git 目录组成

![在这里插入图片描述](https://img-blog.csdnimg.cn/2021011816103058.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDU5NDU2NA==,size_16,color_FFFFFF,t_70)

## git的内部数据管理

传统理解：~~**工作区 --> 暂存区 --> 版本库**~~ 

实际上在工作区到版本库这个地方，是工作区先到了版本库然后被拉到了暂存区

实际：**工作区 --> 版本库 --> 暂存区-->版本库**

## 底层命令

有三个对象需要注意：git对象、树对象、提交对象

### 工作区到版本库

#### git hash-object 文件名

git init命令将文件夹变为了一个**工作区**

生成了一个.git 的隐藏文件夹

里面的object放着的就是**版本库**

git用的对象（**git对象**（blob））来存文件，是一个blob的键值对   
键是一个生成的hash值  
值则为文件内容

那如何验证上述内容

 1. 首先创建一个文件写入“jiao”
 2. 将这个文件用hash编码 其中-w是保存的意思
 3. 查看是否保存成功了
    
    会用hash的前两个字符作为一个文件夹其他字符作为其中的文件名，这一整个hash值就是一个键
    
 4. 看值  
    cat被形象的理解为猫一眼  
    -p 是根据编码 给出一个理想的形式，方便用户看

```shell
echo “jiao” > test.txt
git hash-objext -w test.txt
find ./.git/oject -type f
git cat-file -p 哈希值
```

当然修改test.txt这个文件，并用hash保存后（不保存object是不会有这个文件的，方便理解工作区与暂存区的概念），并不会修改原本的那个hash文件，而是又新建了一个hash文件出来，这样可以版本控制回滚

这里的键值对，没有真正的文件名  
在Git中，文件名并没有被保存——我们仅保存了文件的内容

解决方案：树对象  
**一个git对象代表一个文件 不能代表一个版本**

### 暂存区
#### git update-index --add --cacheinfo 100644 哈希值 test.txt与git write-tree

**树对象**

树对象中的每一个对象对应一次版本的快照

树对象是对整个项目的管理，如**管理项目名**

操作：  
 利用update index 命令 为 test.txt 文件的首个版本——创建一个暂存区。  
并通过write tree 命令 生成树对像。  
查看生成对象类型

命令：

```shell
git update-index --add --cacheinfo 100644 哈希值 test.txt
git write-tree
```

>文件模式  
 100644 ，表明这是一个普通文件  
100755，表示一个可执行文件  
120000 ，表示一个符号链接 。  
--add 选项  
因为此前该文件并不在暂存区中  
***首次需要 add***  
--cacheinfo 选项  
因为将要添加的文件位于  
Git 数据库中，而不是位于当前  
目录下 所有需要 cacheinfo

这时.git目录下会增加一个**index暂存区文件**

 1.将文件直接放到版本库
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210116174505381.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDU5NDU2NA==,size_16,color_FFFFFF,t_70)

2.从版本库拉取到暂存区，并写入树对象中

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210116174601341.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDU5NDU2NA==,size_16,color_FFFFFF,t_70)

这时，这个项目已经有了第一个版本


3.看新加的树对象的类型  （-t用来查看类型）

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210116174708521.png)

4.查看暂存区

![在这里插入图片描述](https://img-blog.csdnimg.cn/2021011617474570.png)

5.修改test.txt 直接放到版本库并拉到暂存区

查看暂存区，可以发现test的hash已经被替换

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210116175210718.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDU5NDU2NA==,size_16,color_FFFFFF,t_70)

修改文件重新拉到暂存区可以不加--add， 这里是之前复制上面的所以加了

6.添加一个new.txt文件, 直接放到版本库并拉到暂存区

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210116175343412.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210116175415752.png)

发现这两步比较繁琐可以合为一步

```shell
echo 'jiao' > new.txt
git udpate-index --add new.txt
```

#### 注意点

---
这一步就是之后高级命令里面的**git add**
以上的内容也说明了实际是先**从工作区到版本库再到暂存区的**
也就是说即使只通过git add命令在暂存区记录其实也可以进行版本控制（回滚）

---

7.更新树对象

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210116175526760.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDU5NDU2NA==,size_16,color_FFFFFF,t_70)

### 提交对象

一个树对象也可以加入之前的树对象，作为一个git对象其中

新树的hash：c3350ef71d839d3f63d44b8278a31b1746106237

老树的hash：8c6cd6603120e3d10ff18e352b0f0a024eaa9f44

**将第一个树对象加入第二个树对象，使其成为新的树对象**


![在这里插入图片描述](https://img-blog.csdnimg.cn/20210116182405239.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDU5NDU2NA==,size_16,color_FFFFFF,t_70)

目前形成这样的结构


![在这里插入图片描述](https://img-blog.csdnimg.cn/20210116183328661.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDU5NDU2NA==,size_16,color_FFFFFF,t_70)

现在有三个树对象 （执行了三次 write tree ）），分别代表了我们想要跟踪的不同项目快照。然而问题依旧：若想重用这些快照，你 必须记住所有三个哈希值。 并且，你也完全不知道是谁保存了这些快照，在什么时刻保存的，以及为什么保存这些快照。 而以上这些，正是提交对象（ commit object）能为你保存的基本信息


所以总之，提交对象就是一个树对象+描述信息

提交对象的使用

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210116214214805.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDU5NDU2NA==,size_16,color_FFFFFF,t_70)

>由图可以看出这个提交对象是有一个tree对象和一个作者，提交者以及描述信息组成

这里使用的是第一个树对象，所以不需要指定父对象

第二树的hash：**c3350ef71d839d3f63d44b8278a31b1746106237**

第一树的hash：**8c6cd6603120e3d10ff18e352b0f0a024eaa9f44**

但是之后的树对象是要指定父对象的

注意这里指的父对象是一个提交对象
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021011621540758.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDU5NDU2NA==,size_16,color_FFFFFF,t_70)

## 高层命令

### git add +文件名或路径

是底层命令  
 git hash-object -w 文件名  
 git update-index --add --cacheinfo hash值 文件名  
 的组合  
 实现了文件先到版本库又被拉回暂存区（原本意义上的 工作区到暂存区）

### git commit -m ""

是底层命令  
git write-tree   
 git commit-tree  
 的结合
 
暂存区到版本库，代表了一次版本的正式提交上线
### git status
查看文件的状态

### 工作区的状态

分为已跟踪和未跟踪，已经add的文件就是以跟踪状态，反之...

1. 新建一个test.txt文件，放到暂存区![在这里插入图片描述](https://img-blog.csdnimg.cn/20210117100920555.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDU5NDU2NA==,size_16,color_FFFFFF,t_70)

>有untracked files ，有未跟踪的文件，然后add 到暂存区

3. 修改一下test文件，查看状态![在这里插入图片描述](https://img-blog.csdnimg.cn/20210117101006212.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDU5NDU2NA==,size_16,color_FFFFFF,t_70)

>发现修改后的文件 not staged ,没有放到暂存区

5. 将文件add到暂存区![在这里插入图片描述](https://img-blog.csdnimg.cn/2021011710120316.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDU5NDU2NA==,size_16,color_FFFFFF,t_70)

### git diff 和 git diff --cached

**已跟踪未暂存的更新和查看已暂存**

>实际上 git status 的显示比较简单，仅仅是列出了修改过的文件，如果要
查看具体修改了什么地方，可以用 git diff 命令 这个命令 它已经能 解决 我们
两个问题了： **当前做的哪些更新还没有暂存？有哪些更新已经暂存起来准备好了下次提交？**


1. 当前做的哪些更新还没有暂存？，  
命令： git diff 不加参数直接输入 git diff  
2. 有哪些更新已经暂存起来准备好了下次提交？  
命令： git diff cached 或者 git diff staged(1.6.1 以上  

紧跟上文截图

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210117104652378.png)

>目前所有的文件都处于跟踪，所以并未有未暂存的文件

**注意：** **没有跟踪的文件，使用git diff 也是不到的，只能看到一些跟踪过又修改的还为暂存的文件**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210117104753474.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDU5NDU2NA==,size_16,color_FFFFFF,t_70)

>由于目前还都没有提交过，所以git diff --cached 的test.txt文件都是新+的

接下来提交项目

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210117105042832.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDU5NDU2NA==,size_16,color_FFFFFF,t_70)

可以看到提交项目后，暂存区和未暂存区修改的内容清空	

修改test.txt文件，用git diff 看为暂存修改的内容

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210117105448708.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDU5NDU2NA==,size_16,color_FFFFFF,t_70)

此时，暂存区修改无修改的内容

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210117105828522.png)

可以看到提交项目后，暂存区和未暂存区修改的内容清空，但是暂存区其实没有清空是有一个git对象类型的文件
	

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210117111701937.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDU5NDU2NA==,size_16,color_FFFFFF,t_70)

>因为此时还没add新修改的文件，所以暂存区存放的还是之前提交的文件

add后

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210117112011168.png)

>可以看到hash值改变

查看暂存区是否有修改

![在这里插入图片描述](https://img-blog.csdnimg.cn/2021011711210591.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDU5NDU2NA==,size_16,color_FFFFFF,t_70)

---
还没有提交会显示，changes to be commited ,准备去提交的文件
提交过后，noting to commit

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210117112329136.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDU5NDU2NA==,size_16,color_FFFFFF,t_70)

**提交时也可直接 git commit 不加 -m 会进入到vim编辑器，编辑描述信息**

### 跳过使用暂存区

也可以直接从工作区直接建立一个新的版本出来一步到位

直接`git commit -a -m`


**注意**：这个命令只能提交哪些，跟踪过的文件，如修改后的文件，未跟踪的文件，不能用

>文件修改后的状态依然是跟踪的，不过是未暂存（这也许就是跟踪和暂存的区别吧）

**代码证明：**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210118163302870.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDU5NDU2NA==,size_16,color_FFFFFF,t_70)

### git rm

git 给的删除操作

#### 注意理解

git rm 是删除在工作区的文件，提交后，但是在版本库objects中相当于是做了一个新增操作，新增了一个提交后的删除文件后的一个快照，如果版本库真啥都没了，你还版本控制个屁

写入一个新文件new.txt，并提交  
删除一个test文件，加入一个demo文件不add  
删除demo 

demo就像是人间白来了一场，任何痕迹没留下

***

>注意:在删除文件时，因为在文件git add被跟踪过其实是从版本库拉到暂存区的，这时版本库是有备份的，但是你在修改过一个文件时，因为没有提交，所以版本库是没有备份的，这时，如果修改后执行删除命令，会报错。可以采用 `git rm --cached 文件名`，这可以将文件的原有暂存区给删了，但是工作区不删，`git rm -f 文件名`，暂存区，工作区都会删除

![9](https://img-blog.csdnimg.cn/20210117120325618.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDU5NDU2NA==,size_16,color_FFFFFF,t_70)

>注意删除test.txt 文件实际执行了两个步骤，先rm 文件，有add 了一下目录

### 修改文件名

git mv 源文件名 新文件名

git mv 实际上执行了三个命令:

>mv 源文件名 新文件名  
git rm  源文件名  
git add 新文件名

验证如下

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210118154311889.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDU5NDU2NA==,size_16,color_FFFFFF,t_70)

>先用基础命令，mv一个文件，git status一下状态，可以发现mv指令删除了文件，创建了一个新文件，且新文件未被跟踪

接着尝试git mv

![在这里插入图片描述](https://img-blog.csdnimg.cn/2021011815434938.png)

>一步到位

### 查看一个项目日志

 三个命令
 
>git log  
git log --pretty=oneline   
git log --oneline  
常用第三个显示简洁信息

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210118155014345.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDU5NDU2NA==,size_16,color_FFFFFF,t_70)

### git reflog 

这个命令，**记录着head的移动**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210121155027606.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDU5NDU2NA==,size_16,color_FFFFFF,t_70)

**按 q 退出**

## 分支（branch）

分支相当于一个指针

一直指向最新的提交对象

之前所有的分支都是master

实际上通过上图可以看出，head->master

### 形象理解

>head 是一个控制的指针，master相当于一个用户，这个用户是默认创建的，可以当做一个树的主干，其他分支就是可以由它向外开发，当其他分支安全稳定了，就可以向这个主干，输送养分了，甚至还可以它这个分支变成主干

### 代码理解

1.新建分支，并切换到这个分支

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210118162548352.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDU5NDU2NA==,size_16,color_FFFFFF,t_70)

>新建分支，再切换到那个分支，这两步可以合为一步  
>`git checkout -b 新分支名`

2.在该分支上新建一个文件

![在这里插入图片描述](https://img-blog.csdnimg.cn/2021011816262715.png)

3.提交

![在这里插入图片描述](https://img-blog.csdnimg.cn/2021011816264593.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDU5NDU2NA==,size_16,color_FFFFFF,t_70)

>可以看到jiao指针（分支）前进了。而master指针（主干）并没有前进


### 显示分支列表

git branch

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210118163635363.png)

### 删除分支

**注意：** 不能自己删自己

所以要删除分支，需要先前切换到其他分支

git branch -d 分支名

注意：会有提示，没有与主干合并is not fully merged

需要 git branch -D 分支名 // 强制删除

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210118231735680.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDU5NDU2NA==,size_16,color_FFFFFF,t_70)

### 查看项目分叉历史

```
git log --oneline --decorate --graph --all
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210118231809892.png)

这个太长了

####  别名

可以起**别名**

$ `git config --global alias.co checkout`

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210118232055573.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDU5NDU2NA==,size_16,color_FFFFFF,t_70)

可以看到起到了同样的效果

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210118232411296.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDU5NDU2NA==,size_16,color_FFFFFF,t_70)

### 查看每一个分支最新一次的提交

git branch -v

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210118232533357.png)

<span id="版本回溯"></span>

### 版本回溯

为不影响主干，新建分支回溯

加入需要回溯的hash值，通过提交的版本左侧可得

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210118233812395.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDU5NDU2NA==,size_16,color_FFFFFF,t_70)

>这里我新建分支yue，回溯到test.txt删除之前，可见回溯成功  
>**有个注意的地方：记得切换到新建的分支上去**  
>还有一个大地方，会发现，在**切换分支时，工作区的目录变了**  
>所以在**每次需要切换分支时，最好先提交一下！！**

有一个场景，在一个分支中，存在一个未跟踪的文件，没有add，这时切换到主分支，为了安全，这个文件不会随分支切换工作区的更改而更改，也就是文件会出现在主干中，这个是合理的，但是是不愿看到的，所以先提交没毛病

### 合并

合并是应该从主干来吸纳分支

```
git merge 分支名
```

#### 查看合并到当前分支的分支列表

```
git branch --merged
```

#### 查看没有合并到当前分支的分支列表

```
git branch --no-merged
```

 ##  目前为止的实际案例
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20210119005903165.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDU5NDU2NA==,size_16,color_FFFFFF,t_70)
 
### 实现

1、在yun分支的基础上进行，新建了一个day.txt 完成50% 提交

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210119014250629.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDU5NDU2NA==,size_16,color_FFFFFF,t_70)

<span id="没有必要的提交">没有必要的提交</span>


>这里的提交其实是没有必要的，但是为了安全性，不得不提交所以在一下小结介绍一种存储方式，解决这个问题  
[git的临时存储栈](#git的临时存储栈)


2、在master新键一个分支hotbug，更改aa.txt 中的bug ,并提交合并

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210119014529258.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDU5NDU2NA==,size_16,color_FFFFFF,t_70)

>发现此时主干走在了分支前面，且出现了分叉，（注意是从下往上看）

3、接着继续自己的工作，提交

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210119015018284.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDU5NDU2NA==,size_16,color_FFFFFF,t_70)

4、之前的hotbug由于修改完bug 已经没有了，就可以删除了

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210119015148111.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDU5NDU2NA==,size_16,color_FFFFFF,t_70)

5、提交合并今天的工作

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210119015359170.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDU5NDU2NA==,size_16,color_FFFFFF,t_70)

<span id="分支合并冲突"></span>

#### 注意

>***可以发现，合并并不是很成功，是因为，之前主干走在了分支前面，且分支之后又修改了aa.txt*(主干也有)，这时，不知道你是否愿意接受aa.txt的更改，也许那些更改会影响你的功能实现，这时系统告诉你产生了冲突,让你自己抉择***
>同样可以注意：发生冲突后，分支的括号多了个 |merging(合并中...)

自己可以去修改一个冲突的那个文件

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210119015808112.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDU5NDU2NA==,size_16,color_FFFFFF,t_70)

这时vim编辑器中的内容，可以看到，git 给每个分支做的不同的修改，给分开显示用=====隔开了

自己编辑好之后，add 然后commit

这时主干的所有文件就正式上线了

查看一下

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210119020228618.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDU5NDU2NA==,size_16,color_FFFFFF,t_70)

### git 的临时存储栈

<span id="git的临时存储栈">git的临时存储栈</span>

问题：  
[工作进行到一半问题，需要进行其他分支的现分支保存问题](#没有必要的提交)

为了解决这个问题有一个git存储功能

![git stash](https://img-blog.csdnimg.cn/20210119112711508.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDU5NDU2NA==,size_16,color_FFFFFF,t_70)

一般建议一次只存放一个东西到栈内，不然要涉及到，栈的后进先出，先进后出问题

使用时使用 `git stash pop` 一步到位，**直接应用加删除**

**！！注意** ： 使用时，一定要**干完其他事情回到自己的分支去，在用这个命令**。不然会弄到主干，而且主干还不认这个文件，需要强制移出，而且用了这个命令，栈内该文件也没有了 ，就很麻烦

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210119141845743.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDU5NDU2NA==,size_16,color_FFFFFF,t_70)

可以看到，并不会产生提交记录

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210119142250348.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDU5NDU2NA==,size_16,color_FFFFFF,t_70)

>可以看到实际是提交了的，但是还原之后，不会有日志

## 后悔药

分为三块

>如果是在工作区跟踪都没跟踪过，直接要么直接rm 删了，要么在编辑器中撤销

### 工作区

也可以说是 **在暂存区已修改未暂存**  
查看`git status`自有解法

这步适用于，之前暂存过，现在修改了，在工作区也有了，但是不想再次提交到暂存区  

`git restore 文件名` 这个命令之前不是这样的原命令为 `git checkout -- 文件名 （--后面有空格）`,不同版本不一致

所以就看着`git status`来就行

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210121145925400.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDU5NDU2NA==,size_16,color_FFFFFF,t_70)

### 已经提交到暂存区

查看`git status`自有解法

这个适用于已经在暂存区，又不想让它在暂存区的了

>其实这个与 git rm --cached 文件名 有异曲同工之妙

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210121150247239.png)

### 已经提交到版本库

这步适用于那些提交时看错了，如有的修改，没有 git add 就提交了，导致新修改没有提交上，或者提交的时候，描述信息写错了

如果有新的修改没有add， 那就直接 `git add`

然后 `git commit --amend`，这时会跳到vim编辑器，让你修改描述信息， 这时就会覆盖上次，提交可以通过看日志，`git sab` (这个是别名)看到

## reset

我是这样理解的：**后悔药是有一些待解决的问题要回头，而reset是写错了要回头**

这两个的命令，有些还是异曲同工

如果我之前提交了，现在我不想要这个提交

我们可以这样做，因为head指向最新的提交对象，现在让head指向前一个提交对象

>注意这里的head移动时，是带着指针（分支），在同一个分支的不同版本一起移动，之前的checkout是head在不同的分支下切换，所以前者是**版本穿梭**，而后者是**分支穿梭**


### git reset --soft head~

1、修改yue.txt  
2、提交到版本库  
3、执行命令`git reset --soft head~`  
4、查看头指针位置  
5、查看版本日志
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021012218021321.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDU5NDU2NA==,size_16,color_FFFFFF,t_70)

>可以看到相当于版本撤销了，撤销了提交

6、查看暂存区，没变  
7、查看工作区，没变

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210122180716480.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDU5NDU2NA==,size_16,color_FFFFFF,t_70)

>总结：`git reset --soft head~` ,只是撤销了当前提交的版本，暂存区和工作区内容都不改变


### git reset [--mixed] head~

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210122184815754.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDU5NDU2NA==,size_16,color_FFFFFF,t_70)

>总结：`git reset  head~` 撤销了版本提交，同时把暂存区的本次的内容给清， 这个命令也是最常用得到了所以可以省略 --mixed

当然可以单独回退一个文件，毕竟一次加入了好多，一个都从暂存区删除了不至于

```
git reset 文件名
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210122225659349.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDU5NDU2NA==,size_16,color_FFFFFF,t_70)

### git reset --hard head~

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210122223110736.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDU5NDU2NA==,size_16,color_FFFFFF,t_70)

>总结：这个命令会使整个上一次提交到上上次提交之间的所有工作全部撤销，这个命令是git 会真正的销毁数据的仅有的几个操作之一，不能撤销


<span id="版本回退"></span>

## git reset --hard hash值

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210122231455749.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDU5NDU2NA==,size_16,color_FFFFFF,t_70)

这个几乎实现了**版本穿梭**  
当然需要回退可以用git branch 新分支 hash 之后在合并上去

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210122230046544.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDU5NDU2NA==,size_16,color_FFFFFF,t_70)

>最后扔个图，checkout 与 reset的区别

***

## 数据恢复

当使用版本回退后  
[版本回退](#版本回退)  
可以用git branch 新分支 hash 之后在合并上去  
之前有在 git branch 有讲 [版本回溯](#版本回溯)  
**也可通过git reflog获得hash**


##  打tag

>tag 与分支类似，但是不能动，一般就是用来表示版本号的

git tag tag名 //给现在的提交对象打tag

git tag tag名 hash //给之前特定的提交对象打tag

git tag //查看tag list

git checkout tag名 

>注意：这个可以切到tag那个版本，但是切过去分支指针不会跟着过去，也就是说，当前的head无指向，所以需要创建一个新的分支指向，同时也方便了之后的操作

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210122234307874.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDU5NDU2NA==,size_16,color_FFFFFF,t_70)

**由此 git本地操作完**

***

# 远程仓库

git config --list //查看一些本地配置

## 远程仓库别名

git remote -v // 查看远程仓库

git remote add 别名 地址

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210124150100632.png)

## git push [仓库别名]  \[分支]

推送到远程仓库

## git clone 地址

克隆远程仓库

新建一个文件夹打开，运行git 命令行，**直接运行 git clone 地址 不需要Git init**

之后会自动生成 .git 文件夹

>克隆之后，查看这个文件目录中的git remote -v 

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210124163915520.png)

>可以看到系统自动起了别名

**向远程仓库提交后，git会生成一个如图的远程跟踪分支**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210124165418680.png)

>在首次git push 时，会生成远程跟踪分支  
>远程跟踪分支是本地分支与远程仓库分支的媒介，从远程分支上拉东西下来，先传给远程跟踪分支  
>然后用远程跟踪分支与本地分支合并

## git fetch + 库别名

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210124195054995.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDU5NDU2NA==,size_16,color_FFFFFF,t_70)

>将我之前写的notes 拿下来，但是工作区目录并没有改变，因为现在还是在远程跟踪分支那里

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210124195347610.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDU5NDU2NA==,size_16,color_FFFFFF,t_70)

**创建一个分支与note/master对等的分支**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210124195930260.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDU5NDU2NA==,size_16,color_FFFFFF,t_70)

**用创建的分支，合并到现在的分支上去**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210124202357350.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDU5NDU2NA==,size_16,color_FFFFFF,t_70)

>由于这个是以前写的东西，放在了这个的历史里面，没有关联，需要加一个`--allow-unrelated-histories`

此时就给合并上来了

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210124202541826.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDU5NDU2NA==,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210124202843314.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDU5NDU2NA==,size_16,color_FFFFFF,t_70)

>可以看到此时的2021/master这个远程跟踪分支没有移动，**我们是移动不了远程跟踪分支的**，远程跟踪分支是根据git push的版本来确定，自动移动的


![在这里插入图片描述](https://img-blog.csdnimg.cn/20210124203706996.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDU5NDU2NA==,size_16,color_FFFFFF,t_70)

>可以看到确实移动了


## git pull

总结：我们可以看到，从远程合并到当前很麻烦，需要先给到远程跟踪分支（**git fetch**），自己再从远程跟踪分支上拿到，合并

很麻烦，我们可以直接 **git pull** 完成上面的步骤,从远程分支拉文件到本地分支

![在这里插入图片描述](https://img-blog.csdnimg.cn/202101242042091.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDU5NDU2NA==,size_16,color_FFFFFF,t_70)

>报错，是因为本地分支并没有和远程跟踪分支共进退，推荐我们使用`git branch --set-upstream-to....`，这样就可以两者共进退，看着像是没有了远程跟踪分支，直接本地分支和远程分支进行交互

>之后在克隆后的目录中，远程跟踪分支才和本地分支自动同步其他的都要自己设置同步

###  git branch -u 远程跟踪分支

上面提到的本地分支和远程跟踪分支共进退，推荐使用`git branch --set-upstream-to-....`，这里可以简写，用`git branch -u 远程跟踪分支`,方便使用 这样会使**本地分支变成一个跟踪分支**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210124205200224.png)

>可以看到此时，本地当前库的master在跟踪远程的2021远程库的master

![在这里插入图片描述](https://img-blog.csdnimg.cn/2021012420534840.png)

>当然本地和远程库目前是一样的，因为没有人对远程库进行修改，当前实际中，远程库不止自己一个人在使用，所以会有不同

自己手动在github上修改过了现在就可以pull过来了

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210124210018295.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDU5NDU2NA==,size_16,color_FFFFFF,t_70)

>总结：原本的`git fetch 别名`这个命令是先放到远程跟踪分支上去的，它的主要是如果自己不想直接拉取到自己的分支上去，自己可以先放在 远程跟踪分支上，然后 `git checkout -b 分支 远程跟踪分支`，这样先对远程跟踪分支修改，修改后的结果在合并到自己的分支中去，这样做也是很方便的

通过 git branch -vv 可以查看**跟踪分支**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210124211240770.png)

## git checkout --track 远程跟踪分支

跟踪分支 上文有简略的提交的跟踪分支，这里也简单讲一下

当我们在远程仓库 **git fetch** 一个自己没有用过的一个其他分支时,可以用

```
 git checkout --track 远程跟踪分支
```

这个命令会生成一个与远程分支同名的分支，并切换到该分支
>当然如果已经有分支了，就用上面的git branch -u 远程跟踪分支

## 冲突问题

当同一文件，别人修改了向远程库推，你也修改了向远程库推，由于你们彼此没有对方修改过的东西，这样提交会报错

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210124213555977.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDU5NDU2NA==,size_16,color_FFFFFF,t_70)

所以在向远程库推之前，应该先拉一下，把别人可能修改的拉过来，这时会造成两个版本的东西在一个文件里，git 会自动告诉你，并帮你标识两个版本修改过的地方，然后只需要，修改，add,commit就可以了

>这个与之前的，开分支，主干走在分支前，分支和主干都修改了相同文件，合并操作产生的冲突类似  
[分支合并冲突](#分支合并冲突)

## 删除远程分支

git push origin --delete serverfix //删除远程分支 

git remote prune origin --dry-run //列出仍在远程跟踪但是远程已被删除的无用分支 

git remote prune origin // 清除上面命令列出来的远程跟踪

>注意：删除远程分支肯定会留下远程跟踪分支，所以上面的是一套组合拳

## github 远程仓库的一些问题

 有个fork，可以把别人的代码拿到自己的给GitHub中,修改后，想和原作者说的话可以pull request

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210124223557856.png)

与原作者团队，发信息

## 忽略文件

可在 .git 同级目录下新建一个 .gitignore 文件 这里面保存一些上传时忽略上传的文件，因为并不是所有文件都应该上传

如：  
node_modules/  
.idea  
等等

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210124230246617.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDU5NDU2NA==,size_16,color_FFFFFF,t_70)

示例

![在这里插入图片描述](https://img-blog.csdnimg.cn/2021012423033071.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDU5NDU2NA==,size_16,color_FFFFFF,t_70)

<a href="https://github.com/github/gitignore" target="frame1">https://github.com/github/gitignore</a>

## SSH

ssh是一个公私钥，用来验证身份使用

这样上传代码就不会验证账号密码，直接验证ssh是否配对成功

在window的cmd 中运行

```
ssh-keygen -t rsa -C 你的邮箱 //生成公私钥
```

生成的文件位置：  
 C:\Users\Administrator\.ssh

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210124232420337.png)

.pub的给GitHub

这样就配对了

```
ssh -T git@github.com  //测试公私钥是否已经配对
```

***

由此，git 本地与远程仓库都更新完毕，通过这次学习感觉通透了很多


