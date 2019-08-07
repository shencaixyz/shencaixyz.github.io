---
title: Linux 设置软连接
date: 2019-08-07 10:18:18
tags: [hexo,github,多端同步]
---
1. 以安装 laravel-echo-server 为例,安装了之后,安装路径为 /etc/node-v10.15.3-linux-x64/bin/laravel-echo-server

2. 执行 cd /usr/bin

3. 执行 ln -s /etc/node-v10.15.3-linux-x64/bin/laravel-echo-server laravel-echo-server

4. 然后就可以任意位置愉快的执行 laravel-echo-server 了

