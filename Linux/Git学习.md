1. 集中化的版本控制系统（Centralized Version Control Systems， CVCS）
2. 分布式版本控制系统（Distributed Version Control System，简称 DVCS）
2. [图文并茂git笔记](https://blog.csdn.net/Augenstern_QXL/article/details/120088567)

# Git基础命令

1. 切换到桌面：`cd desktop`
3. 新增文件：vim hello.txt 然后按 i 键进入 INSERT，要想复制粘贴 ，需要先按 esc 键，之后 `yy` 复制，`p` 粘贴
   1. 文件内容输入完毕，需要先按`:`,输入`wq`，然后才算完成新增文件，再次查看
   2. $ cat hello.txt (`cat 文件名` 查看文件内容)
3. 克隆并自定义本地仓库的名字: `git clone [url] directoryname`
4. 初始化本地仓库：`git init`   查看当前状态：`git  status`
5. 提交到暂存区：==`git add *`(所有文件)==、`git add *.txt`（支持通配符，所有 .txt 文件）
6. 提交更新: `git commit -m "代码提交信息"`
7. 跳过使用暂存区域更新的方式 : `git commit -a -m "代码提交信息"`。(跳过 git add 步骤。)
8. ==查看日志：$ `git log --pretty=oneline`==；`git reflog` 查看版本信息；`git log` 查看版本详细信息
9. ==版本穿梭：git reset --hard 版本号==

##  分支

1. 查看分支：`git branch -v`
2. 创建分支：`git branch 分支名`   比如：git branch hot-fix
3. 切换分支：`git checkout 分支名` 
4. 合并分支：`git merge 分支名`   比如：在master分支上合并hot-fix分支

#  Git进阶

##  git-config

1. 显示所有帮助：`git help -a` ，显示add的详细信息：`git help add`（F向下翻页，B向上翻页）
2. 设置用户名：$ git config --global user.name jun
3. 查看自己的用户名或邮箱：$ `git config --global` user.name ，$ git config --global user.email
4. 查看配置信息：`git config --list`
5. 执行：`git config unset --global user.name`时出现错误信息error：key dose not contain a section：unset
   1. 查看出错的信息：git config --help，发现输入错了改正为：`git config --unset --global user.name`
6. 让git输出的类容带颜色：`git config --global color.ui true`
7. 在全局里的设置，都会放在当前用户的主目录里：查看主目录：`cat ~/.gitconfig`

## git-diff

1. 在暂存区：查看一个文件修改前后的差别：`git diff index.html`
2. 如果想要比较暂存区与工作目录的对比：`git diff --staged`

#  Github

> 为了把本地的仓库传到github，还需要配置ssh key。

- 在本地创建ssh key
  - $ ssh-keygen -t rsa -C "your_email@youremail.com"
- 验证是否成功，在git bash下输入:
  - $ ssh -T git@github.com
- 查看当前所有远程地址别名：`git remote -v `    【起的别名最好和本地库的名称一致】
- 添加一个远程仓库：`git remote add 别名 远程地址`
  - $ git remote add origin git@github.com:geek202021/ssm_crud.git
- 检查一个远程仓库
  - $ git remote show origin
- 修改一个远程仓库的别名
  - $ git remote rename origin origin2
- **上传项目到GitHub远程仓库** ：`git push 别名 分支`
  - **`$ git push -u origin master`**
  - -u 代表更新
- **将远程仓库对于分支最新内容拉下来后与当前本地分支直接合并**
  - `git pull 远程库地址别名 远程分支名`
  - $ git pull origin master


1. 查看远程服务器地址和仓库名称：git remote -v 
2. 设置远程仓库地址(用于修改远程仓库地址)：git remote set-url origin git@ github.com:robbin/robbin_site.git



#  遇到的错误

1. fatal: in unpopulated submodule 'SSM'：致命：在未填充的子模块“SSM”中 ==`unpopulated`==：无人居住;无人居住的;无人区;**未填充**

