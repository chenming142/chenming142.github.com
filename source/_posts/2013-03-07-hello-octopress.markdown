---
layout: post
title: "Hello Octopress,在windwow平台下自己动手搭建一个GitHub Blog!"
date: 2013-03-07 11:08
comments: true
categories: 开发工具库
tags: Octopress Github
---
> `github`本身不仅作为代码共享仓库，并且支持`github-pages`功能，而`octopress`就是在这样的环境下诞生的博客系统. 
     
#### 需要安装的软件
	1. git
	2. ruby(或者说rubyinstaller, 版本号须 >= 1.9.2)
	3. DevKit(windows平台下编译和使用本地C/C++扩展包的工具,即用来模拟Linux平台下的make,gcc,sh命令来进行编译)
	4. python(支持代码高亮) 

######关于 github 值得说说的事情
1. 先注册一个github账号和创建一个repos  
	创建一个新的Repository.这里必须重视,若想是博客首页是http://yourname.github.com,则Repositoty的project name就必须是__yourname.github.com__.
2. 设置SH Keys
	履行`ssh-keygen -t rsa -C “your_email＠youremail.com”`，回车;然后输入两遍暗码.
	到c:\Users\用户名.ssh\目次找到id_rsa.pub，并用文本软件打开复制全部(目录是隐藏的)
3. 将SSH Key添加到GitHub
	到github网站选择“Account Settings”>>“SSH Public Keys”>>“Add another public key”，将刚才复制的内容粘贴到key文本框内
4. 测试  
	为确保设置成功，现在可以测试设置成果啦。记住，”__git@github.com__”是默认的，无需修改  
		$ ssh -T git@github.com  （也可以用 SSH -v git@github.com ）
5. 设置个人信息   
	你已经成功安装Git，并设置好SSH Keys，现在设置个人信息：   
	设置用户名和电子邮件   
		$ git config --global user.name "Firstname Lastname"
		$ git config --global user.email "youremail@youremail.com"
如果你的~/目录下生成了 .gitconfig文件,即说明配置成功!你可以打开看看,是否是你配置的信息.OK,就说这么多了.

<!-- more -->

#### 搭建ruby环境和配置工具
依次安装`git`,`rubyinstaller`和`python`,建议最好安装在C盘(系统盘)下,并把相应路径加入系统环境变量中,以避免不必要的错误。之后进入cmd窗口或git窗口来安装DevKit,解压DevKit,比如C:/DevKit,然后执行如下命令:
	c:                          #进入c盘
	cd c:/DevKit  
    ruby -v                     #此命令为了验证 ruby 是否正确安装              
	ruby dk.rb init             #执行该命令时,可能出现编码错误!    
	                            #可能是你在系统环境变量中提前设置了LANG=zh_CN.UTF-8和LC_ALL=zh_CN.UTF-8   
                                #此时应该先删除这两个环境变量,再执行该命令        
                                #此处的编码问题还可以使用 chcp 命令查询`活动代码页`序号,936 : 简体中文 
	ruby dk.rb install
至此你都未出现任何错误的话,搭建成功!    
安装成功后,你可以想测试一下,那么可以使用如下命令:
	gem install rdiscount --platform=ruby
如果安装成功后,就可以使用一些`Ruby`的工具了,也为后面搭建博客提供了基础环境.但是也可能会给你带来麻烦,这个另说.

####配置本地编码环境
Otcopress采用`UTF-8无BOM`编码格式,故使用其他格式可能产生不可预计的错误.配置本地window的编码格式:
	set LANG   = zh_CN.UTF-8
	set LC_ALL = zh_CN.UTF-8

####下载并配置Octopress
假设Octopress安装在`~/repos`下,在git或cmd下执行命令:
	cd repos                                                    #~表示git主目录,不存在repos则 mkdir
	git clone git://github.com/imathis/octopress.git octopress  #octopress表示clone后所在的本地目录   
	                                                            #没有即使用原项目名即 octopress
