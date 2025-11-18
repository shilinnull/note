## 创建分支
+ 查看当前本地仓库中有哪些分支

```bash
git branch
```

+ HEAD所指向的分支就是当前正在工作的分支

```bash
cat .git/HEAD
```

![](./Git分支管理.assets/1746089614035-bf516c86-8e5d-450e-9a02-5b52dd486b31.png)

+ 创建一个分支

```bash
git branch dev
```

创建好了，但是目前还是指向`master`

![](./Git分支管理.assets/1746089614017-3f19dc0b-3735-466e-bbf6-4bcf2dfcc379.png)  
用tree命令也可以看到已经创建分支成功了

![](./Git分支管理.assets/1746089614012-897394a3-cae9-4481-9065-42ee31bfa0f4.png)

创建出来的分支，和主分支的最新记录是一样的

![](./Git分支管理.assets/1746089614022-5a1521cb-ac3e-4543-9079-57c560ba37fc.png)

## 切换分支
切换分支就是让HEAD指向我们的dev分支

```bash
git checkout dev
```

![](./Git分支管理.assets/1746089614017-8e099992-2f5c-4bc0-9ef5-a285007287d2.png)

我们在dev分支上堆ReadMe文件进行了修改

![](./Git分支管理.assets/1746089614307-6c4d2689-f9fa-4bbb-a701-05d21f595459.png)

再进行提交

![](./Git分支管理.assets/1746089614358-c48825e1-2d17-44e1-a13d-28249a7fca22.png)

这个时候再切换回master分支  
查看文件

![](./Git分支管理.assets/1746089614393-cc146489-8c3f-46f0-8f4c-245d411ba222.png)

可以看到刚刚新加的那行文件不见了

那我们再切换回dev分支上看

![](./Git分支管理.assets/1746089614431-198f4c39-930d-4f3f-b6fc-a1de1ba4d0d6.png)  
发现那行新加的还在

我们查看这里发现已经变了  
![](./Git分支管理.assets/1746089614427-0e30a21e-1e70-4a40-9026-27e54ec98032.png)

我们查看记录  
dev上是最新的记录，master分支第二

![](./Git分支管理.assets/1746089614642-c81960b7-bc3b-4ade-9b0c-712056b9eef3.png)

我们最终的效果是在master分支上看到我们的效果，我们怎么操作呢？

## 合并分支
这就要我们合并分支，在合并分支之前就需要先切换到我们master分支上

```bash
git merge dev
```

Fast-forward 代表“快进模式”，也就是直接把master指向dev的当前提交，所以合并速度非常快。

![](./Git分支管理.assets/1746089614656-ddfa764c-6cd9-447e-aefd-e3e1325008b8.png)

## 删除分支
只能在其他的分支上删除本分支

```bash
git branch -d dev
```

![](./Git分支管理.assets/1746089614707-0602d49c-7f5c-4b73-9ded-f56e7facb373.png)



<font style="color:rgb(0, 0, 0);">删除指定的远程分支：</font>

```bash
git push <远程仓库名> --delete <分支名>
```

<font style="color:rgb(0, 0, 0);">通常远程仓库的默认名称是 </font>`<font style="color:rgba(0, 0, 0, 0.85) !important;">origin</font>`<font style="color:rgb(0, 0, 0);">，所以实际使用时命令一般是：</font>

```bash
git push origin --delete feature-branch
```

<font style="color:rgb(0, 0, 0);">这个命令会将指定的远程分支从远程仓库中删除。</font>

<font style="color:rgb(0, 0, 0);">删除后，其他协作者可以通过 </font>`<font style="color:rgba(0, 0, 0, 0.85) !important;">git fetch --prune</font>`<font style="color:rgb(0, 0, 0);"> 命令来更新本地的远程分支列表，移除已被删除的远程分支引用：</font>

```bash
git fetch --prune
# 或者简写为
git fetch -p
```



## 合并冲突
在合并分支的时候，我们在master分支上已经修改了文件，而我在dev分支上也修改了文件，然后合并的时候就会出现报错，我们来演示一下

快速创建分支并且进入分支

```bash
git checkout -b dev1
```

![](./Git分支管理.assets/1746089614922-2606be93-d621-414b-939f-08add7bdbe36.png)

我们将原来的aaa改成了bbb

![](./Git分支管理.assets/1746089614936-3514ba73-48dd-43fc-86ea-2c98260c26ad.png)

然后提交上去

![](./Git分支管理.assets/1746089615050-fc10fa50-e184-433c-88ff-1a16be8a66b5.png)

我们切换到master分支上查看一下文件内容，发现还是aaa，因为还没有合并

![](./Git分支管理.assets/1746089615205-325005b9-d103-46c5-a7dd-5f9e8a696302.png)  
![](./Git分支管理.assets/1746089615203-33eb2c39-fe2b-4560-90b0-bd749b9d2156.png)

接下来我们就继续将这个aaa改成ccc，然后再进行合并分支

![](./Git分支管理.assets/1746089615481-444c78de-98b6-4125-bb1d-92f24a7860ab.png)

![](./Git分支管理.assets/1746089615517-cb7802dd-f6b8-4ae1-aea5-66e1657bddc9.png)  
这个时候再进行合并，会提示合并冲突

```bash
git merge dev1
```

![](./Git分支管理.assets/1746089615613-f14dcfda-3367-4ab9-960a-72afac9ea274.png)

