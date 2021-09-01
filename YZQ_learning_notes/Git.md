# Git

### 一、Git是什么？

#### 1.概念

Git 是一个开源的***分布式版本控制系统***，用于敏捷高效地处理任何或小或大的项目。

#### 2.与其他版本控制工具的区别和特点

- 与常用的版本控制工具 CVS, Subversion 等不同，主要差别在于 Git 对待数据的方式。

- 其它大部分系统*以文件变更列表的方式存储信息*，Git 更像是把数据看作是对*小型文件系统的一系列快照*。
-  在 Git 中，每当你提交更新或保存项目状态时，它基本上就会对当时的全部文件创建一个快照并保存这个快照的索引。 为了效率，如果文件没有修改，Git 不再重新存储该文件，而是只保留一个链接指向之前存储的文件。
-  Git 对待数据更像是一个***快照流***。
- 在 Git 中的绝大多数操作都只需要访问***本地文件和资源***，一般不需要来自网络上其它计算机的信息，可在离线时进行任意操作。
- Git 中所有的数据在存储前都***计算校验和***，然后以校验和来引用。 这意味着不可能在 Git 不知情时更改任何文件内容或目录内容。
- Git几乎只往数据库中 ***添加*** 数据。 很难使用 Git 从数据库中删除数据，也就是说 Git 几乎 不会执行任何可能导致文件不可恢复的操作。

#### 3.三种状态

 Git 有三种状态，你的文件可能 处于其中之一： **已提交**（committed）、**已修改**（modified） 和 **已暂存**（staged）。

- 已修改表示修改了文件，但还没保存到数据库中。 
- 已暂存表示对一个已修改文件的当前版本做了标记，使之包含在下次提交的快照中。 
- 已提交表示数据已经安全地保存在本地数据库中。 

 因此Git 项目拥有三个阶段：*工作区、暂存区以及 Git 目录。*

<img src="C:\Users\颜泽卿\AppData\Roaming\Typora\typora-user-images\image-20210829103614248.png" alt="image-20210829103614248" style="zoom:50%;" />

- 工作区：包括git已经管理的文件区域和新增以及修改的文件区域；
- 暂存区：把工作区中的部分或者全部已经确认需要保存的文件提交至暂存区，先保存一下，如果直接确认可以提交到本地仓库中；如果暂时不能做决定可以暂时先放到暂存区，等可以做决定了，再进行下一步操作。
- Git目录：将某一个历史节点的保存文件列表当做是一个版本。

### 二、Git基础

本节主要介绍如何配置并初始化一个仓库（repository）、跟踪（track）文件、暂存（stage）或提交（commit）更改；演示了如何配置 Git 来忽略指定的文件和文件模式、如何迅速而简单地撤销错误操作、如何向远程仓库推送（push）和拉取（pull）文件。

#### 1.获取 Git 仓库

通常有两种获取 Git 项目仓库的方式：

- 将尚未进行版本控制的本地目录**转换**为 Git 仓库；

- 从其它服务器 **克隆** 一个已存在的 Git 仓库。

##### 1.1在已存在目录初始化仓库--git init

首先需要进入该项目目录中之后执行：

```console
$ git init
Reinitialized existing Git repository in D:/a IIE/git/learngit/.git/
```

该命令将创建一个名为 `.git` 的子目录，这个子目录含有初始化的 Git 仓库中所有的必须文件，这些文件是 Git 仓库的骨干。

##### 1.2克隆现有的仓库--git clone

Git 克隆的是该 Git 仓库服务器上的几乎所有数据， 当执行 `git clone` 命令的时候，默认配置下远程 Git 仓库中的*每一个文件的每一个版本*都将被拉取下来。 事实上，如果服务器的磁盘坏掉了，你通常可以使用任何一个克隆下来的用户端来重建服务器上的仓库 。

克隆仓库的命令是 `git clone <url>` 。 比如，要克隆 Git 的链接库，可以用下面的命令：

```console
$ git clone https://github.com/E0HYL/AndrozooDownloader.git
Cloning into 'AndrozooDownloader'...
remote: Enumerating objects: 77, done.
remote: Counting objects: 100% (77/77), done.
remote: Compressing objects: 100% (49/49), done.
remote: Total 77 (delta 43), reused 56 (delta 25), pack-reused 0
Receiving objects: 100% (77/77), 113.28 KiB | 73.00 KiB/s, done.
Resolving deltas: 100% (43/43), done.
```