然后更新 Octopress的gem更新源:进入`~/repos/octopress`目录,用文本编辑器(如EditPlus)打开文件`Gemfile`,将里面`source "http://rubygems.org"`改为`source "http://ruby.taobao.org/"`.最后安装Octopress的依赖项,在git或cmd窗口输入命令:
	cd octopress                     #进入 octopress 目录
	gem install bundler              #安装 bundler
	bundle install                   #然后安装一些依赖的工具
如果安装一切没有,那么恭喜你,你已经完成了整个环境的搭建,接下来的真正的主角登场了.不过人生总有那么几件不如意的事,那么可能出现那些错误呢?我们来分析一下:
	问题    : bundle install时,总是提示某些包没有目录或找不到     
	解决方法 : 找到 ~/repos/octopress/ 目录下打开 Gemfile.lock文件查看specs:下的依赖包的版本    
	          对比 [ruby安装目录(c:\Ruby192)]/lib/ruby/gems/[可能的版本(1.9.1)]/gems下的包的名称      
	          修改Gemfile.lock的版本即可.但是不要修改那些存在 >= 的版本号(这就是我在第一步所说的麻烦) 

####在git@github.com上建立Repositories
登录Github后,需要建立一个命名为`username.github.com`的Repo,这样命令的好处是,在运行命令`rake setpu_github_pages`能够自动创建`master`和`source`分支.创建(必须拥有github账号)如下图:![create repository](/images/common/2013-03-07-hello-octopress/create_repos.jpg "create repository of github")

####发布Octopress到Github
进入git bash后,进入到Octopress所在目录,输入命令:
	rake setup_github_pages
按照提示输入刚才新建的Repos地址,类似:git@github.com:用户名/用户名.github.com

####Octopress设置
	rake install           #安装octopress默认主题
	rake generate          #产生静态网页
	rake preview           #本地预览网站效果,在浏览器输入地址http://127.0.0.1:4000
人生不如意十有八九,以上三个命令看似简单,但有可能会成为拦路虎,而成为我们成功的绊脚石,让我们看看又会出现什么?
	rake generate失败
	 1. 检查_config.yml，注意每个冒号后面都有空格
	 2. 大多数原因是中文解析问题
	      首先所有的markdown文件应为UTF-8格式，然后修改 你ruby安装目录下的convertible.rb：
	      self.content = File.read(File.join(base, name))修改为
	      self.content = File.read(File.join(base, name), :encoding => "UTF-8")
	
	使用rake 命令遇到问题：       
	     rake aborted! No Rakefile found (looking for: rakefile, Rakefile, rakefile.rb, Rakefile.rb) 
	原因：没有进入工程目录

####发表文章
以发表 Hello World的文章为例:
	rake new_post["Hello World"]   #将在 octopress主目录/source/_posts/目录下生成    
	                                '年-月-日-hello-world.markdown'格式的文件   
.markdown文件可以用编辑软件打开,添加文章内容,具体用法参考[markdown用法(中文版)](http://wowubuntu.com/markdown/ "markdown用法(中文版)")和[octopress](http://octopress.org/ "octopress")

####绑定域名
github支持绑定独立域名,在`source/`目录下建立一个无扩展名的`CNAME`文件(文件名为CNAME),在里面写入域名:
	your_domain_name           # 用户名.github.com

####更新并发布
	rake gen_deploy         #等同于 rake generate 和 rake deploy两个命令  
	                        #此处有个坑爹的BUG,rake deploy时   
	                          rake aborted!   
	                          No such file or directory -public/_posts/...
	                        #解决方法:在public目录下创建_posts目录然后执行rake deploy

####备份
	git add *
	git commit -a -m "your comment message"     #提交信息不能为空
	git push origin source                      #把资源备份保存到 source 分支下[这里有个非常好的好处!]
	git checkout master                         #切换回 master 分支

关于此备份,为什么需要备份?   
博客文章部署到远程master分支下,然后将本地的所有源文件都git push 到source分支下,这样的话你可以在不同的地方都可以写blog.  
只要你在写博客之前把source分支下的所有文件git pull 到本地即可,当然你的所有修改都必须在source分支体现(现在我还没有想到更好的方法,我只能在每次操作后都是__先删除sourc分支,然后重建分支__),如果有人知道好的方法或不认同该方法都可以__chenming142@sina.com__,谢谢!!

