---
title: 优化睡觉的沙发
date: 2018-08-20 14:15:15
categories: 
tags:
- 安利系列
---

> 博客太久没动了，刚好没事做的时候，就特别改博客主题，感觉像是强迫症，得改~
>
> 玩博客久了，就像谈恋爱，刚开始很美好，久而久之，问题就冒出来，总想让她变的更好，
>
> 其实这些都是次要的，我也知道更应该专注文章的内容，哎......

## 集成Gitment

[**Gitment**](https://github.com/imsun/gitment) 是作者实现的一款基于 GitHub Issues 的评论系统。支持在前端直接引入，不需要任何后端代码。可以在页面进行登录、查看、评论、点赞等操作，同时有完整的 Markdown / GFM 和代码高亮支持。尤为适合各种基于 GitHub/Coding Pages 的静态博客或项目页面

- [**项目地址**](https://github.com/imsun/gitment)
- [**示例页面**](https://imsun.github.io/gitment/)

当然，人家作者也说了，着这个项目的时候考虑过到底有没有滥用 GitHub ，为此还特地 Email 了官方，GitHub 给出的回复是：

> We’re pleased to see you making use of the tools and resources available on GitHub.

所以啊，我们在使用的时候，也要恪尽职守，维护一个好的开源环境，善用 `GitHub`

好了，说多了，下面开始：

### 注册 OAuth Application

[点击此处](https://github.com/settings/applications/new) 来注册一个新的 OAuth Application，成功之后就会拿到 `Client ID` 和 `Client Secret`

### 修改主题配置文件

```yaml
##############################################################################
# Plugins
##############################################################################

## Gitment
gitment:
  enable: true
  owner: your_id
  repo: your_repo
  client_id: your_client_id
  client_secret: your_client_secret
```

### 为 Gitment 添加 layout 布局

这里以我的主题 [**cactus**](https://github.com/probberechts/hexo-theme-cactus) 为例，在 `layout/_partial/comments.ejs` 下列代添加码：

```ejs
<% if(is_post() || page.comments) { %>
<div id="gitment_title" class="gitment_title"></div>
<div id="container" style="display:none"></div>
<script src="https://imsun.github.io/gitment/dist/gitment.browser.js"></script>
<script src="https://code.jquery.com/jquery-3.3.1.min.js"></script>
<script>
  const myTheme = {
    render(state, instance) {
      const container = document.createElement('div');
      container.lang = "en-US";
      container.className = 'gitment-container gitment-root-container';
      container.appendChild(instance.renderHeader(state, instance));
      container.appendChild(instance.renderEditor(state, instance));
      container.appendChild(instance.renderComments(state, instance));
      container.appendChild(instance.renderFooter(state, instance));
      return container;
    }
  }

  function showGitment() {
    $("#gitment_title").attr("style", "display:none");
    $("#container").attr("style", "").addClass("gitment_container");
    var gitment = new Gitment({
      id: decodeURI(window.location.pathname),
      theme: myTheme,
      owner: "<%=theme.gitment.owner%>",
      repo: "<%=theme.gitment.repo%>",
      oauth: {
	      client_id: "<%=theme.gitment.client_id%>",
	      client_secret: "<%=theme.gitment.client_secret%>"
      }
    });
    gitment.render('container');
  }
  showGitment();
</script>
<% } %>
```

### 为 Gitment 添加样式

[**_gitment.styl**](https://github.com/leeifme/leeifme.github.io/blob/hexo/themes/cactus/source/css/_gitment.styl) 这是我自定义的 Gitment 的样式文件，比较契合我的主题 `cactus` ，可以自行修改；将样式文件拷贝到 `source/css/` 目录下

**引入 Gitment 样式**

在 `source/css/style.styl` 文件中，添加：

```stylus
@import "_gitment.styl"
```

ok，更新博客，就可以看到评论了

## 主题首页添加 一言

> 一言：随机一句话副标题，
>
> 可以是一本书，一部电影，一句歌词，
>
> 可以是中文，英文，日语，
>
> 总之是一句话

感谢作者：[**isecret**](https://github.com/isecret)，是从他他制作的 Hexo 主题 [**Hola**](https://github.com/isecret/Hola) 获取的灵感

### 修改主题配置文件

```yaml
##############################################################################
# Plugins
##############################################################################

## 一言
hitokoto: enabled
```

### 为一言添加 layout 布局

在 `layout/index.styl` 文件中的 `<section id="about">` 内添加：

```stylus
<% if (theme.hitokoto == 'enabled') { %>
    <p>
      <script>
          fetch('https://api.hitokoto.cn')
              .then(function (res){
                  return res.json();
              })
              .then(function (data) {
                  var hitokoto = document.getElementById('hitokoto');
                  hitokoto.innerText =  data.hitokoto + ' ——' + '『' + data.from + '』';
              })
              .catch(function (err) {
                  console.error(err);
              });
      </script>
      <a id="hitokoto"></a>
    </p>
 <% } %>
```

[**演示地址**](https://leeif.me/) 

## gulp 文件压缩

### 安装 npm 依赖组件

```bash
# 在站点的根目录下执行以下命令

npm install gulp -g
npm install gulp-minify-css gulp-uglify gulp-htmlmin gulp-htmlclean gulp --save

# 模块作用
gulp-htmlclean // 清理html
gulp-htmlmin // 压缩html
gulp-minify-css // 压缩css
gulp-uglify // 混淆js
gulp-imagemin // 压缩图片
```

### 新建 gulp 配置

```js
/* 在站点根目录添加 gulpfile.js 文件，内容如下： */

var gulp = require('gulp');
var minifycss = require('gulp-minify-css');
var uglify = require('gulp-uglify');
var htmlmin = require('gulp-htmlmin');
var htmlclean = require('gulp-htmlclean');
// 压缩 public 目录 css
gulp.task('minify-css', function() {
    return gulp.src('./public/**/*.css')
        .pipe(minifycss())
        .pipe(gulp.dest('./public'));
});
// 压缩 public 目录 html
gulp.task('minify-html', function() {
  return gulp.src('./public/**/*.html')
    .pipe(htmlclean())
    .pipe(htmlmin({
         removeComments: true,
         minifyJS: true,
         minifyCSS: true,
         minifyURLs: true,
    }))
    .pipe(gulp.dest('./public'))
});
// 压缩 public/js 目录 js
gulp.task('minify-js', function() {
    return gulp.src('./public/**/*.js')
        .pipe(uglify())
        .pipe(gulp.dest('./public'));
});
// 执行 gulp 命令时执行的任务
gulp.task('default', [
    'minify-html','minify-css','minify-js'
]);
```

### 部署命令

在站点根目录执行 `gulp` 命令，就能压缩 `public` 文件夹里面的文件了

```bash
hexo g && gulp;
hexo d;
```



> 后续有其他优化，会持续更新的......