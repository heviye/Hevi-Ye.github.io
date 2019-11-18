## 如何使用

文章提交前可以先在本地查看效果，满意了推送到远端，可以使用以下工具或命令

### 创建一个新的博客

1. 安装完整的[Ruby开发环境](https://jekyllrb.com/docs/installation/)
2. 安装Jekyll和[bundler](https://jekyllrb.com/docs/ruby-101/#bundler) [gem](https://jekyllrb.com/docs/ruby-101/#gems)

```shell
gem install jekyll bundler
```

3. 创建一个新的Jekyll网站

```shell
jekyll new myblog
```

### 启动本地服务器

```shell
# 进入目录
cd myblog

# 启动服务
bundle exec jekyll serve
```

现在浏览到http://localhost:4000

详细文档查看[中文文档](https://www.jekyll.com.cn)，[英文文档](https://jekyllrb.com)

## 所有文章写在`_posts`文件夹里
格式：YEAR-MONTH-DAY-title.md

新增或修改文章后push到github就好了，github会自动部署，不过时间比较长，10分钟左右