# Git结构

![](D:\File\学习笔记\开发工具\Git\images\搜狗截图20190728221533.png)

# Git命令行操作

## 1.1、本地仓库初始化

**命令：** git init

![](D:\File\学习笔记\开发工具\Git\images\1564043444(1).jpg)

**效果：**

![1564047660792](C:\Users\csw\AppData\Roaming\Typora\typora-user-images\1564047660792.png)

**注意：** .git 目录中存放的是本地库相关的子目录和文件，不要删除，也不要胡乱修改。

## 1.2、设置签名

作用：区分不同开发人员的身份

命令：

- 项目级别/仓库级别：仅在当前本地库范围有效	
  - git **config** user.name csw

  ![1564236984064](D:\File\学习笔记\开发工具\Git\images\1564236984064.png)

  - git **config** user.email caisw11@chinaunicom.cn
  

![](D:\File\学习笔记\开发工具\Git\images\搜狗截图20190727221857.png)

  - 信息保存位置：./.git/config
  
    ![](D:\File\学习笔记\开发工具\Git\images\搜狗截图20190727221938.png)
  
- 系统级别：登录当前操作系统的用户范围

  - git **config --global** user.name csw
  - git **config** --global user.email caisw11@chinaunicom.cn
  - 信息保存位置：~/.gitconfig

- 级别优先级

  - 就近原则：项目级别优先于系统级别，二者都用时采用项目级别签名
  - 如果只有系统用户级别的签名，就以系统用户级别的签名为准
  - 二者都没有不允许

## 1.3、基本命令

### 1.3.1、状态查看

**git status** ：查看工作区、暂存区状态

### 1.3.2、添加

**git add** [file name] ：将工作区的“新建/修改”添加到暂存区

### 1.3.3、提交

**git commit -m** "commit message" [file name] ：将暂存区的内容提交到本地库

### 1.3.4、查看历史记录

- **git log** 

![](D:\File\学习笔记\开发工具\Git\images\搜狗截图20190728224404.png)

- **git log --pretty=oneline**

![](D:\File\学习笔记\开发工具\Git\images\搜狗截图20190728224603.png)

- **git log --oneline**

![](D:\File\学习笔记\开发工具\Git\images\搜狗截图20190728224656.png)

- **git reflog**

![](D:\File\学习笔记\开发工具\Git\images\搜狗截图20190728225127.png)

### 1.3.5、分支操作

- git branch [分支名] ：创建分支
- git branch -v ：查看分支
- git checkout [分支名] ：切换分支
- 合并分支：
  1. git checkout [分支名] ：切换到接受合并（被合并，添加新内容）的分支上；
  2. git merge [分支名] ：合并分支，这里的分支名为要进行合并的分支名

- 解决冲突

  - 冲突的表现

    ![](D:\File\学习笔记\开发工具\Git\images\搜狗截图20190728230046.png)

  - 冲突的解决
    1. 编辑文件，删除特殊符号
    2. 修改文件，保存退出
    3. git add [文件名]
    4. git commit -m "commit message"：此时 commit 后不能带文件名，否则会报错！

### 1.3.6、

