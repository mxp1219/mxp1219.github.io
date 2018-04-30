---

---

# Github Pages + Hexo + Travis CI 搭建个人博客

- 在本地写 markdown
- 写完 push 到仓库
- 个人博客自动得到更新
- 也可以配置自己的专属域名

## Github Pages

- 把静态页面推送到 Github 仓库
- 在设置页面中找到 Github Pages
- 选择 master 分支，然后点击 save     setting ->GitHub Pages->**Source**_>master->save
- 接下来就可以访问了
- 注意：https://你的github用户名.github.io/仓库名称/
- 这种方式必须作为子级目录才能访问到页面
- 如何直接通过 https://你的github用户名.github.io 访问你的网站呢？
- 如果需要通过 https://你的github用户名.github.io 访问你的仓库，则你的仓库名称必须叫：**github用户名.github.io**
- 对于 github用户名.github.io 仓库，github 会默认开启 github pages 功能，只要里面有一个 index.html 文件，那么你就可以直接通过 https://你的github用户名.github.io 来访问了。

## Hexo

安装

```shell
yarn global add hexo-cli    /npm install  hexo-cli
```

初始化生成博客

```shell
# blog 是目录名称，hexo 会在执行该命令的路径下生成一个名称为 blog 的博客项目
hexo init blog

# 进入刚刚生成的博客目录
cd blog

# 安装依赖
yarn install

# 启动本地预览
hexo server
```

## 完整流程

**1. 初始化 Hexo 博客项目**

```
hexo init blog
cd blog
yarn install

# 本地预览
hexo server
```

**2. 创建远程仓库**
![](github-pages-personal-blog/github-pages-new repository.png)

![](github-pages-personal-blog/github-pages-personal-blog14.png)

**3. 将本地的 hexo 博客推送到远程仓库**

```shell
# 把 hexo 网站源码推送到 demoThink.io 仓库的 hexo 分支中
git init  初始化仓库
git add *    
git commit -m "Blog Initialize"
git remote add origin https://github.com/demoThink/demoThink.github.io.git
git push origin master:hexo
```

---

## 使用 Travis CI 持续集成自动发布部署博客

- https://www.travis-ci.org/

**1. 使用 Github 登陆 Travis CI**


**2. 使用 使用第三方登陆的方式登陆**

![](github-pages-personal-blog/github-pages-personal-blog0.png)

**2. 进入个人中心**

![](github-pages-personal-blog/github-pages-personal-blog1 .png)

**3. 勾选需要被监视的仓库**

![](github-pages-personal-blog/github-pages-personal-blog2.png)

**4. 点击小齿轮进入设置页面**

![](github-pages-personal-blog/github-pages-personal-blogs2.png)

**5. 勾选几个选项**

![](github-pages-personal-blog/github-pages-personal-blog3.png)

**6. 在 Github 账户设置中生成 access-token 让 Travis CI 具有对你的仓库操作的权限**

![](github-pages-personal-blog/github-pages-personal-blog4.png)

![](github-pages-personal-blog/github-pages-personal-blog5.png)

![](github-pages-personal-blog/github-pages-personal-blog6.png)

![](github-pages-personal-blog/github-pages-personal-blog7.png)

![](github-pages-personal-blog/github-pages-personal-blog8.png)

![](github-pages-personal-blog/github-pages-personal-blog9.png)

![](github-pages-personal-blog/github-pages-personal-blog10.png)

1d5e1c1379a2834f8a314c2ab7055b279318110e

**7. 在 Travis CI 中把刚才生成得到的 token 配置到环境变量中**

![](github-pages-personal-blog/github-pages-personal-blog11.png)

![](github-pages-personal-blog/github-pages-personal-blog12.png)

**8. 把 `.travis.yml` 和 `publish-to-gh-pages.sh` 放到你的博客项目中**

![](github-pages-personal-blog/github-pages-personal-blog13.png)

`.travis.yml` 文件内容如下

```yml
language: node_js # 设置语言
node_js: stable # 设置相应版本
cache:
    apt: true
    directories:
        - node_modules # 缓存不经常更改的内容
before_install:
    - export TZ='Asia/Shanghai' # 更改时区
    - npm install hexo-cli -g
    - chmod +x ./publish-to-gh-pages.sh  # 为shell文件添加可执行权限
install:
    - npm install # 安装hexo及插件
script:
    - hexo clean # 清除
    - hexo g # 生成
after_script:
    - ./publish-to-gh-pages.sh
branches:
    only:
        - hexo # 只监测hexo分支
env:
    global:
        - GH_REF: github.com/lipengzhou/lipengzhou.github.io.git # 设置GH_REF，注意更改成自己的仓库地址
```

`publish-to-gh-pages.sh` 文件内容如下

```
#!/bin/bash
set -ev
git clone https://${GH_REF} .deploy_git
cd .deploy_git
git checkout master
cd ../
mv .deploy_git/.git/ ./public/
cd ./public
git config user.name  "lipengzhou" # 这里改成自己的 Github 用户名
git config user.email "lpzmail@163.com" # 这里改成自己的 Github 邮箱
# add commit timestamp
git add .
git commit -m "Travis CI Auto Builder at `date +"%Y-%m-%d %H:%M"`"
git push --force --quiet "https://${github_token}@${GH_REF}" master:master # github_token 改成你在 Travis CI 中添加的环境变量名称
```

**9. 发布**

把刚才的修改提交到 Github 仓库的 `hexo` 分支上

```shell
git add --all
git commit -m "配置 Travis CI 持续集成自动发布服务"
git push origin master:hexo   本地的master 分支 到远程仓库的hexo 分支
```

