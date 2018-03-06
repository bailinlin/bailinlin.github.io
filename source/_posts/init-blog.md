---
title: 使用 hexo 搭建你的 github.io 博客网站
date: 2016-06-30 20:37:03
tags: [hexo]
---
作为前端er,我们每天都要保持学习,前阵子有一片文章很火[在 2016 年学 JavaScript 是一种什么样的体验？](https://zhuanlan.zhihu.com/p/22782487),前端技术日新月异,我们不仅要保持学习,还需要掌握一个好的,适合自己的学习方式.在这个信息这个发达的时代,你不缺输入的机会,却少的反而是我们的输出机会.大多数的前端没有去记录自己的所学,所用,即使有记录,也都是在一些第三方的博客网站, 这就会有这么一个情况.持续性不够,发布过的文章没有去回顾,这个博客网站发一篇,那个博客网站发一篇,而且很没归属感.

然而,github 作为我们的程序员圈子的同性交友社区,面试必看的内容,我们为什么不在 github 上搭一个自己的博客呢,在 github 上你可以直接建立一个仓库,然后在 issue 里写博客,我个人比较建议的是,搭一个属于你自己的 github pages,在 github pages 上记录你的博客. 首先是外观上比较好看,而且你可以依照你的心情改变你的博客主题,其次,你可以通过你的 github 用户名点 github.io 的方式来直接访问你的博客网站,感觉有了一个属于自己的网站一样,不要钱,还在一定程度上满足了你的虚荣心.
## 快速搭建

如果你是有一定开发经验的前端,你的电脑一定已经安装了 git 和 node ,那你在搭建博客之前你只要先全局安装 hexo 的命令行工具,命令如下

``` bash
$ npm install -g hexo-cli
```
然后你在你的电脑里新建一个空的文件夹命名就按照 userName.github.io 的方式来命名 例如我的项目我命名为 bailinlin.github.io, 其次,你需要进入你新建的这个文件夹,执行命令

``` bash
$ hexo init
```

等待一定时间,你会看到你新建的文件夹里出现了好多初始化文件,现在这个时候一个基本的简单博客搭建成功了,你运行以下命令

 ``` bash
 $ hexo s -p 3000
 ```

 就能在 localhost:3000 中看到一个博客网站了,现在你要做的就是为你的博客网站做一些个性化的定制,打开_config.yml 文件.修改文件内的参数,

     ``` bash
     title: 白霸天的博客              //博客网站的title
     subtitle:                      //子标题
     description: 前端,白霸天,博客   //网站描述
     author: bailinlin             //网站作者
     language: zh-Hans            //网站语言,用于本地化
     ...                          // 等其它配置
     ```

## 发布博客

### 关联远程仓库

博客搭建好之后,你需要做的是把博客发布到 github 上,首先你需要在 github 上新建一个仓库,命名和你新建的文件夹一样, username.githhub.io 值得注意的是,这里的 username 必须和你的github 用户名一样,大小写也要一样.然后你需要进入你的本地文件夹,关联你的远程仓库.

首先
    ```bash
    $ git init
    ```
然后
    ``` bash
    $ git remote add origin git@github.com:username/username.github.io.git
    ```

然后我们只要把代码推到远程仓库就行了,不过这里我们要有个约定,我们在 master 上发布博客,在 dev 分支上修改我们的博客内容和项目配置,也就是说我们发布博客后,master 分支上就是我们的博客的静态文件,dev 分支上的代码就是我们世纪维护的博客内容,如图
master 分支:
![master分支](/images/blog/master.png)
dev 分支:
![master分支](/images/blog/master.png)
### 安装 hexo 的一键部署命令工具

 通过npm 安装
 ``` bash
 $ npm install hexo-deployer-git --save
 ```

 修改部署配置,在 _config.yml 中添加如下配置

 ``` bash
 deploy:
   type: git
   branch: master
   repo: git@github.com:bailinlin/bailinlin.github.io.git  //此处记得改成你的 github 仓库的地址
 ```

 ### 你的 first commit

按照约定我们先切分支,通过命令把项目切换到 dev 分支
    ``` bash
    $ git checkout -b dev
    ```
然后 add 你的项目代码
    ``` bash
    $ git add .
    ```
你的 first commit
    ```bash
    $ git commit -m ":tada: init blog"
    ```
然后 push 你的代码到远程
    ```bash
    $  git push --set-upstream origin dev
    ```
### 你的第一次博客发布
发布其实很简单的,我们之前也说是一个命令行发布的,所以你只要
    ```bash
    $ hexo deploy
    ```
然后你就可以看到一个属于你的 github.io 的网站了

## 改变你的博客网站主题
个人比较推荐的是 next ,大量的留白,让你的博客看起来简洁大方,或则你喜欢其它的主题也行,你需要在你的博客文件夹里clone 下你需要的主题

```bash
$ git clone https://github.com/iissnan/hexo-theme-next themes/next
```
 然后在 _config.yml 文件夹里修改你的主题配置

 ```bash
 $ theme: next
 ```
 然后 hexo s 就能看到你修改的主题生效了,为了防止 clone 下来的主题文件和自己的博客文件冲突,我删除了clone 下来的 .git 文件.你再按照你的 first commit 的操作提交你的修改就行了,然后别忘记了 deploy

