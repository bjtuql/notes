# Git

## 版本控制

### 创建版本库

```bash
git init # 会在当前目录下多一个.git的目录
```

### 添加文件

```bash
git add filesname # 添加文件到暂存区
git commit -m msg # 添加文件到仓库，并附带msg消息
```

### 查看

```bash
git status # 查看当前仓库状态
git diff filename # 查看文件被修改的内容
```

### 回退

```bash
git log [--pretty=oneline] # 查看历史记录[简化输出信息]
git reset --hrad 版本号 # 回退到上一版本，HEAD^表示上一版本，HEAD^^表示上上版本，HEAD~100表示上100个版本
git reflog # 记录每一次的git命令
```

**注意**：Git版本回退速度非常快，改变`HEAD`指针即可。

### 撤销修改

```bash
git checkout -- filename # 丢弃工作区的修改 
# 没有`--`，git checkout表示切换分支
```

> 两种情况：
>
> 1. 对于修改后没被放到暂存区的，撤销修改后回到和版本库一模一样的状态
> 2. 对于已加入暂存区又作修改的，撤销修改后回到添加到暂存区后的状态。

### 删除

```bash
git rm filename # 从版本库中删除改文件
```

## 远程仓库

```bash
ssh-keygen -t rsa -C "邮件地址" # 生成id_rsa私钥和id_rsa.pub公钥
git remote add origin git@github.com:用户名/仓库名.git # 本地与github远程连接 [origin是远程库名字]
```

### 推送

```bash
git push -u origin master # 将master分支推送到远程。第一次需加-u将本地master和远程master关联起来
```

### 克隆

```bash
git clone git@github.com:用户名/仓库名.git # 克隆远程到本地当前目录
```



## 分支管理

### 创建切换

```bash
git branch # 查看所有分支，当前分支前有*
git branch 新分支 # 创建新分支
git switch -c 新分支 / git checkout -b 新分支# 创建并切换到新分支，不加-c/-b表示切换到已有分支
```

### 合并

```bash
git meger 某分支 # 合并某分支到当前分支
```

### 删除

```bash
git branch -d 某分支 # 删除某分支，-D强制删除
```

### 冲突

两个分支对同一个文件的同一行进行了修改，合并分支时会产生冲突。

Git中使用`<<<<<<<`，`=======`，`>>>>>>>`标记处不同分支的内容，只需要把不同分支中冲突部分修改成一样的就能解决冲突。

```bash
<<<<<<< HEAD
Creating a new branch is quick & simple.
=======
Creating a new branch is quick AND simple.
>>>>>>> feature1
```

### 管理策略

合并分支时使用`--no-ff`参数，表示禁用Fast forward模式，在merge时生成一个新的commit，这样就能从分支历史上看出分支信息。

### 储藏

```bash
git stash # 将当前工作现场保存起来，等以后恢复现场后继续工作
git stash apply stash@{x} # 恢复现场，但不删除stash内容
git stash drop # 删除stash内容
git stash pop # 恢复现场同时删除stash内容
```

> stash后当前工作区便是干净的，并且未完成的工作也不会被提交

### 复制部分修改

```bash
git cherry-pick 版本号 # 将特定的commit复制提交到当前分支
```

