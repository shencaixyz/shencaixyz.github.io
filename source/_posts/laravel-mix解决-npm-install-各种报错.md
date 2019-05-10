---
title: laravel mix 解决 window 下 npm install 各种报错
date: 2019-05-10 14:45:19
tags: [laravel,mix,npm]
toc: true
---
### 问题
新建 laravel 项目后 yarn 或 npm install 命令的时候，window 总是各种报错，所以整理了一部分解决方法

### 终极大法

执行以下命令：

1. $ rm -rf node_modules

2. $ yarn config set registry http://registry.cnpmjs.org

3. $ yarn install --no-bin-links

4. 接下来打开 pakage.json 修改【去掉四处 cross-env 】

5. 执行 $ npm run watch-poll 或者 npm run dev（我是执行的后者）

### 参考

1. 如果遇到错误 error An unexpected error occurred: "EINVAL: invalid argument, symlink

   请在你的执行命令之后添加 --no-bin-links
    ```
    yarn install --no-bin-links
    ```
2. 如果遇到错误 error: xxxx node-sass: Command failed

   将 sass-binary-site 添加至 config 中
    ```
    yarn config set sass-binary-site https://npm.taobao.org/mirrors/node-sass
    npm config set sass-binary-site https://npm.taobao.org/mirrors/node-sass
    ```
   指定 node-sass 从 npm 的淘宝源中下载。

3. 如果遇到错误
    ```
    ERROR Failed to compile with 2 errors 11:10:34
    error in ./resources/assets/sass/app.scss
    ```
    - 检查你使用的源是否更换，当然可以选择使用代理 :joy:
    - 检查 sass-binary-site 是否指定
    - 执行 yarn install --force node-sass npm 对应的是 npm rebuild node-sass
    - 我习惯直接 yarn install --force 将整个重新进行构建
4. 如果你使用 yarn 安装了 vue 这样的 cli 工具。然后无法执行命令。请将 yarn global bin 添加至 PATH 环境变量中。
5. 如果遇到 cross-env 报错 执行 npm i -g cross-env



