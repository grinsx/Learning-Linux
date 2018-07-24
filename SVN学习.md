### SVN学习
Subversion(SVN) 是一个开源的版本控制系統, 也就是说 Subversion 管理着随时间改变的数据。 这些数据放置在一个中央资料档案库(repository) 中。 这个档案库很像一个普通的文件服务器, 不过它会记住每一次文件的变动。 这样你就可以把档案恢复到旧的版本, 或是浏览文件的变动历史。

#### SVN的一些概念
* repository(源代码库)：源代码统一存放的地方
* checkout(提取)：当你手上没有源代码的时候，你需要从repository checkout一份
* commit(提交)：当你已经修改了代码，你就需要commit到repository
* update(更新)：当你已经checkout了一份源码，update一下你就可以和repository上的源代码保持同步，你手上的代码就会有新的变更。

#### SVN的生命周期
####创建版本库
版本库相当于一个集中的空间，用于存放开发者所有的工作成果。版本库不仅能存放文件，还包括了每次修改的历史，机每个文件的变动历史。

create操作是用来创建一个新的版本库。大多数情况下这个操作只会执行一次。当你创建一个新的版本库的时候，你的版本控制系统会让你提供一些信息来标识版本库，例如创建的位置和版本库的名字。

#### 复查变化
当你检出工作副本或者更新工作副本后，你的工作副本就跟版本库完全同步了。但是当你对工作副本进行一些修改之后，你的工作副本会比版本库要新。在 commit 操作之前复查下你的修改是一个很好的习惯。

status操作列出了工作副本中所进行的变动。正如我们之前提到的，你对工作副本的任何改动都会成为待变更列表的一部分。Status 操作就是用来查看这个待变更列表。

Status 操作只是提供了一个变动列表，但并不提供变动的详细信息。你可以用 diff 操作来查看这些变动的详细信息。

#### 修复错误
我们来假设你对工作副本做了许多修改，但是现在你不想要这些修改了，这时候 revert 操作将会帮助你。

Revert 操作重置了对工作副本的修改。它可以重置一个或多个文件/目录。当然它也可以重置整个工作副本。在这种情况下，revert 操作将会销毁待变更列表并将工作副本恢复到原始状态。

#### 解决冲突
合并的时候可能会发生冲突。Merge 操作会自动处理可以安全合并的东西。其它的会被当做冲突。例如，"hello.c" 文件在一个分支上被修改，在另一个分支上被删除了。这种情况就需要人为处理。Resolve 操作就是用来帮助用户找出冲突并告诉版本库如何处理这些冲突。

#### 分支
分支其实就是trunk版(主干线)的一个copy版，不过分支也是具有版本控制功能的，而且是和主干线相互独立的，当然，到最后我们可以通过(合并)功能，将分支合并到trunk上来，从而最后合并为一个项目。

在本地副本中创建一个my_branch分支：svn copy trunk/    branches/my_branch
![svn_copy](https://raw.githubusercontent.com/lengender/MarkdownPhotos/master/svn_copy.png)

提交新增的分支到版本库: svn commit -m "add my_branch"
![svn分支提交](https://raw.githubusercontent.com/lengender/MarkdownPhotos/master/svn%E5%88%86%E6%94%AF%E6%8F%90%E4%BA%A4.png)

接着到my_branch分支进行开发，切换到分支并创建index.html文件：
![分支开发](https://raw.githubusercontent.com/lengender/MarkdownPhotos/master/%E5%88%86%E6%94%AF%E5%BC%80%E5%8F%91.png)

切换到trunk，执行svn update，然后将my_branch分支合并到trunk中：svn merge ../branches/my_branch
![分支合并](https://raw.githubusercontent.com/lengender/MarkdownPhotos/master/%E5%88%86%E6%94%AF%E5%90%88%E5%B9%B6.png)

将合并好的 trunk 提交到版本库中：
![trunk提交](https://raw.githubusercontent.com/lengender/MarkdownPhotos/master/trunk%E6%8F%90%E4%BA%A4.png)