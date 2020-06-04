
# 使用github+hexo搭建博客

## 配置基本环境
 * 安装Git
 * 安装node.js
 * 新建github项目，项目名必须为：<你的github用户名>.github.io

## 安装hexo
```js
npm install -g hexo   //如果没有权限，请加上sudo
```

## 初始化你的blog
```js
hexo init myblog  //myblog为博客的文件夹名，没有限制
cd myblog
npm install --save
```

## 博客本地预览
```js
cd myblog
hexo server   // https://hexo.io/docs/server.html
```
默认预览网址：http://localhost:4000/

## 修改主题
下载主题到themes目录，如：git clone https://github.com/iissnan/hexo-theme-next themes/
修改配置文件 `_config.yml`
```js
theme: hexo-theme-next
```
ok

## 部署github地址
安装git插件
```js
npm install hexo-deployer-git --save
```

修改配置文件 `_config.yml`
```js
# Deployment
## Docs: http://hexo.io/docs/deployment.html
deploy:
  type: git
  repo: https://github.com/test/test.github.io
  branch: master
  message: "Site updated: {{ now('YYYY-MM-DD HH:mm:ss') }}"   //提交信息格式
```

## 基本命令
```js
hexo new "文章名"   //添加一篇新的文章
hexo new <页面模板名> "文章名"   //使用指定页面模板创建一遍文章，模板名不指定则用post模板
hexo generate | g    //生成文章, g为简写
hexo deploy | d    //发布文章, https://hexo.io/docs/deployment.html
```
完成这些步骤后，打开网址 test.github.io，即可看到效果

## 上传源码到github（可选步骤）
前面步骤只会发布生成的文章。
如果需要保存源码到github，如下：
```js
cd myblog
git init
git remote add origin git@github.com:test/test.github.io   //此处改为git@github...可以不用输入用户名和密码提交
git checkout -b source
git push -u origin source
```
## Next theme 配置参考
- http://theme-next.iissnan.com/
- https://github.com/iissnan/hexo-theme-next/wiki/%E5%88%9B%E5%BB%BA%E5%88%86%E7%B1%BB%E9%A1%B5%E9%9D%A2

## 配置多说评论框
- http://taolin.duoshuo.com/admin/