我们打开ReadMe文件查看一下

![](./Git分支管理.assets/1746089615681-51d82ec4-4cee-46d8-872b-717557a35a1e.png)

这个时候就要手动选择要保留哪些代码

假设我们就保留这些代码

![](./Git分支管理.assets/1746089615711-14f1fda8-8bd4-4ee8-adaa-c438f49371af.png)

然后再进行提交

![](./Git分支管理.assets/1746089615981-0bc37c18-7f2a-41b6-88a7-ca5086f21e26.png)  
查看是否是最新提交

![](./Git分支管理.assets/1746089616055-d9def917-3dc0-408b-a9fa-bb4030735f1f.png)

可视化的查看方法

```bash
git log --graph --abbrev-commit
```

![](./Git分支管理.assets/1746089616055-2fd31259-8244-4aca-92e9-e282400024a2.png)

## 分支管理策略
在Fast forward 模式下，删除分支后，查看分支历史时，会丢掉分支信息，看不出来最新提交到底是 merge 进来的还是正常提交的。

![](./Git分支管理.assets/1747527619287-c5184715-0a80-4f61-822a-76ed74f742f9.png)

不使用`Fast forward`模式，这样的好处是，从分支历史上就可以看出分支信息。

![](./Git分支管理.assets/1747527624237-188b35a0-016b-45db-89f7-8e2432c4e9bc.png)

创建一个新分支

```bash
git checkout -b dev2
```

![](./Git分支管理.assets/1746089616402-18e4348d-339b-4fc2-8d23-13fc16c6a9a9.png)

修改ReadMe文件，并提交

![](./Git分支管理.assets/1746089616243-4562817a-c9ff-41e7-a6f7-689c818ed49d.png)

![](./Git分支管理.assets/1746089616472-f88bbfb6-86e4-4292-9a41-ec31f97a8fca.png)

切换回master分支后进行合并

不使用`Fast forward`模式，并且指向新的提交，禁用`Fast forward`模式后合并会创建一个新的commit ，所以加上`-m`参数，把描述写进去。

```bash
git merge --no-ff -m "merge with no-ff" dev2
```

![](./Git分支管理.assets/1746089616512-1afa588f-8f44-4eeb-8958-11884596842d.png)

## bug分支
假如我们现在正在 dev2 分支上进行开发，开发到一半，突然发现master分支上面有 bug，需要解决。在Git中，每个 bug 都可以通过一个新的临时分支来修复，修复后，合并分支，然后将临时分支删除。

![](./Git分支管理.assets/1746089616573-ed67021d-ef67-42a7-8719-7bdb5f917db1.png)

+ 这个时候主分支出现了一个bug，这个时候就要切换到master分支

![](./Git分支管理.assets/1746089616740-a54ec227-e842-48dd-a563-d1476f815a87.png)

那我们不想这样，我们可以这样做

将工作区的内容进行保存

```bash
git stash
```

![](./Git分支管理.assets/1746089616930-18d21dca-7ab0-4820-bcd3-401f653e9b4e.png)

修复完bug后，我们就需要进行重新回到dev2分支继续开发

你可以多次`stash`，恢复的时候，先用`git stash list`查看，然后恢复指定的stash，用命令`git stash apply stash@{0}`

```bash
git stash pop
```

恢复的话也可以采用`git stash apply`恢复，但是恢复后，stash内容并不删除，需要用`git stash drop`来删除

![](./Git分支管理.assets/1746089617033-5952b2ec-2121-445e-a084-f8beb0277c2a.png)  
现在我们到了dev2分支上了，我们继续开发

![](./Git分支管理.assets/1746089617002-2aea8ad4-0c4d-4307-ac22-40caf38c7711.png)  
然后提交，在dev分支上进行了新的提交

![](./Git分支管理.assets/1746089617021-692b019c-c50f-4cfe-94d0-07ed274061b2.png)

这个时候就需要合并了，但是合并的时候就会出现冲突，刚刚master修改了bug了，这次又要进行合并分支，我们需要解决错误。

我们需要不在master上合并分支，在dev合并master主分支，把问题在本地上解决了再做下一步。

![](./Git分支管理.assets/1747528051995-991d638f-b022-4062-a5b0-2f316c3ba38f.png)

![](./Git分支管理.assets/1747528055866-5c95f63e-b13f-4e10-80ef-8880d9c74af4.png)

![](./Git分支管理.assets/1747528061714-4a32c509-4aba-49fb-bbc3-8987dc27d611.png)

**我们在dev2分支上进行合并master**  
![](./Git分支管理.assets/1746089617346-5b12d6c4-105e-41b4-964b-5b26d1c3e4a1.png)

手动修改冲突  
![](./Git分支管理.assets/1746089617499-0c758907-9a89-49d6-a39d-8e8620685b9e.png)

然后就可以合并了

![](./Git分支管理.assets/1746089617447-747b2e44-f667-4ed1-add3-6884dbeab57c.png)

最后不要忘了，把刚刚的临时分支和开发分支删除

![](./Git分支管理.assets/1746089617460-0e17e30e-b55c-4006-997f-686f723d4694.png)

## 强制删除分支
如果在开发中如果在一个分支上已经开发，对代码进行提交了，这个时候用传统的方法进行删除是不能删除的，我们需要用到`-D`来进行强制删除

```bash
git branch -D dev3
```

