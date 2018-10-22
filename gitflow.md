# Git Flow 介绍

​	Gitflow工作流程围绕项目发布定义了严格的分支模型。尽管它比Feature Branch Workflow更复杂一些，但它也为管理更大规模的项目提供了坚实的框架。与Feature Branch Workflow比起来，Gitflow流程并没有增加任何新的概念或命令。其特色在于，它为不同的分支分配了非常明确的角色，并且定义了使用场景和用法。除了用于功能开发的分支，它还使用独立的分支进行发布前的准备、记录以及后期维护。当然，你还是能充分利用Feature Branch Workflow的好处：拉拽请求（Pull Request）、隔离的试验以及更高效率的合作。

Git flow是git的一个扩展集，它基于Vincent Driessen 的分支模型，文章*A successful Git branching model*对这一分支模型进行了描述，其示意图如下：

![gitflow示意图](https://img-blog.csdn.net/20150819223225341)

上图从左往右看，分别为 

- 时间轴，从上往下时间在流逝 
- feature分支(玫红)，图上有两个feature分支，在这个分支上，进行功能特性的开发 
- develop分支(黄色)，git flow的主分支，feature分支和release分支都会将代码合并到此分支上 
- release分支(绿色)，总是基于develop分支创建，最后合并到develop分支和master分支 
- hotfix分支(红色)，总是基于master分支创建，最后合并到master分支和develop分支 
- master分支(蓝色)，git flow的主分支，在开发的整个阶段一直存在，平时不在此分支开发，因此代码比较稳定，可以用来发布 

### 版本发布分支

master，用于保存正式的发布历史，打标签，所有版本发布在master上发布。

原则上每个月最后一周的周五发布版本，发布前1周打出release分支，在release上进行回归测试，问题全部修改完毕后，release合并到master，在master上发布，同时release合并到develop，将修改的bug合并进来。

版本发布后，不需要的feature、release全部删除。

### 补丁发布分支

补丁发布，现场发现bug后，基于master上最新版本打出分支hotfix（前提要求现场升级到最新版本），命名规则，hotfix+当前打出分支的版本号，如hotfix/v3.8.1，hotfix/v3.9.1.1，hotfix测试完毕后合并到master发布，同时合并到develop。

如果一个版本下有连续多个补丁，可以这些补丁公用一个hotfix，如果补丁不是连续多个，则每个版本打一个hotfix。

### 开发分支

feature，每个新功能or需求开发，都需要打一个feature分支，在feature上进行开发，测试完毕后合并到develop。

feature粒度，每个可以独立发布的功能建立一个feature，两个功能即使工作量非常小（比如1天就开发完的），也要建立两个featrue；一个功能即使工作量非常大（比如开发一类案件全流程），只需建立一个feature，此时可以多人同时开发一个feature。

feature命名规则，采用版本+功能名称方式，如：feature/v3.7.1-文书直报，feature/v3.8-久押未决。

feature合并，在feature上测试完毕后合并到develop，合并时间在打出release分支之前，在打release之后合并，只能放到下个版本发布。

### 测试分支

功能测试在feature上进行，feature完成需求验证、系统联调、初测、复测、并且修改完全部bug后，方可合并到develop，简单来说就是feature全部完成后才能合并到develop，合并后仅作简单的合并测试。

### 亮点

项目中分支结构是它的最大亮点：

（1）用于记录历史的分支

（2）用于功能开发的分支

（3）用于发布的分支

（4）用于维护的分支