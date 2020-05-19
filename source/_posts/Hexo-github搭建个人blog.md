---
title: Hexo+GitHub搭建个人blog
date: 2020-05-19 13:22:53
author: 零度
tags: 
	- Hexo
	- Blog
categories: 环境搭建
abbrlink: 9799c9c57b84aaea
---

很久以前就想搞一个个人blog，但是想想搞blog你得搞个服务器吧，除了服务器也还得会前端吧。当时脑海中的各种预演直接劝退了我，直到我从某大佬那里知道了GitHub Pages和Hexo，直接满足了需求。那还等啥呢，直接搞起来。

<!-- more -->

## 流程

### 准备工作

#### Node.js

由于Hexo基于Node.js实现，所以我们首先要安装一下Node.js。

[Node.js 官网](https://nodejs.org/zh-cn/)

这里建议大家下载长期支持版。全部默认安装即可。

#### Git&GitHub

因为要将其发布在GitHub Pages上，我们需要在GitHub上注册一个账户，同时还要安装一下Git便于发布和提交代码。

[Git 官网](https://git-scm.com/download)

从Git官网下载并安装后，你可以根据自身需求选择是否需要编辑器，例如[VS Code](https://code.visualstudio.com/)。随后在GitHub网站上注册账号，英文用户名会成为你的免费域名的前缀。

然后登录GitHub，点击右上角`+`，建立一个新的`repository`，并把`repository`命名为`你的英文用户名.github.io`，英文用户名大小写无所谓。

最后点击`Create repository`创建仓库。

### 安装Hexo

[Hexo官网](https://hexo.io/)

[Hexo文档](https://hexo.io/zh-cn/docs/index.html)（一定要好好看文档呐！找个问题找半天，问了大佬才发现在文档里有写）

在终端输入以下命令：

```shell
npm install hexo-cli -g
```

-g 表示以全局方式安装。

然后在终端利用`cd`命令进入你想安放网站文件的地方，初始化博客模板并安装相应依赖。

```shell
cd <yourpath>
hexo init <你的名字.github.io>
npm install
```

接下来可以输入

```shell
#可缩写为 hexo s
hexo server
```

开启本地Hexo服务，这时可以打开浏览器，在地址栏输入`localhost:4000`查看本地网页。

按`Ctrl+C`中断本地Hexo服务。

### 使用Hexo主题

Hexo有很多主题，你可以在[Themes](https://hexo.io/themes/)中寻找你自己喜欢的主体配置。

这里介绍我使用的[云游君](https://www.yunyoujun.cn/)开发的主题[hexo-theme-yun](https://github.com/YunYouJun/hexo-theme-yun)。

这位大佬非常牛逼，提供了详细的[配置文档](https://yun.yunyoujun.cn/)，有兴趣的同学可以自行根据配置文档进行自定义。

#### 下载Hexo主题

进入终端（路径别出错哦，是你初始化Hexo的文件目录下），输入以下命令

```shell
git clone https://github.com/YunYouJun/hexo-theme-yun themes/yun
```

等待完成。

#### 编辑Hexo配置

在你之前Hexo初始化的文件目录下有一个`_config.yml`文件，你可以通过[Hexo配置](https://hexo.io/zh-cn/docs/configuration)查看并修改相应的配置。这里你需要找到`theme`字段，并将其修改为`yun`。

```yaml
theme: yun
```

由于`yun`主题使用了 `pug`和`stylus`，所以还需安装相应的渲染器。

```shell
npm install hexo-render-pug hexo-renderer-stylus
```

此时你可以使用`hexo server`启动本地Hexo服务查看本地网页。

#### 自定义主题配置

首先不建议直接修改主题的配置`themes/yun/_config.yml`，建议在`source/_data/yun.yml`新建配置文件，以便日后主题升级。

`yun`主题的配置可以参考[配置文档](https://yun.yunyoujun.cn/)，这位大佬写的十分详细。

### 生成静态文件

在我们本地配置完成后，我们就需要把本地文件放到GitHub上了。在这之前我们需要生成静态文件，GitHub仅支持静态页面，这也就意味着没有后端，没法评论啥的了。但是在大佬的主题中有多种第三方评论解决方案，可以按需配置。

```shell
# 多次生成静态文件前建议清除缓存
hexo clean
# 生成静态文件，可缩写为 hexo g
hexo generate
```

此时你的文件夹目录下会出现`public`文件夹，里面就是你的静态文件。

### 与GitHub仓库建立关联并部署

下面我们需要将本地的Git和GitHub上的仓库进行关联。

```shell
# 初始化Git仓库
git init
```

在部署到GitHub Pages之前，我们可以先建立一个分支，用来储存源代码。

GitHub Pages会默认使用master分支进行静态页面部署，我们可以建立一个hexo分支来存放网页的源代码文件。

```shell
git checkout -b hexo
```

这里推荐两个大佬的Git学习笔记。

[云游君的Git学习笔记](https://www.yuque.com/yunyoujun/notes/git-learn-note)

[Mind Walker]([https://njuwuyuxin.github.io/2019/04/18/git%E5%BF%AB%E9%80%9F%E4%B8%8A%E6%89%8B/](https://njuwuyuxin.github.io/2019/04/18/git快速上手/))

Git 真的很好用，不止是代码，写文章也可以用2333。

Hexo给我们提供了部署插件`hexo-deployer-git`。

```shell
npm install hexo-deployer-git
```

编辑Hexo初始化目录下`_config.yml`文件

```yaml
deploy:
  type: git
  repo: 你此前新建的仓库的链接 # https://github.com/xxxx/xxxx.github.io
  branch: master # 默认使用 master 分支
  message: Update Hexo Static Content # 你可以自定义此次部署更新的说明
```

保存之后便可部署（会要你输入用户名和密码）

```shell
hexo deploy
```

等待完成后打开网址`https://xxxx.github.io`就能看到你的网站了。

### 备份

为了避免哪天你的硬盘咔擦一下彻底退休了，我们还需要网站源代码文件推到GitHub仓库进行备份。

```shell
# 与远程 Git 仓库建立连接，只此一次即可
git remote add origin https://github.com/你的用户名/你的名字.github.io
```

接下来就是提交了

```shell
# 添加到缓存区
git add -A
git commit -m "这次做了什么更改，简单描述下即可"
# 推送至远程仓库
git push
# 第一次提交，你可能需设置一下默认提交分支
# git push --set-upstream origin hexo
```

如果觉得麻烦，还可以使用脚本。

在根目录下新建`update.sh`。

```bash
# 如果没有消息后缀，默认提交信息为 `:pencil: update content`
info=$1
if ["$info" = ""];
then info=":pencil: update content"
fi
git add -A
git commit -m "$info"
git push origin hexo
```

这样的话，以后只需要在终端执行`sh update.sh`即可。

有关自动部署内容我没有细究。

这里云游君推荐了他的小伙伴[ChrAlpha](https://blog.ichr.me/post/automated-deployment-of-serverless-static-blog/)的做法，有兴趣的同学可以自行查看。

### 写作

输入以下命令即可创建

```shell
hexo new [layout] <title>
```

`layout`默认post布局，采用Markdown语法。

[Markdown指南](https://github.com/younghz/Markdown)

### 文章永久链接

Hexo默认文章永久链接为`https://yoursite.com/yea/month/day/title`。

详情可查看 [Hexo永久链接](https://hexo.io/zh-cn/docs/permalinks)

这一链接有个问题，永久链接过长且会随文章标题而变化。

我采用了Node.js的Grunt插件解决该问题（我找不到提供这解决方案的网站了qaq）。

首先打开终端安装Grunt，并把它配置到项目package.json中。

```shell
npm install -g grunt-cli
npm install grunt --save-dev
npm install grunt-rewrite --save-dev
npm install
```

然后新建`Gruntfile.js`文件，并键入如下代码

```javascript
module.exports = function(grunt) {
  grunt.initConfig({
    rewrite: {
      abbrlink: {
        src: 'source/_posts/**/*.md',
        editor: function(contents, filepath){
          const crypto = require('crypto');
          const hash = crypto.createHash('sha256');

          hash.update(contents);
          var hashValue = hash.digest('hex');

          return contents.replace(/9799c9c57b84aaea/g, hashValue.substring(0, 16));
        }
      },
    },
  });
  grunt.loadNpmTasks('grunt-rewrite');
  grunt.registerTask('default', ['rewrite']);
};

```

保存后我们需要在文章的`Front-matter`下加上一行

```
title: Hexo+GitHub搭建个人blog
date: 2020-05-19 13:22:53
author: 零度
tags: 
categories: 
# 加入该行
abbrlink: 9799c9c57b84aaea
```

回到主目录下，执行`grunt`命令，会自动根据文章内容进行hash取代`9799c9c57b84aaea`。

然后编辑主目录下`_config.yml`文件，找到`permalink`字段

```yaml
# URL
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
url: http://lingdu-zero.github.io
root: /
permalink: # 修改为 posts/:abbrlink.html
permalink_defaults:
pretty_urls:
  trailing_index: true # Set to false to remove trailing 'index.html' from permalinks
  trailing_html: true # Set to false to remove trailing '.html' from permalinks

```

大功告成。

如果对Grunt感兴趣，可访问[Grunt快速入门](https://www.gruntjs.net/getting-started)查看。

## 参考资料

0. 排名不分先后

1. [_云游君的小站_](https://www.yunyoujun.cn/)

2. [_Hexo文档_](https://hexo.io/zh-cn/docs/index.html)

3. [_Mind Walker_](https://njuwuyuxin.github.io/)

4. [_hexo-theme-yun文档_](https://yun.yunyoujun.cn/)
5. [_Markdown指南_](https://github.com/younghz/Markdown)
6. [Grunt快速入门](https://www.gruntjs.net/getting-started)

## 致谢

十分感谢两位大佬[云游君的小站](https://www.yunyoujun.cn/)和[Mind Walker](https://njuwuyuxin.github.io/)拉我入坑并解答我的问题。

