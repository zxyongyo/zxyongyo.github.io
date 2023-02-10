---
title: git rebase 用法详解与工作原理(转载)
categories: Git
tags:
  - Git
keywords: 'git,rebase'
description: git rebase 用法详解与工作原理
copyright_author: Waynerv
copyright_author_href: 'https://waynerv.com'
copyright_url: 'https://waynerv.com/posts/git-rebase-intro/'
copyright_info: 转载于 Waynerv
abbrlink: dfe0e190
date: 2023-02-08 11:22:48
cover: 'https://cdn.jsdelivr.net/gh/zxyongyo/pictures/blog/202302101343488.webp'
---

以前对 `git rebase -i` 的用法一直是一知半解，一次在需要合并多个提交时刚好用到，一顿操作差点把提交都搞丢了，幸好后面顺利找回，因此记录一下学习 `rebase` 命令的过程。

## 理解 Rebase 命令

`git rebase` 命令的文档描述是 `Reapply commits on top of another base tip`，从字面上理解是「在另一个基端之上重新应用提交」，这个定义听起来有点抽象，换个角度可以理解为「将分支的基础从一个提交改成另一个提交，使其看起来就像是从另一个提交中创建了分支一样」，如下图：

![git-rebase.png](https://cdn.jsdelivr.net/gh/zxyongyo/pictures/blog/202302081158447.png)

假设我们从 `Master` 的提交 A 创建了 `Feature` 分支进行新的功能开发，这时 A 就是 `Feature` 的基端。接着 `Matser` 新增了两个提交 B 和 C， `Feature` 新增了两个提交 D 和 E。现在我们出于某种原因，比如新功能的开发依赖 B、C 提交，需要将 `Master` 的两个新提交整合到 `Feature` 分支，为了保持提交历史的整洁，我们可以切换到 `Feature` 分支执行 `rebase` 操作：

```bash
git rebase master
```

`rebase` 的执行过程是首先找到这两个分支（即当前分支 `Feature`、 `rebase` 操作的目标基底分支 `Master`） 的最近共同祖先提交 A，然后对比当前分支相对于该祖先提交的历次提交（D 和 E），提取相应的修改并存为临时文件，然后将当前分支指向目标基底 `Master` 所指向的提交 C, 最后以此作为新的基端将之前另存为临时文件的修改依序应用。

我们也可以按上文理解成将 `Feature` 分支的基础从提交 A 改成了提交 C，看起来就像是从提交 C 创建了该分支，并提交了 D 和 E。但实际上这只是「看起来」，在内部 Git 复制了提交 D 和 E 的内容，创建新的提交 D' 和 E' 并将其应用到特定基础上（A→B→C）。尽管新的 `Feature` 分支和之前看起来是一样的，但它是由全新的提交组成的。

`rebase` 操作的实质是丢弃一些现有的提交，然后相应地新建一些内容一样但实际上不同的提交。

## 主要用途

`rebase` 通常用于重写提交历史。下面的使用场景在大多数 Git 工作流中是十分常见的：

- 我们从 `master` 分支拉取了一条 `feature` 分支在本地进行功能开发
- 远程的 `master` 分支在之后又合并了一些新的提交
- 我们想在 `feature` 分支集成 `master` 的最新更改

### rebase 和 merge 的区别

以上场景同样可以使用 `merge` 来达成目的，但使用 `rebase` 可以使我们保持一个线性且更加整洁的提交历史。假设我们有如下分支：

```bash
  D---E feature
 /
A---B---C master
```

现在我们将分别使用 `merge` 和 `rebase`，把 `master` 分支的 B、C 提交集成到 `feature` 分支，并在 `feature` 分支新增一个提交 F，然后再将 `feature` 分支合入 `master` ，最后对比两种方法所形成的提交历史的区别。

- 使用 `merge`

  1. 切换到 `feature` 分支： `git checkout feature`。
  2. 合并 `master` 分支的更新： `git merge master`。
  3. 新增一个提交 F： `git add . && git commit -m "commit F"` 。
  4. 切回 `master` 分支并执行快进合并： `git chekcout master && git merge feature`。

  执行过程如下图所示：

  ![Dec-30-2020-merge-example](https://cdn.jsdelivr.net/gh/zxyongyo/pictures/blog/202302081159353.gif)

  我们将得到如下提交历史：

  ```bash
  * 6fa5484 (HEAD -> master, feature) commit F
  *   875906b Merge branch 'master' into feature
  |\  
  | | 5b05585 commit E
  | | f5b0fc0 commit D
  * * d017dff commit C
  * * 9df916f commit B
  |/  
  * cb932a6 commit A
  ```

- 使用 `rebase`

  步骤与使用 `merge` 基本相同，唯一的区别是第 2 步的命令替换成： `git rebase master`。

  执行过程如下图所示：

  ![Dec-30-2020-rebase-example](https://cdn.jsdelivr.net/gh/zxyongyo/pictures/blog/202302081159624.gif)

  我们将得到如下提交历史：

  ```bash
  * 74199ce (HEAD -> master, feature) commit F
  * e7c7111 commit E
  * d9623b0 commit D
  * 73deeed commit C
  * c50221f commit B
  * ef13725 commit A
  ```

可以看到，使用 `rebase` 方法形成的提交历史是完全线性的，同时相比 `merge` 方法少了一次 `merge` 提交，看上去更加整洁。

### 为什么要保持提交历史的整洁

一个看上更整洁的提交历史有什么好处？

1. 满足某些开发者的洁癖。
2. 当你因为某些 bug 需要回溯提交历史时，更容易定位到 bug 是从哪一个提交引入。尤其是当你需要通过 `git bisect` 从几十上百个提交中排查 bug，或者有一些体量较大的功能分支需要频繁的从远程的主分支拉取更新时。

使用 `rebase` 来将远程的变更整合到本地仓库是一种更好的选择。用 `merge` 拉取远程变更的结果是，每次你想获取项目的最新进展时，都会有一个多余的 `merge` 提交。而使用 `rebase` 的结果更符合我们的本意：我想在其他人的已完成工作的基础上进行我的更改。

### 其他重写提交历史的方法

当我们仅仅只想修改最近的一次提交时，使用 `git commit --amend` 会更加方便。

它适用于以下场景：

- 我们刚刚完成了一次提交，但还没有推送到公共的分支。
- 突然发现上个提交还留了些小尾巴没有完成，比如一行忘记删除的注释或者一个很小的笔误，我们可以很快速的完成修改，但又不想再新增一个单独的提交。
- 或者我们只是觉得上一次提交的提交信息写的不够好，想做一些修改。

这时候我们可以添加新增的修改（或跳过），使用 `git commit --amend` 命令执行提交，执行后会进入一个新的编辑器窗口，可以对上一次提交的提交信息进行修改，保存后就会将所做的这些更改应用到上一次提交。

如果我们已经将上一次提交推送到了远程的分支，现在再执行推送将会提示出错并被拒绝，在确保该分支不是一个公共分支的前提下，我们可以使用 `git push --force` 强制推送。

注意与 `rebase` 一样，Git 在内部并不会真正地修改并替换上一个提交，而是创建了一个全新的提交并重新指向这个新的提交。

## 使用 rebase 的交互模式重写提交历史

`git rebase` 命令有标准和交互两种模式，之前的示例我们用的都是默认的标准模式，在命令后添加 `-i` 或 `--interactive` 选项即可使用交互模式。

### 两种模式的区别

我们前面提到， `rebase` 是「在另一个基端之上重新应用提交」，而在重新应用的过程中，这些提交会被重新创建，自然也可以进行修改。在 `rebase` 的标准模式下，当前工作分支的提交会被直接应用到传入分支的顶端；而在交互模式下，则允许我们在重新应用之前通过编辑器以及特定的命令规则对这些提交进行合并、重新排序及删除等重写操作。

两者最常见的使用场景也因此有所不同：

1. 标准模式常用于在当前分支中集成来自其他分支的最新修改。
2. 交互模式常用于对当前分支的提交历史进行编辑，如将多个小提交合并成大的提交。

### 不仅仅是分支

虽然我们之前的示例都是在不同的两个分支之间执行 rebase 操作，但事实上 rebase 命令传入的参数并不仅限于分支。

任何的提交引用，都可以被视作有效的 `rebase` 基底对象，包括一个提交 ID、分支名称、标签名称或 `HEAD~1` 这样的相对引用。

自然地，假如我们对当前分支的某次历史提交执行 `rebase`，其结果就是会将这次提交之后的所有提交重新应用在当前分支，在交互模式下，即允许我们对这些提交进行更改。

### 重写提交历史

终于进入到本文的主题，前面提到，假如我们在交互模式对当前分支的某次提交执行 `rebase`，即（间接）实现了对这次提交之后的所有提交进行重写。接下来我们将通过下面的示例进行详细介绍。

假设我们在 `feature` 分支有如下提交：

```bash
74199cebdd34d107bb67b6da5533a2e405f4c330 (HEAD -> feature) commit F
e7c7111d807c1d5209b97a9c75b09da5cd2810d4 commit E
d9623b0ef9d722b4a83d58a334e1ce85545ea524 commit D
73deeedaa944ef459b17d42601677c2fcc4c4703 commit C
c50221f93a39f3474ac59228d69732402556c93b commit B
ef1372522cdad136ce7e6dc3e02aab4d6ad73f79 commit A
```

接下来我们将要执行的操作是：

- 将 B、C 合并为一个新的提交 ，并仅保留原提交 C 的提交信息
- 删除提交 D
- 将提交 E 移动到提交 F 之后并重新命名（即修改提交信息）为提交 H
- 在提交 F 中加入一个新的文件更改，并重新命名为提交 G

由于我们需要修改的提交是 B→C→D→E，因此我们需要将提交 A 作为新的「基端」，提交 A 之后的所有提交会被重新应用：

```bash
git rebase -i ef1372522cdad136ce7e6dc3e02aab4d6ad73f79 # 参数是提交 A 的 ID
```

接下来会进入到如下的编辑器界面：

```bash
pick c50221f commit B
pick 73deeed commit C
pick d9623b0 commit D
pick e7c7111 commit E
pick 74199ce commit F

# 变基 ef13725..74199ce 到 ef13725（5 个提交）
#
# 命令:
# p, pick <提交> = 使用提交
# r, reword <提交> = 使用提交，但修改提交说明
# e, edit <提交> = 使用提交，进入 shell 以便进行提交修补
# s, squash <提交> = 使用提交，但融合到前一个提交
# f, fixup <提交> = 类似于 "squash"，但丢弃提交说明日志
# x, exec <命令> = 使用 shell 运行命令（此行剩余部分）
# b, break = 在此处停止（使用 'git rebase --continue' 继续变基）
# d, drop <提交> = 删除提交
......
```

（注意上面提交 ID 之后的提交信息只起到描述作用，在这里修改它们不会有任何效果。）

具体的操作命令在编辑器的注释中已解释的相当详细，所以我们直接进行如下操作：

1. 对提交 B、C 作如下修改：

   ```bash
   pick c50221f commit B
   f 73deeed commit C
   ```

   由于提交 B 是这些提交中的第一个，因此我们无法对其执行 `squash` 或者 `fixup` 命令（没有前一个提交了），我们也不需要对提交 B 执行 `reword` 命令以修改其提交信息，因为之后在将提交 C 融合到提交 B 中时，会允许我们对融合之后的提交信息进行修改。

   注意该界面提交的展示顺序是从上到下由旧到新，因此我们将提交 C 的命令改为 `s（或 squash）` 或者 `f（或 fixup）` 会将其融合到（上方的）前一个提交 B，两个命令的区别为是否保留 C 的提交信息。

2. 删除提交 D：

   ```bash
   d d9623b0 commit D
   ```

3. 移动提交 E 到提交 F 之后并修改其提交信息：

   ```bash
   pick 74199ce commit F
   r e7c7111 commit E
   ```

4. 在提交 F 中加入一个新的文件更改：

   ```bash
   e 74199ce commit F
   ```

5. 保存退出。

接下来会按照从上到下的顺序依次执行我们对每一个提交所修改或保留的命令：

1. 对提交 B 的 `pick` 命令会自动执行，因此不需要交互。

2. 接着执行对提交 C 的 `squash` 命令，会进入一个新的编辑器界面允许我们修改合并了B、C 之后的提交信息：

   ```bash
   # 这是一个 2 个提交的组合。
   # 这是第一个提交说明：
   
   commit B
   
   # 这是提交说明 #2：
   
   commit C
   ......
   ```

   我们将 `commit B` 这一行删除后保存退出，融合之后的提交将使用 `commit C` 作为提交信息。

3. 对提交 D 的 `drop` 操作也会自动执行，没有交互步骤。

4. 执行 `rebase` 的过程中可能会发生冲突，这时候 `rebase` 会暂时中止，需要我们编辑冲突的文件去手动合并冲突。解决冲突后通过 `git add/rm <conflicted_files>` 将其标记为已解决，然后执行 `git rebase --continue` 可以继续之后的 `rebase` 步骤；或者也可以执行 `git rebase --abort` 放弃 `rebase` 操作并恢复到操作之前的状态。

5. 由于我们上移了提交 F 的位置，因此接下来将执行对 F 的 `edit` 操作。这时将进入一个新的 Shell 会话：

   ```bash
   停止在 74199ce... commit F
   您现在可以修补这个提交，使用
   
     git commit --amend 
   
   当您对变更感到满意，执行
   
     git rebase --continue
   ```

   我们添加一个新的代码文件并执行 `git commit --amend` 将其合并到当前的上一个提交（即 F），然后在编辑器界面中将其提交信息修改为 `commit G`，最后执行 `git rebase --continue` 继续 `rebase` 操作。

6. 最后执行对提交 E 的 `reword` 操作，在编辑器界面中将其提交信息修改为 `commit H` 。

大功告成！最后让我们确认一下 `rebase` 之后的提交历史：

```bash
64710dc88ef4fbe8fe7aac206ec2e3ef12e7bca9 (HEAD -> feature) commit H
8ab4506a672dac5c1a55db34779a185f045d7dd3 commit G
1e186f890710291aab5b508a4999134044f6f846 commit C
ef1372522cdad136ce7e6dc3e02aab4d6ad73f79 commit A
```

完全符合预期，同时也可以看到提交 A之后的所有提交 ID 都已经发生了改变，这也印证了我们之前所说的 Git 重新创建了这些提交。

## Rebase 的进阶用法

### 合并之前执行 rebase

另一种使用 `rebase` 的常见场景是在推送到远程进行合并之前执行 `rebase`，一般这样做的目的是为了确保提交历史的整洁。

我们首先在自己的功能分支里进行开发，当开发完成时需要先将当前功能分支 `rebase` 到最新的主分支上，提前解决可能出现的冲突，然后再向远程提交修改。 这样的话，远程仓库的主分支维护者就不再需要进行整合且创建一条额外的 `merge` 提交，只需要执行快进合并即可。即使是在多个分支并行开发的情况，最终也能得到一条完全线性的提交历史。

### rebase 到其他分支

我们可以通过 `rebase` 对两个分支进行对比，取出相应的修改，然后应用到另一个分支上。例如：

```bash
    F---G patch
   /
  D---E feature
 /
A---B---C master
```

假设我们基于 `feature` 分支的提交 D 创建了分支 `patch`，并且新增了提交 F、G，现在我们想将 `patch` 所做的更改合并到 `master` 并发布，但暂时还不想合并 `feature` ，这种情况下可以使用 `rebase` 的 `--onto <branch>` 选项：

```bash
git rebase —onto master feature patch
```

以上操作将取出 `patch` 分支，对比它基于 `feature` 所做的更改， 然后把这些更改在 `master` 分支上重新应用，让 `patch` 看起来就像直接基于 `master` 进行更改一样。执行后的 `patch` 如下：

```bash
A---B---C---F'---G' patch
```

然后我们可以切换到 `master` 分支，并对 `patch` 执行快进合并：

```bash
git checkout master
git merge patch
```

### 通过 rebase 策略执行 `git pull`

Git 在最近的某个版本起，直接运行 `git pull` 会有如下提示消息：

```bash
warning: 不建议在没有为偏离分支指定合并策略时执行 pull 操作。 您可以在执行下一次 pull 操作之前执行下面一条命令来抑制本消息：

  git config pull.rebase false  # 合并（缺省策略）
  git config pull.rebase true   # 变基
  git config pull.ff only       # 仅快进

......
```

原来 `git pull` 时也可以通过 `rebase` 来进行合并，这是因为 `git pull` 实际上等于 `git fetch` + `git merge` ，我们可以在第二步直接用 `git rebase` 替换 `git merge`来合并 `fetch` 取得的变更，作用同样是避免额外的 `merge` 提交以保持线性的提交历史。

两者的区别在上文中已进行过对比，我们可以把对比示例中的 `Matser` 分支当成远程分支，把 `Feature` 分支当成本地分支，当我们在本地执行 `git pull` 时，其实就是拉取 `Master` 的更改然后合并到 `Feature` 分支。如果两个分支都有不同的提交，默认的 `git merge` 方式会生成一个单独的 merge 提交以整合这些提交；而使用 `git rebase` 则相当于基于远程分支的最新提交重新创建本地分支，然后再重新应用本地所添加的提交。

具体的使用方式有多种：

- 每次执行 pull 命令时添加特定选项： `git pull --rebase` 。
- 为当前仓库设定配置项： `git config pull.rebase true`，在 `git config` 后添加 `--global` 选项可以使该配置项对所有仓库生效。

## 潜在弊端和反对意见

从以上场景来看 `rebase` 功能非常强大，但我们也需要意识到它不是万能的，甚至对新手来说有些危险，稍有不慎就会发现 `git log` 里的提交不见了，或者卡在 `rebase` 的某个步骤不知道如何恢复。

我们上面已经提到了 `rebase` 有保持整洁的线性提交历史的优点，但也需要意识到它有以下潜在的弊端：

- 如果涉及到已经推送过的提交，需要强制推送才能将本地 `rebase` 后的提交推送到远程。因此绝对不要在一个公共分支（也就是说还有其他人基于这个分支进行开发）执行 `rebase`，否则其他人之后执行 `git pull` 会合并出一条令人困惑的本地提交历史，进一步推送回远程分支后又会将远程的提交历史打乱（详见 [Rebase and the golden rule explained](https://www.daolf.com/posts/git-series-part-2/)），较严重的情况下可能会对你的人身安全带来风险。
- 对新手不友好，新手很有可能在交互模式中误操作「丢失」某些提交（但其实是能够找回的）。
- 假如你频繁的使用 `rebase` 来集成主分支的更新，一个潜在的后果是你会遇到越来越多需要合并的冲突。尽管你可以在 `rebase` 过程中处理这些冲突，但这并非长久之计，更推荐的做法是频繁的合入主分支然后创建新的功能分支，而不是使用一个长时间存在的功能分支。

另外有一些观点是我们应该尽量避免重写提交历史：

> 有一种观点认为，仓库的提交历史即是 记录实际发生过什么。 它是针对历史的文档，本身就有价值，不能乱改。 从这个角度看来，改变提交历史是一种亵渎，你使用 谎言 掩盖了实际发生过的事情。 如果由合并产生的提交历史是一团糟怎么办？ 既然事实就是如此，那么这些痕迹就应该被保留下来，让后人能够查阅。

以及频繁的使用 `rebase` 可能会使从历史提交中定位 bug 变得更加困难，详见 [Why you should stop using Git rebase](https://medium.com/@fredrikmorken/why-you-should-stop-using-git-rebase-5552bee4fed1)。

## 找回丢失的提交

在交互式模式下进行 `rebase` 并对提交执行 `squash` 或 `drop` 等命令后，会从分支的 `git log` 中直接删除提交。如果你不小心操作失误，会以为这些提交已经永久消失了而吓出一身冷汗。

但这些提交并没有真正地被删除，如上所说，Git 并不会修改（或删除）原来的提交，而是重新创建了一批新的提交，并将当前分支顶端指向了新提交。因此我们可以使用 `git reflog` 找到并且重新指向原来的提交来恢复它们，这会撤销整个 `rebase`。感谢 Git ，即使你执行 `rebase` 或者 `commit --amend` 等重写提交历史的操作，它也不会真正地丢失任何提交。

### `git reflog` 命令

reflogs 是 Git 用来记录本地仓库分支顶端的更新的一种机制，它会记录所有分支顶端曾经指向过的提交，因此 reflogs 允许我们找到并切换到一个当前没有被任何分支或标签引用的提交。

每当分支顶端由于任何原因被更新（通过切换分支、拉取新的变更、重写历史或者添加新的提交），一条新的记录将被添加到 reflogs 中。如此一来，我们在本地所创建过的每一次提交都一定会被记录在 reflogs 中。即使在重写了提交历史之后， reflogs 也会包含关于分支的旧状态的信息，并允许我们在需要时恢复到该状态。

注意 reflogs 并不会永久保存，它有 90 天的过期时间。

### 还原提交历史

我们从上一个例子继续，假设我们想恢复 `feature` 分支在 `rebase` 之前的 A→B→C→D→E→F 提交历史，但这时候的 `git log` 中已经没有后面 5 个提交，所以需要从 reflogs 中寻找，运行 `git reflog` 结果如下:

```bash
64710dc (HEAD -> feature) HEAD@{0}: rebase (continue) (finish): returning to refs/heads/feature
64710dc (HEAD -> feature) HEAD@{1}: rebase (continue): commit H
8ab4506 HEAD@{2}: rebase (continue): commit G
1e186f8 HEAD@{3}: rebase (squash): commit C
c50221f HEAD@{4}: rebase (start): checkout ef1372522cdad136ce7e6dc3e02aab4d6ad73f79
74199ce HEAD@{5}: checkout: moving from master to feature
......
```

`reflogs` 完整的记录了我们切换分支并进行 `rebase` 的全过程，继续向下检索，我们找到了从 `git log` 中消失的提交 F:

```bash
74199ce HEAD@{15}: commit: commit F
```

接下来我们通过 `git reset` 将 `feature` 分支的顶端重新指向原来的提交 F：

```bash
# 我们想将工作区中的文件也一并还原，因此使用了--hard选项   
$ git reset --hard 74199ce                                      
HEAD 现在位于 74199ce commit F
```

再运行 `git log` 会发现一切又回到了从前：

```bash
74199cebdd34d107bb67b6da5533a2e405f4c330 (HEAD -> feature) commit F
e7c7111d807c1d5209b97a9c75b09da5cd2810d4 commit E
d9623b0ef9d722b4a83d58a334e1ce85545ea524 commit D
73deeedaa944ef459b17d42601677c2fcc4c4703 commit C
c50221f93a39f3474ac59228d69732402556c93b commit B
ef1372522cdad136ce7e6dc3e02aab4d6ad73f79 commit A
```


> **参考链接**
> [*git rebase - Atlassian Git Tutorial*](https://www.atlassian.com/git/tutorials/rewriting-history/git-rebase)
> [*git amend - Atlassian Git Tutorial*](https://www.atlassian.com/git/tutorials/rewriting-history)
> [*Git Pull - Atlassian Git Tutorial*](https://www.atlassian.com/git/tutorials/syncing/git-pull)
> [*Git - 变基*](https://git-scm.com/book/zh/v2/Git-分支-变基)
> [*Git - git-rebase Documentation*](https://git-scm.com/docs/git-rebase)
> [*Git - git-reflog Documentation*](https://git-scm.com/docs/git-reflog)
> [*Why you should stop using Git rebase*](https://medium.com/@fredrikmorken/why-you-should-stop-using-git-rebase-5552bee4fed1)
> [*Understand how does git rebase work and compare with git merge and git interactive rebase*](https://medium.com/@fabisiakradoslaw/understand-how-does-git-rebase-work-and-compare-with-git-merge-and-git-interactive-rebase-cce2c9775e43)
