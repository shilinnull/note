使用windows和linux分别模拟开发者：

## 多人协作情况一
在windows环境下，模拟另外一个人来开发，将仓库clone下来

windows下：

![](./Git多人协作.assets/1747540753629-1bce4037-dc05-4f72-8a3a-022ad204801d.png)

Linux下：

![](./Git多人协作.assets/1747540988989-5a2f07bd-8ee8-4f7f-92bc-4b5b9db6f813.png)

在实际开发中，如果要多人进行协同开发，必须要将用户添加进开发者，用户才有权限进行代码提交

![](./Git多人协作.assets/1747540775353-a0f96ee0-23f2-4813-8116-6507aeeecadb.png)

邀请用户：

![](./Git多人协作.assets/1747540866067-6b0886cd-e74c-47da-a91f-f40cb243a621.png)



现在相当于在我的linux机器上一个用户，windows上一个用户。

目前，我们的仓库中只有一个 master 主分支，但在实际的项目开发中，在任何情况下其实都是不允许直接在 master 分支上修改代码的，这是为了保证主分支的稳定。所以在开发新功能时，常常会新建其他分支，供开发时进行迭代使用。

![](./Git多人协作.assets/1747541467580-498f8cc3-4dea-4d1e-98bf-eee7759af142.png)

创建成功的远程分支是可以通过 Git 拉取到本地来，以实现完成本地开发工作。

使用`git branch -r`来查看远程分支

![](./Git多人协作.assets/1747541680164-4c0a7d9a-0ad6-4d70-a703-dd31d558e2d3.png)

但是到这一步的话还需要进行本地分支和远程分支的关系链接

```shell
git checkout -b dev origin/dev
```

![](./Git多人协作.assets/1747541808746-9d057ca3-44f6-4597-8fb7-4e1b0301731e.png)

拉取后便可以看到远程的 dev 分支，接着切换到 dev 分支供我们进行本地开发。

---

首先让linux机器上进行第一次开发，并且推送到远程

![](./Git多人协作.assets/1747543947387-99ea2de5-7404-4095-be61-d6ffacc87bb8.png)

在码云上可以看到dev分支上已经有了记录了

![](./Git多人协作.assets/1747544002991-181a9cc8-03ce-4be7-909b-2f3425629c8a.png)

那么同时在windows上也要进行开发，也要对file.txt文件进行修改，并push，就会提示冲突

![](./Git多人协作.assets/1747544371547-5bc95ec4-c877-4a85-83eb-ecc8b3801c09.png)

解决办法也很简单，Git已经提示我们，先用`git pull`把最新的提交从`origin/dev`抓下来，然后在本地进行合并，并解决冲突，再推送。

![](./Git多人协作.assets/1747544496691-5be052f9-bcbf-4d7d-beb2-73a3b73de22e.png)

在解决后，重新push

![](./Git多人协作.assets/1747544630396-093a04d1-c184-4639-bacf-66da967d6b72.png)

在码云上就也可以看到了

![](./Git多人协作.assets/1747544683166-967444f7-4945-472e-addc-133ff7e5b11c.png)

所以两名开发者已经开始可以进行协同开发了，不断的`git pull/add/commit/push`。

最终的目的是要将开发后的代码合并到`master`上去，让我们的项目运行最新的代码。

---

在linux机器上进行拉去windows下写的代码，然后可以进行合并到master

切换至`master`分支, `pull`一下，保证本地的`master`是最新内容。

![](./Git多人协作.assets/1747544922108-cb2892b4-10ca-4b42-91ad-3a6ca55726cf.png)

再次切换至`dev`分支, 合并`master`分支，这么做是因为如果有冲突，可以在dev分支上进行处理，而不是在`master`上解决冲突，要不然就乱了。

![](./Git多人协作.assets/1747545271375-90f26d35-3242-4fbc-8b81-9e370cfc598b.png)

就可以看到master分支上就能看到file文件了

![](./Git多人协作.assets/1747545316297-beb75f0c-6940-4ba1-99da-e0ddbb732433.png)

此时dev分支的任务就完成了，可以直接在远程仓库中的dev分支进行删除

![](./Git多人协作.assets/1747545375753-533c8f31-6e54-4916-a942-cd6cbc4e2bca.png)

在本地可以使用`git branch -a`来查看远程分支，发现还是有dev的