这会在当前目录下创建一个名为 “`AndrozooDownloader`” 的目录，并在这个目录下初始化一个 `.git` 文件夹， 从远程仓库拉取下所有数据放入 `.git` 文件夹，然后从中读取最新版本的文件的拷贝。 如果你进入到这个新建的 `AndrozooDownloader` 文件夹，你会发现所有的项目文件已经在里面了，准备就绪等待后续的开发和使用。

如果你想在克隆远程仓库的时候，自定义本地仓库的名字，你可以通过额外的参数指定新的目录名：

```console
$ git clone https://github.com/E0HYL/AndrozooDownloader.git myandrozoo
```

这会执行与上一条命令相同的操作，但目标目录名变为了 `myandrozoo`。

<img src="C:\Users\颜泽卿\AppData\Roaming\Typora\typora-user-images\image-20210829133323058.png" alt="image-20210829133323058" style="zoom: 80%;" />

Git 支持多种数据传输协议。 上面的例子使用的是 `https://` 协议，不过你也可以使用 `git://` 协议或者使用 SSH 传输协议，比如 `user@server:path/to/repo.git` 。

####  2.记录每次更新到仓库

现在我们有了一个 **真实项目** 的 Git 仓库，通常，我们会对这些文件做些修改，每当完成了一个阶段的目标，想要将记录下它时，就将它提交到到仓库。

工作目录下的每一个文件都不外乎这两种状态：**已跟踪** 或 **未跟踪**。 已跟踪的文件是指那些被纳入了版本控制的文件，在上一次快照中有它们的记录，在工作一段时间后， 它们的状态可能是未修改，已修改或已放入暂存区。简而言之，*已跟踪的文件就是 Git 已经知道的文件。*

工作目录中除已跟踪文件外的其它所有文件都属于未跟踪文件，它们既不存在于上次快照的记录中，也没有被放入暂存区。 初次克隆某个仓库的时候，工作目录中的所有文件都属于已跟踪文件，并处于未修改状态，因为 Git 刚刚检出了它们， 而尚未编辑过它们。

编辑过某些文件之后，Git 将它们标记为已修改文件。 在工作时，可以选择性地将这些修改过的文件放入暂存区，然后提交所有已暂存的修改，如此反复。

