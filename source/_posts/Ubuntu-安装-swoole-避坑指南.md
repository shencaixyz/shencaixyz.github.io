---
title: Ubuntu 安装 swoole 避坑指南
date: 2019-05-16 16:59:40
tags: [Ubuntu,swoole]
toc: true
---
### 准备工作

以下操作根据实际情况选择执行

#### 安装C++库

```
apt-get install build-essential 
apt-get install g++
```

#### 安装pecl

```
apt install php-pear
```

#### 安装phpize

```
apt-get install php7.2-dev
```

### 开始安装

#### 编译安装

```
cd ~ 
sudo mkdir /soft
sudo chown ashen /soft
wget https://codeload.github.com/swoole/swoole-src/tar.gz/v4.3.3
tar zxvf swoole-src-4.3.3.tar.gz
cd swoole-src-4.3.3
/soft/php/bin/phpize
./configure --with-php-config=/usr/bin/php-config
make
make install
```

注：
1. swoole 版本号根据实际版本号选择
2. ashen 为当前用户名
3. ./configure --with-php-config=/usr/bin/php-config 的 /usr/bin/ 路径通过 which php 获取详细路径

#### 一键安装

```
sudo pecl install swoole
```

#### 修改php.ini

进入 /etc/php/7.2/cli/php.ini,添加 extension=swoole.so

执行php -m 即可查看是否安装成功