<font style="color:rgb(77, 77, 77);">清除</font>`<font style="color:rgb(77, 77, 77);">remote</font>`<font style="color:rgb(77, 77, 77);">分支（清除 origin 已经不存在，但是</font>`<font style="color:rgb(77, 77, 77);">remote</font>`<font style="color:rgb(77, 77, 77);">还存在的分支）</font>

```shell
git remote prune origin
```

![](./Git多人协作.assets/1747545873693-99e0981b-62f7-42c2-9211-d9975305eb11.png)

那么我不想在远程上删除分支，可以使用下面命令进行删除远程分支

```shell
git push origin --delete [branchName]
```

或者这个

```shell
git push origin :[branchName]
```

![](./Git多人协作.assets/1747546087590-9480620a-c49a-412c-b59f-364dd20a5c43.png)

总结一下，在同一分支下进行多人协作的工作模式通常是这样：

1. 首先，可以试图用`git push origin branch-name`推送自己的修改；
2. 如果推送失败，则因为远程分支比你的本地更新，需要先用`git pull`试图合并；
3. 如果合并有冲突，则解决冲突，并在本地提交；
4. 没有冲突或者解决掉冲突后，再用`git push origin branch-name`推送就能成功！
5. 功能开发完毕，将分支`merge`进`master`，最后删除分支。

## 多人协作情况二
一般情况下，如果有多需求需要多人同时进行开发，是**不会在一个分支上进行多人开发**，而是一个需求或一个功能点就要创建一个feature分支。

在linux创建分支，再推送到远端

![](./Git多人协作.assets/1747546374873-3b37de47-9aa4-4cb7-a15b-1e8402c89669.png)

在windows创建分支，再推送到远端

![](./Git多人协作.assets/1747546620963-ee3ebb58-1a9d-4986-b65a-770b0f84f9e1.png)

此时，在本地，你看不见他新建的文档，他看不见你新建的文档。并且推送各自的分支时，并没有任何冲突，互不影响。

码云上就有那两个分支了，各个分支上有着自己管理的文件

![](./Git多人协作.assets/1747546683008-e142cdd8-c3db-4ab1-ba06-329ef8e1144c.png)

如果windows上的机器突然坏了，不能进行开发了，就需要切换到linux机器上进行继续开发

![](./Git多人协作.assets/1747546838453-76720186-3a17-4f27-87c4-8562d12a2835.png)

可以看到就有windows上创建的分支了

![](./Git多人协作.assets/1747546872429-ab1fe707-8637-4a12-b5a8-e83ecafc2a5d.png)

然后切换到windows上创建的分支，与远程的feature-2进行关联起来，否则将来使用`git push`推送内容会失败

使用`git branch -vv`查看关联情况

![](./Git多人协作.assets/1747546997906-1cca48ff-993d-4c2e-b753-84e6738b9045.png)

```shell
git checkout -b feature-2 origin/feature-2
```

![](./Git多人协作.assets/1747547050623-10908d3c-b443-4324-ae84-abd083c07f08.png)

对windows机器上面写的代码继续进行开发

![](./Git多人协作.assets/1747547241129-8b353056-321d-4b67-8735-4f1e90c0bbae.png)

查看码云

![](./Git多人协作.assets/1747547294264-d9b800e3-19b4-4e04-88a3-e2a617a8455a.png)

这个时候，当windows机器修好后，那么windows机器就需要先拉去我开发的代码

![](./Git多人协作.assets/1747547404312-b49a50a5-d63b-4d73-b078-d2c8532f7f36.png)

Pull 无效的原因是小伙伴没有指定本地`feature-2`分支与远程`origin/feature-2`分支的链接，根据提示，设置`feature-2`和`origin/feature-2`的链接即可：

```shell
git branch --set-upstream-to=origin/feature-2 feature-2
```

![](./Git多人协作.assets/1747547545073-7d1b473f-51f3-4548-b541-bab8ccd85238.png)

目前，小伙伴的本地代码和远端保持严格一致。你和你的小伙伴可以继续在不同的分支下进行协同开发了。

各自功能开发完毕后，不要忘记我们需要将代码合并到master中才算真正意义上的开发完毕。由于你的小伙伴率先开发完毕，于是开始 merge ：

![](./Git多人协作.assets/1747547896938-879e62a8-d172-4d9c-b076-8db04f75c7bf.png)

