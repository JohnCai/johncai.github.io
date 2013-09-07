---
layout: post
title: "快速使用Github Pages搭建个人博客"
description: "最快速上手，利用Jekyllbootstrap,搭建一个托管在Github Pages上的个人blog"
category: Howto
tags: [Github page, Jekyll]
---
{% include JB/setup %}

###Github上建立Repository
1. 这里假设你已经有个Github账号，知道怎么在本地建立Repository，怎么push到Github上。如果还不知道，参见[这篇官方文章](https://help.github.com/articles/generating-ssh-keys)。
2. 在Github上创建一个名为USERNAME.github.io的repository（把USRENAME换成你的Github账号）

###安装Ruby
1. 到[这里](http://rubyinstaller.org/downloads/)下载Ruby 1.9.3-p448（_注意版本_，下面的步骤需要这个版本），以及对应的DevKit-tdm-32-4.5.2-20111229-1559-sfx.exe。
2. 安装ruby，注意安装时check一个选项，把它加到PATH里。
3. 然后双击DevKit-tdm-32-4.5.2-20111229-1559-sfx.exe解压到比如D:\DevKit
4. 以管理员模式打开cmd，进入到D:\Devkit
  -1. 先执行 `ruby dk.rb init`
  -2. 再执行 `ruby dk.rb install`
5. 默认ruby是从https://rubygems.org下载gem的，我们把它改成淘宝镜像，身为中国人，别问为什么。

```
gem sources --remove https:/rubygems.org/
gem sources -a http://ruby.taobao.org/
```

###安装本地Jekyll环境
1. Github很体贴的帮我们做了一个gem。执行`gem install github-pages`即可。
2. 找到`D:\Ruby193\lib\ruby\gems\1.9.1\gems\jekyll-1.1.2\lib\jekyll\convertible.rb`文件，
     把31行的`self.content = File.read(File.join(base, name))` 
     改成`self.content = File.read(File.join(base, name), :encoding => "utf-8")`
3. 找到`D:\Ruby193\lib\ruby\gems\1.9.1\gems\jekyll-1.1.2\lib\jekyll\tags\include.rb`文件，
     把65行的`source = File.read(@file)` 
     改成`source = File.read(@file, :encoding => "utf-8")`

###从Jekyllbootstrap开始
如果你熟悉Jekyll，你可以开始写博客了，如果不熟悉，请[Jekyllbootstrap](https://github.com/plusjade/jekyll-bootstrap.git)帮我们的忙。

* 进入Gitbash
执行

```
$ git clone https://github.com/plusjade/jekyll-bootstrap.git USERNAME.github.io
```

* 到你的USRERNAME.github.io文件夹下，打开`\_config.yml`文件,
	在顶部`pygments: true`下加上一行
		 `markdown: rdiscount` 或者
		 `markdown: redcarpet`

(这是因为Github Pages使用的不是标准的Markdown，而是有一点变种，叫[Github flavored markdown](http://github.github.com/github-flavored-markdown/), 上面那句话就是为了使用Github flavored markdown)

* 本地运行

进入到你的目录，启动Jekyll

```
cd USRERNAME.github.io
jekyll serve
```

然后到http://localhost:4000就可以看到你的博客了。

* 创建第一篇博客
clone自Jekylllbootstrap以后，其实在\_posts目录下已经有了一篇示例博客，你可以把它删掉，不过建议先留着，作参考。
我们来创建自己的第一篇博客,先按`ctrl+c`退出本地Jekyll，然后运行

```
rake post title="Hello World"
```

就可以在\_posts下看到一个名为20XX-xx-xx-hello-world.md的文件，打开它，你需要学习Markdown语法来编辑它。
再次执行Jekyll serve。就可以看到你的新博客了。

##push到Github
相信你在Github上已经建立了USRENAME.github.io的repository。push上去。

```
$ cd USERNAME.github.io
$ git remote set-url origin git@github.com:USERNAME/USERNAME.github.io.git
$ git push
```

然后（也许要等几分钟），到http://USRENAME.github.io就可以看到你博客了

##参考资料
* [Generating SSH Keys](https://help.github.com/articles/generating-ssh-keys)
* [Using Jekyll with Pages](https://help.github.com/articles/using-jekyll-with-pages)
* [使用Github Pages建独立博客](http://beiyuu.com/github-pages/)
* [在Windows上建立Jekyll平台](http://pengx17.me/learning/jekyll/2013/06/03/setup-local-jekyll-server-on-windows/)
* [Jekyll Quick Start](http://jekyllbootstrap.com/usage/jekyll-quick-start.html)
* [Markdown Cheetsheet](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet)