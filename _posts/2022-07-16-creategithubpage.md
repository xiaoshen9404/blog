---

layout: default
title: "Create Github Pages"
date: 2022-07-16 22:03:00 -0000
categories: Environment

---



```
** Github Pages build **

I hope you like it!
```

# Github Pages 搭建
## 环境准备（本文使用的环境）

>
>   * Git
>   * Ruby
>   * Bundler (Ruby的外部依赖管理工具,配合Gemfile外部依赖信息文件)
>

## 创建Github仓库
#### 创建仓库

![create repository](/assets/images/blogmaterial/githubrep.png "repository")

#### 设置仓库信息

>下面这里需要注意一下，格式为:  **你的github用户名** *.github.io*
>按格式设置，可以实现直接使用 **https://<span>用户名<span>.github.io** 来访问
>修改完后点击 **Create repository**

![create repository](/assets\images\blogmaterial\github_rep_info_set.png "repository info")

#### 在Setting中开启Pages
>点击 **Change theme** 选择一个自己喜欢的主题，并且下载资源到本地，后面起本地服务需要用到
>提交代码变动，仓库中会出现两个文件 **_config.yml , index<span>.md**
>并且pages页会出现你的主页地址


![create repository](/assets/images/blogmaterial/github_setting_btn.png "page setting")
![create repository](/assets/images/blogmaterial/rep_create_comp.png "page setting")

#### 拉取代码到本地
> 使用git拉取代码到本地
> 复制刚才下载的主题资源到仓库根目录下，会覆盖你的 **_config.yml , index<span>.md**，现在还是空文件我们直接覆盖即可

#### 执行git
``` git
git add .
git commit -m "init page code"
git push
``` 

#### 运行本地测试
检查本地环境是否配置正确
1. **检查 ruby**
```
 ruby --version
```
2. **检查 Bundler**
```
bundle -v
```
3. **使用 bundle 安装依赖（依赖配置在Gemfile中）**
```
bundle install
```
4. **运行 jekyll server**
```
bundle exec jekyll server
```
**Ruby3之后标准库不再包含webrick gem**
此处如果报错 无法加载 webrick/httputils ，就需要安装 webrick ,完成后重新运行jekyll server
```
bundle add webrick
```
5. **运行成功后在浏览器中打开查看**
```
http://localhost:4000 
```

## 添加帖子
1. **在根目录 _posts 中添加文件**
```
文件名格式: YEAR-MONTH-DAY-title<span>.md
```
```
打开文件，添加头部信息
注意时间最好不要超过github的时间，因为存在时差，发布时会报错
```
```
   ---
layout: default
title: "Create Github Pages"
date: 2022-07-13 22:03:00 -0000
categories: Environment
   ---
```