此时查看码云

![](./Git多人协作.assets/1747547956989-cf26f5a0-d279-497a-b020-24a0106f3dcd.png)

那么我也可以将linux机器上开发的代码也合并到master分支上，进行推送到远端，步骤和上面一样

![](./Git多人协作.assets/1747548368107-bb3f9a2e-ef20-4bae-b1fa-4ed5d24cce86.png)

![](./Git多人协作.assets/1747548423061-2c64a655-5859-4986-9e26-eb620d388459.png)

查看远端仓库

![](./Git多人协作.assets/1747548453475-4c9fb66f-11ac-4e9b-a1a7-4ac5d28205a5.png)

此时， feature-1 和feature-2 分支对于我们来说就没用了， 那么我们可以直接在远程仓库中将dev分支删除掉：

### 远程分支删除后，本地`git branch -a`依然能看到的解决办法
当前我们已经删除了远程的几个分支，使用`git branch -a`命令可以查看所有本地分支和远程分支，但发现很多在远程仓库已经删除的分支在本地依然可以看到。

![](./Git多人协作.assets/1747548547376-16667ed1-5283-4280-99a7-f13596a65c27.png)

使用命令`git remote show origin`，可以查看remote地址，远程分支，还有本地分支与之相对应关系等信息。

![](./Git多人协作.assets/1747548605697-8c01a0f0-004c-49dd-936a-71a91a583582.png)

此时我们可以看到那些远程仓库已经不存在的分支，根据提示，使用`git remote prune origin`命令

这样就删除了那些远程仓库不存在的分支

![](./Git多人协作.assets/1747548685180-5bc274e0-d40f-4c51-9653-e08d8c917fcb.png)

## Git 分支设计规范
对于开发人员来说，一般会针对不同的环境来设计分支

| 分支 | 名称 |  适用环境 |
| --- | --- | --- |
| master | 主分支 |  生产环境 |
| release | 预发布分支 | 预发布/测试环境 |
| develop | 开发分支 | 开发环境 |
| feature | 需求开发分支 | 本地 |
| hotfix | 紧急修复分支 | 本地 |


### master 分支
+ master 为主分支，该分支为只读且唯一分支。用于部署到正式发布环境，一般由合并release分支得到。
+ 主分支作为稳定的唯一代码库,任何情况下不允许直接在 master 分支上修改代码。
+ 产品的功能全部实现后，最终在master分支对外发布，另外所有在master分支的推送应该打标签（tag）做记录，方便追溯。
+ master 分支不可删除。

### release 分支
+ release 为预发布分支，基于本次上线所有的feature 分支合并到develop 分支之后，基于develop分支创建。可以部署到测试或预发布集群。
+ 命名以 release/ 开头，建议的命名规则：release/version_publishtime 。
+ release 分支主要用于提交给测试人员进行功能测试。发布提测阶段，会以release分支代码为基准进行提测。
+ 如果在 release 分支测试出问题，需要回归验证 develop 分支看否存在此问题。
+ release 分支属于临时分支，产品上线后可选删除。

### develop 分支
+ develop 为开发分支，基于master分支创建的只读且唯一分支，始终保持最新完成以及 bug修复后的代码。可部署到开发环境对应集群。
+ 可根据需求大小程度确定是由 feature 分支合并，还是直接在上面开发（非常不建议）。

### feature 分支
+ feature 分支通常为新功能或新特性开发分支，以develop 分支为基础创建feature 分支。
+ 命名以 feature/ 开头，建议的命名规则： feature/user_createtime_feature 。
+ 新特性或新功能开发完成后，开发人员需合到develop分支。
+ 一旦该需求发布上线，便将其删除。

###   hotfix 分支
+ hotfix 分支为线上 bug 修复分支或叫补丁分支，主要用于对线上的版本进行 bug 修复。当线上出现紧急问题需要马上修复时，需要基于master 分支创建 hotfix 分支。
+ 命名以 hotfix/ 开头，建议的命名规则: hotfix/user_createtime_hotfix
+ 当问题修复完成后，需要合并到master 分支和develop 分支并推送远程。一旦修复上线，便将其删除。

  ![](./Git多人协作.assets/1747549038890-07ef29fc-c894-4bc5-a784-5ca1890d4eb3.png)

上面是是企业级常用的一种 Git 分支设计规范：**Git Flow 模型**