![Git 下文件生命周期图。](https://git-scm.com/book/en/v2/images/lifecycle.png)

#### 3.检查当前文件状态--git status

可以用 `git status` 命令查看哪些文件处于什么状态。在该仓库下添加一个`README`文件后，使用 `git status` 命令：

```console
$ git status
On branch master

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
        new file:   main.py

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        AndrozooDownloader/
        README.txt
        myandrozoo/
```

#### 4.跟踪新文件--git add

使用命令 `git add` 开始跟踪一个文件。 

```console
$ git add README.txt
```

此时再运行 `git status` 命令，会看到 `README` 文件已被跟踪，并处于暂存状态：

```console
$ git status
On branch master

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
        new file:   README.txt
        new file:   main.py

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        AndrozooDownloader/
        myandrozoo/
```

只要在 `Changes to be committed` 这行下面的，就说明是已暂存状态。 如果此时提交，那么该文件在运行 `git add` 时的版本将被留存在后续的历史记录中。 

##### 状态简览--git status -s

```
$ git status -s
AM README.txt
A  main.py
?? AndrozooDownloader/
?? myandrozoo/
```

新添加的未跟踪文件前面有 `??` 标记，新添加到暂存区中的文件前面有 `A` 标记，修改过的文件前面有 `M` 标记。

#### 5.忽略文件--.gitignore

一般我们总会有些文件无需纳入 Git 的管理，也不希望它们总出现在未跟踪文件列表。 通常都是些自动生成的文件，比如日志文件，或者编译过程中创建的临时文件等。 在这种情况下，我们可以创建一个名为 `.gitignore` 的文件，列出要忽略的文件的模式。

```console
$ vim .gitignore
```

类似Linux的操作，编辑.`gitignore`文件，输入想要忽略的文件类型，：wq退出保存即可。

<img src="C:\Users\颜泽卿\AppData\Roaming\Typora\typora-user-images\image-20210829141817064.png" alt="image-20210829141817064" style="zoom:80%;" />

#### 6.移除文件--git rm

要从 Git 中移除某个文件，就必须要从暂存区域移除，然后提交。 可以用 `git rm` 命令完成此项工作，并连带从工作目录中删除指定的文件，这样以后就不会出现在未跟踪文件清单中了。

```console
$ rm test.txt

$ git status
On branch master

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
        new file:   README.txt
        new file:   main.py
        new file:   test.txt

Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   README.txt
        deleted:    test.txt

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        .gitignore
        AndrozooDownloader/
        myandrozoo/
```

然后再运行 `git rm` 记录此次移除文件的操作：

```console
$ git rm test.txt
rm 'test.txt'
```

下一次提交时，该文件就不再纳入版本管理了。 如果要删除之前修改过或已经放到暂存区的文件，则必须使用强制删除选项 `-f`。 这是一种安全特性，用于防止误删尚未添加到快照的数据，这样的数据不能被 Git 恢复。

#### 7.文件改名--git mv

 要在 Git 中对文件改名，可以这么做：

```console
$ git mv README.txt READ
```

运行 `git mv` 就相当于运行了下面三条命令：

```console
$ mv README.txt READ
$ git rm README.txt
$ git add READ
```

#### 8.提交更新--git commit

在提交之前，务必确认还有什么已修改或新建的文件还没有 `git add` 过， 否则提交的时候不会记录这些尚未暂存的变化。 这些已修改但未暂存的文件只会保留在本地磁盘。 所以，每次准备提交前，先用 `git status` 看下，所需要的文件是不是都已暂存起来了， 然后再运行提交命令 `git commit`：

```
$ git commit
[master (root-commit) 0b18d84] test
 2 files changed, 193 insertions(+)
 create mode 100644 READ
 create mode 100644 main.py
```

#### 9.撤销操作

###### 重新提交

有时候我们提交完了才发现漏掉了几个文件没有添加，或者提交信息写错了。 此时，可以运行带有 `--amend` 选项的提交命令来重新提交：

```console
$ git commit --amend
```

这个命令会将暂存区中的文件提交。 如果自上次提交以来还未做任何修改（例如，在上次提交后马上执行了此命令）， 那么快照会保持不变，而所修改的只是提交信息。

例如，提交后发现忘记了暂存某些需要的修改，可以这样操作：

```console
$ git commit -m 'initial commit'
$ git add forgotten_file
$ git commit --amend
```

###### 撤消对文件的修改

如果我们并不想保留对某文件的修改，希望它还原成上次提交时的样子。使用git checkout对其撤销。

```console
$ git checkout -- READ
```

#### 10.查看日志--git log 

`git log [<options>] [<revision range>] [[\--] <path>…]`

常见的`format`选项：

| 选项  | 说明                                          |
| :---- | :-------------------------------------------- |
| `%H`  | 提交的完整哈希值                              |
| `%h`  | 提交的简写哈希值                              |
| `%T`  | 树的完整哈希值                                |
| `%t`  | 树的简写哈希值                                |
| `%P`  | 父提交的完整哈希值                            |
| `%p`  | 父提交的简写哈希值                            |
| `%an` | 作者名字                                      |
| `%ae` | 作者的电子邮件地址                            |
| `%ad` | 作者修订日期（可以用 --date=选项 来定制格式） |
| `%ar` | 作者修订日期，按多久以前的方式显示            |
| `%cn` | 提交者的名字                                  |
| `%ce` | 提交者的电子邮件地址                          |
| `%cd` | 提交日期                                      |
| `%cr` | 提交日期（距今多长时间）                      |
| `%s`  | 提交说明                                      |

| 选项              | 说明                                                         |
| :---------------- | :----------------------------------------------------------- |
| `-p`              | 按补丁格式显示每个提交引入的差异。                           |
| `--stat`          | 显示每次提交的文件修改统计信息。                             |
| `--shortstat`     | 只显示 --stat 中最后的行数修改添加移除统计。                 |
| `--name-only`     | 仅在提交信息后显示已修改的文件清单。                         |
| `--name-status`   | 显示新增、修改、删除的文件清单。                             |
| `--abbrev-commit` | 仅显示 SHA-1 校验和所有 40 个字符中的前几个字符。            |
| `--relative-date` | 使用较短的相对时间而不是完整格式显示日期（比如“2 weeks ago”）。 |
| `--graph`         | 在日志旁以 ASCII 图形显示分支与合并历史。                    |
| `--pretty`        | 使用其他格式显示历史提交信息。可用的选项包括 oneline、short、full、fuller 和 format（用来定义自己的格式）。 |
| `--oneline`       | `--pretty=oneline --abbrev-commit` 合用的简写。              |

### 三、Git分支

- Git 的分支是它最明显的特性， 大部分人听别人推荐使用git都会听到“git分支操作方便...”，对比其他版本控制系统git 分支操作有难以置信的轻量，创建新分支几乎瞬间完成，不同分支之间切换也非常快捷方便。
- 分支是为了将修改记录的整体流程分叉保存。分叉后的分支不受其他分支的影响，所以在同一个数据库里可以同时进行多个修改。

![img](https://backlog.com/git-tutorial/cn/img/post/stepup/capture_stepup1_1_2.png)

首先，我们查看本地分支，`git branch`命令会列出所有分支，当前分支前面会标一个`*`号。

```
$ git branch
* master
```

创建`dev`分支，然后切换到`dev`分支：

```
$ git checkout -b dev
Switched to a new branch 'dev'
```

`git checkout`命令加上`-b`参数表示创建并切换，相当于以下两条命令：

```
$ git branch dev
$ git checkout dev
Switched to branch 'dev'
```

然后，再查看当前分支：

```
$ git branch
* dev
  master
```

然后，我们就可以在`dev`分支上正常提交，比如对README.txt做个修改，加上一行：

```
create new branch dev..
```

然后提交：

```
$ git add README.txt
$ git commit -m 'create new branch dev..'
[dev 00ee731] create new branch dev..
 5 files changed, 4 insertions(+)
 create mode 160000 AndrozooDownloader
 delete mode 100644 READ
 create mode 100644 README.txt
 create mode 160000 myandrozoo
 create mode 160000 test
```

现在，`dev`分支的工作完成，我们就可以切换回`master`分支：

```
$ git checkout master
Switched to branch 'master'
Your branch is up-to-date with 'origin/master'.
```

切换回`master`分支后，再查看一个README.txt文件，刚才添加的内容不见了！因为那个提交是在`dev`分支上，而`master`分支此刻的提交点并没有变：

![img](https://images2015.cnblogs.com/blog/925240/201604/925240-20160423181854210-135268393.png)

现在，我们把`dev`分支的工作成果合并到`master`分支上：

```
$ git merge dev
CONFLICT (add/add): Merge conflict in README.txt
Auto-merging README.txt
Merge made by the 'recursive' strategy.
 AndrozooDownloader | 1 +
 README.txt         | 1 +
 myandrozoo         | 1 +
 test               | 1 +
 4 files changed, 4 insertions(+)
 create mode 160000 AndrozooDownloader
 create mode 160000 myandrozoo
 create mode 160000 test
```

`git merge`命令用于合并指定分支到当前分支。合并后，再查看README.txt的内容，就可以看到，和`dev`分支的最新提交是完全一样的。

<img src="C:\Users\颜泽卿\AppData\Roaming\Typora\typora-user-images\image-20210829161036146.png" alt="image-20210829161036146" style="zoom:80%;" />

合并完成后，就可以放心地删除`dev`分支了：

**删除本地分支**

```text
// "-d" 如果该分支代码未合并到其他分支，将无法删除；
// "-D" 强制删除分支，不会出现任何提示；
git branch -D xxxx
```

```
$ git branch -d dev
Deleted branch dev (was 00ee731).
```

删除后，查看`branch`，就只剩下`master`分支了：

```
$ git branch
* master
```

因为创建、合并和删除分支非常快，所以Git鼓励使用分支完成某个任务，合并后再删掉分支，这和直接在`master`分支上工作效果是一样的，但过程更安全。

#### 解决冲突

人生不如意之事十之八九，合并分支往往也不是一帆风顺的。

准备新的`feature1`分支，继续我们的新分支开发：

```
$ git checkout -b feature
Switched to a new branch 'feature'
```

修改README.txt最后一行，改为：

```
create new branch feature..
```

在`feature`分支上提交：

```
$ git commit -m 'first modify'
[feature 5944aa6] first modify
 1 file changed, 1 insertion(+), 1 deletion(-)
```

切换到`master`分支：

```
$ git checkout master
Switched to branch 'master'
M       test
```

在`master`分支上把README.txt文件的最后一行改为：

```
goback master....
```

提交：

```
$ git add README.txt
$ git commit -m "goback master first modify"
[master 9e44d80] goback master first modify
 1 file changed, 1 insertion(+), 1 deletion(-)
```

现在，`master`分支和`feature`分支各自都分别有新的提交，变成了这样：

![img](https://images2015.cnblogs.com/blog/925240/201604/925240-20160423183358601-286358803.png)

这种情况下，Git无法执行“快速合并”，只能试图把各自的修改合并起来，但这种合并就可能会有冲突：

```
$ git merge feature
Auto-merging README.txt
CONFLICT (content): Merge conflict in README.txt
Automatic merge failed; fix conflicts and then commit the result.
```

`git status`也可以告诉我们冲突的文件：

```
$ git status
On branch master
You have unmerged paths.
  (fix conflicts and run "git commit")
  (use "git merge --abort" to abort the merge)

Unmerged paths:
  (use "git add <file>..." to mark resolution)
        both modified:   README.txt
```

我们可以直接查看README.txt的内容：

```
<<<<<<< HEAD
goback master....
=======
create new branch feature1..
>>>>>>> feature1
```

Git用`<<<<<<<`，`=======`，`>>>>>>>`标记出不同分支的内容，我们修改如下后保存：

```
goback master....
create new branch feature..
```

再提交：

```
$ git add README.txt
$ git commit -m "fixed conflicts"
[master 0f3d64a] fixed conflicts
```

用带参数的`git log`可以看到分支的合并情况：

```
$ git log --graph --pretty=oneline --abbrev-commit
*   220cfec (HEAD -> master) fixed conflicts
|\
| * 5944aa6 (feature) first modify
* | 9e44d80 goback master first modify
|/
*   2a5bbce Merge branch 'dev'
|\
| * 00ee731 create new branch dev..
* | 777e6df q
|/
* 0b18d84 test
```

最后，删除`feature`分支：

```
$ git branch -d feature
Deleted branch feature (was 5944aa6).
```

当Git无法自动合并分支时，就必须首先解决冲突。解决冲突后，再提交，合并完成。

用`git log --graph`命令可以看到分支合并图。

### 四、远程仓库（gitee）

##### 1.查看远程仓库--git remote

如果想查看已经配置的远程仓库服务器，可以运行 `git remote` 命令。 

```console
$ git clone https://gitee.com/yan-zeqing0628/gittest.git
Cloning into 'gittest'...
remote: Enumerating objects: 6, done.
remote: Counting objects: 100% (6/6), done.
remote: Compressing objects: 100% (6/6), done.
remote: Total 6 (delta 0), reused 0 (delta 0), pack-reused 0
Receiving objects: 100% (6/6), done.
```

可以指定选项 `-v`，会显示需要读写远程仓库使用的 Git 保存的简写与其对应的 URL。

```console
$ git remote -v
origin  https://gitee.com/yan-zeqing0628/gittest.git (fetch)
origin  https://gitee.com/yan-zeqing0628/gittest.git (push)
```

运行 `git remote add <shortname> <url>` 添加一个新的远程 Git 仓库，同时指定一个方便使用的简写：

```console
$ git remote add dp  https://gitee.com/yan-zeqing0628/gittest.git
$ git remote -v
dp      https://gitee.com/yan-zeqing0628/gittest.git (fetch)
dp      https://gitee.com/yan-zeqing0628/gittest.git (push)
origin  https://gitee.com/yan-zeqing0628/gittest.git (fetch)
origin  https://gitee.com/yan-zeqing0628/gittest.git (push)
```

现在可以在命令行中使用字符串 `dp` 来代替整个 URL。 

#### 2.从远程仓库中抓取与拉取--git fetch

从远程仓库中获得数据，可以执行：

```console
$ git fetch <remote>
```

这个命令会访问远程仓库，从中拉取所有我们还没有的数据。 执行完成后，将会拥有那个远程仓库中所有分支的引用，可以随时合并或查看。

#### 3.推送到远程仓库--git push

当你想分享你的项目时，必须将其推送到上游。 这个命令很简单：`git push <remote> <branch>`。 当你想要将 `master` 分支推送到 `origin` 服务器时， 那么运行这个命令就可以将你所做的备份到服务器：

```console
$ git push origin master
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Delta compression using up to 12 threads
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 283 bytes | 283.00 KiB/s, done.
Total 3 (delta 2), reused 0 (delta 0), pack-reused 0
remote: Powered by GITEE.COM [GNK-6.1]
To https://gitee.com/yan-zeqing0628/gittest.git
   e5e555e..68e8eb7  master -> master
```



==



