### 探索GIT

* 什么是git
* git和svn等传统vcs的不同之处
* git的重要概念和操作分类总结
* 尝试github

#### 什么是git？

只要你经常浏览技术类文章，或者关注开源社区和软件的话，一定会注意到git这个新兴的版本管理软件。随着SourceForge和GoogleCode的相继没落，github这两年来后来居上，已经牢牢占领了开源软件托管库的宝座。而github只为git而生，也是综合考虑了这么多年来开源社区的需求发展而推出的一个综合性的代码托管平台。

从热门开源项目角度来分析（比如php，linux等），都是分成核心开发团队和贡献者两类的。一般核心开发团队准入很难，需要严格的小组成员或者创始人审批才可以，他们可以对源代码有最终决定权。贡献者则没有门槛，可以根据自己在使用软件过程中发现的问题，或者协助修改已知问题，来提交补丁(patch)，但是需要核心开发小组的成员进行审批。在早期的sourceforge，googlecode等使用svn，cvs的时代，修改patch是要提交到相关的bug跟踪系统或者发邮件列表的，等待核心组审核。如果可以通过的话，会人工进行采纳(apply/merge)到相关版本。如图：

![cvcs](https://github.com/yangshiqi/wiki/blob/master/imgs/git/cvcs-model.png)


"采用传统的集中式版本控制系统（如SVN）的开源项目，这两个群体的用户体验都不是太好。项目的贡献者（非核心成员）很不“高兴”，因为他们即便有修改源代码的能力和渴望，也不能直接向版本库提交，要想成为提交者需要一个很长的建立信任的过程。然而即便是核心开发团队的成员，体验也不是太好，因为凡是涉及到版本库的操作（检入、检出、查看日志等）都需要在联网的状态下进行，网络带宽对用户体验影响相当大。

Git等分布式版本控制系统的出现，彻底颠覆了原有代码管理的组织模式。使用Git，不再依赖唯一的、集中式的版本库，而是每个开发者本地都拥有一份完整的版本库。Git并不排斥集中式的使用模式，但更倾向于将集中式版本库称为共享版本库。核心开发团队的成员和贡献者（非核心成员）都可以从共享版本库克隆一份本地版本库，但只有核心团队成员才可以将自己本地版本库的提交推送到共享版本库上。" -- 《GotGitHub》

如图：

![dvcs](https://github.com/yangshiqi/wiki/blob/master/imgs/git/dvcs-model.png)

"核心开发团队非常“高兴”，因为他们和共享版本库之间不必一直保持连接状态，诸如查看日志、提交、创建分支等几乎全部操作都（脱离网络）在本地的版本库中完成。项目贡献者（非核心成员）也不再那么沮丧，因为版本库人人皆可更改（当然是对本地版本库而言）。稍微让贡献者感到困难的就是如何将自己对项目的改进被核心开发团队所了解并接纳。Git提供了多种途径，一个方法是先用git format-patch命令将本地提交转换为补丁文件或补丁文件序列，再通过邮件发送给核心开发团队。另外一个办法就是搭建一个自己专有的共享版本库，通过邮件创建一个拉拽请求（Pull Request），让核心团队的开发者到自己的版本库来抓取（Pull）。

GitHub的出现进一步推动了Git的普及，简化了版本控制的管理和操作流程，为开发者提供了更好的交流平台。"  -- 《GotGitHub》

如图：

![github-model](https://github.com/yangshiqi/wiki/blob/master/imgs/git/github-model.png)

"使用GitHub，无论是项目的核心开发团队，还是普通的项目贡献者都工作得非常“愉快”。创建项目变得非常轻松，创建者只需在GitHub上点击一下鼠标即可创建一个新版本库，通过简单的Web操作即可完成项目授权进而组建项目核心团队。在GitHub中，非核心团队成员参与项目也很容易。先找到自己希望参与的项目，然后只需在Web上点击一下鼠标即可在自己的托管空间下创建一个派生（fork）的项目，并对派生项目的版本库具有读写的完全权限，就好像这个项目原本就是由自己创立的那样。当贡献者完成开发并向自己派生的版本库推送后，可以通过GitHub的Web界面向项目的核心开发团队发送一个Pull Request，请求审核。项目的核心团队收到Pull Request后审核代码，审核通过后可以直接通过Web界面执行合并操作接纳贡献者的提交。" -- 《GotGitHub》

以上引用了《GotGitHub》这本书中的介绍，我觉得非常贴切的描述了开源世界的一个vcs发展过程。程序员更是痛苦驱动的，有什么不爽都会想办法去解决。github在开源世界中做出了巨大的贡献，并且延展出了基于源代码的新社交，有兴趣的人建议去读一下这本书，对于你上手github来说非常有帮助，且就看这一本就够了。




#### git和svn等传统vcs的不同之处

[vcs定义在此](http://zh.wikipedia.org/wiki/%E7%89%88%E6%9C%AC%E6%8E%A7%E5%88%B6)，简单来说就是版本控制系统。

刚开始的时候，我也为git无外乎就是一个可以在本地暂时提交代码的工具，在无网络的情况下，可以在本地随意提交，待有网络的时候在远端进行集成，也就是方便了开发者在任意地点可以进行代码修改和提交工作。但是，随着了解的深入我发现我错了，完全没有理解git的精髓。

“Git 和其他版本控制系统的主要差别在于，Git 只关心文件数据的整体是否发生变化，而大多数其他系统则只关心文件内容的具体差异。这类系统（CVS，Subversion，Perforce，Bazaar 等等）每次记录有哪些文件作了更新，以及都更新了哪些行的什么内容，如图：

![cvcs原理](https://github.com/yangshiqi/wiki/blob/master/imgs/git/18333fig0104-tn.png)

Git 并不保存这些前后变化的差异数据。实际上，Git 更像是把变化的文件作快照后，记录在一个微型的文件系统中。每次提交更新时，它会纵览一遍所有文件的指纹信息并对文件作一快照，然后保存一个指向这次快照的索引。为提高性能，若文件没有变化，Git 不会再次保存，而只对上次保存的快照作一链接。Git 的工作方式如图：

![git原理](https://github.com/yangshiqi/wiki/blob/master/imgs/git/18333fig0105-tn.png)

这是 Git 同其他系统的重要区别。它完全颠覆了传统版本控制的套路，并对各个环节的实现方式作了新的设计。Git 更像是个小型的文件系统，但它同时还提供了许多以此为基础的超强工具，而不只是一个简单的 VCS。" -- 《Pro Git》

git的绝大部分操作都是本地操作，非常快，这和传统的cvs/svn等完全相反，因为传统的vcs几乎所有操作都需要网络支持，而且受限于本地和源之间的网络情况。这也是为什么git和github一出来，就受到了全世界开源社区的极大支持，大家都不必受限于网络到美国的距离了，随意提交变更到本地、diff版本、切分支、合并等等所有操作，无须等待，待网络好的时候，再一次性进行push。

在传统的vcs中，如果还未提交到远端，则有可能丢失修改内容。而git内部采用了快照的方式，确保提交和传输过程中数据的完整性和可靠性。这种新的开发方式让程序员的工作方法做出了极大改变，大家不再担心数据的安全性问题，可以放心大胆去在本地做操作了。


#### git的重要概念和操作分类总结

正如上述git和传统vcs的不同，这种新的工作方式也引入了新的概念，对一个文件来说，在git内部有三种状态：

“已提交（committed），已修改（modified）和已暂存（staged）。已提交表示该文件已经被安全地保存在本地数据库中了；已修改表示修改了某个文件，但还没有提交保存；已暂存表示把已修改的文件放在下次提交时要保存的清单中。

由此我们看到 Git 管理项目时，文件流转的三个工作区域：Git 的工作目录，暂存区域，以及本地仓库。如图：

![git工作目录](https://github.com/yangshiqi/wiki/blob/master/imgs/git/18333fig0106-tn.png)

从项目中取出某个版本的所有文件和目录，用以开始后续工作的叫做工作目录。这些文件实际上都是从 Git 目录中的压缩对象数据库中提取出来的，接下来就可以在工作目录中对这些文件进行编辑。

所谓的暂存区域只不过是个简单的文件，一般都放在 Git 目录中。有时候人们会把这个文件叫做索引文件，不过标准说法还是叫暂存区域。

基本的 Git 工作流程如下：

在工作目录中修改某些文件。
对修改后的文件进行快照，然后保存到暂存区域。
提交更新，将保存在暂存区域的文件快照永久转储到 Git 目录中。
所以，我们可以从文件所处的位置来判断状态：如果是 Git 目录中保存着的特定版本文件，就属于已提交状态；如果作了修改并已放入暂存区域，就属于已暂存状态；如果自上次取出后，作了修改但还没有放到暂存区域，就是已修改状态。到第二章的时候，我们会进一步了解其中细节，并学会如何根据文件状态实施后续操作，以及怎样跳过暂存直接提交。” -- 《Pro Git》


简单来说：
	
	git status #查看本地修改情况
	git add file.name #添加文件
	git commit -m "说明文字"
	git push #将本地修改推送到远端



再一次，引用另外一篇介绍git的文章，方便快速检索和使用：

##### 新建仓库


* git init#初始化
* git status#获取状态
* git add file#.或*代表全部添加
* git commit -m "message"#此处注意乱码
* git remote add origin git@github.com:yanhaijing/test.git#添加源
* git push -u origin master#push同事设置默认跟踪分支

##### 从现有仓库克隆

* git clone git://github.com/yanhaijing/data.js.git      
* git clone git://github.com/schacon/grit.git mypro#克隆到自定义文件夹

##### 本地


* git add *#跟踪新文件
* rm *&git rm *#移除文件
* git rm -f *#移除文件
* git rm --cached *#取消跟踪
* git mv file_from file_to#重命名跟踪文件
* git log#查看提交记录
* git commit#提交更新
* git commit -m 'message'
* git commit -a#跳过使用暂存区域，把所有已经跟踪过的文件暂存起来一并提交
* git commit --amend#修改最后一次提交
* git reset HEAD *#取消已经暂存的文件
* git checkout -- file#取消对文件的修改（从暂存区去除file）
* git checkout branch|tag|commit -- file_name#从仓库取出file覆盖当前分支
* git checkout -- .#从暂存区去除文件覆盖工作区

##### 分支

* git branch#列出本地分支
* git branch -r#列出远端分支
* git branch -a#列出所有分支
* git branch -v#查看各个分支最后一个提交对象的信息
* git branch --merge#查看已经合并到当前分支的分支
* git branch --no-merge#查看为合并到当前分支的分支
* git branch test#新建test分支
* git checkout test#切换到test分支
* git checkout -b test#新建+切换到test分支
* git checkout -b test dev#基于dev新建test分支，并切换
* git branch -d test#删除test分支
* git branch -D test#强制删除test分支
* git merge test#将test分支合并到当前分支
* git rebase master#将master分之上超前的提交，变基到当前分支

##### 远端

* git fetch originname branchname#拉去远端上指定分支
* git merge originname branchname#合并远端上指定分支
* git push originname branchname#推送到远端上指定分支
* git push originname localbranch:serverbranch#推送到远端上指定分支
* git checkout -b test origin/dev#基于远端dev新建test分支
* git push origin :server#删除远端分支

##### 源

git是一个分布式代码管理工具，所以可以支持多个仓库，在git里，服务器上的仓库在本地称之为remote。

个人开发时，多源用的可能不多，但多源其实非常有用。

* git remote add origin1 git@github.com:yanhaijing/data.js.git
* git remote#显示全部源
* git remote -v#显示全部源+详细信息
* git remote rename origin1 origin2#重命名
* git remote rm origin1#删除
* git remote show origin1#查看指定源的全部信息

##### 标签

当开发到一定阶段时，给程序打标签是非常棒的功能。

* git tag#列出现有标签        
* git tag v0.1#新建标签
* git tag -a v0.1 -m 'my version 1.4'#新建带注释标签
* git checkout tagname#切换到标签
* git push origin v1.5#推送分支到源上
* git push origin --tags#一次性推送所有分支
* git tag -d v0.1#删除标签
* git push origin :refs/tags/v0.1#删除远程标签

##### 其它

* git help *#获取命令的帮助信息 
* git status#获取当前的状态，非常有用，因为git会提示接下来的能做的事情




#### 尝试github

关于github，请重点参考[《GotGitHub》](http://www.worldhello.net/gotgithub/index.html)，这边只提一下关于ssh认证的问题。

首先要注册GitHub.com，如：you@email.com

然后在本机生成秘钥：

	ssh-keygen -C "you@email.com" #生成密钥
	cat ~/.ssh/id_rsa.pub| pbcopy #copy公钥
	
再粘贴到GitHub的SSH公钥管理的对话框中，如图：

![setting-github](https://github.com/yangshiqi/wiki/blob/master/imgs/git/setting-ssh-gotgithub.png)

最后进行认证测试：

	ssh -T git@github.com #进行测试
	返回：Hi xxx! You've successfully authenticated, but GitHub does not provide shell access.
	
如果提示失败，请检查：

在git目录下查看当前推送方法：

$ git remote -v

origin https://github.com/youraccount/yourproject.git (fetch) 
origin https://github.com/youraccount/yourproject.git (push)

修改:

git remote set-url origin git@github.com:youraccount/yourproject.git


附录：

* https://try.github.io/levels/1/challenges/1  -- Git互动教程
* http://git-scm.com/book/zh/v1  -- 《Pro Git》
* http://www.worldhello.net/ -- 《GotGitHub》
