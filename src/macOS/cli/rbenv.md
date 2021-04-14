# [rbenv](https://github.com/rbenv/rbenv) - ruby 环境管理工具

## 安装

``` sh
brew install rbenv ruby-build
```

[ruby-build](https://github.com/sstephenson/rbenv) 用来编译安装 Ruby 源码，如果选择手动编译，可不使用这个工具。

## 常用命令

- `rbenv install --list` 查看可用的 ruby版本
- `rbenv install 3.0.1` 从列表中选择 3.0.1 进行安装
- `rbenv versions` 查看已经安装的所有 Ruby 版本
- `rbenv global 3.0.1` 设置全局版本
- `rbenv local 3.0.1` 设置本地版本
- `rbenv shell 3.0.1` 设置当前终端版本
- `rbenv global system` 使用系统 Ruby
- `rbenv rehash` 使用 rbenv 后，gem 还是按照原有的方式进行安装、升级，只是 gem 的安装路径是在 ~/.rbenv 文件夹中当前 Ruby 版本文件夹下。而且，安装带有可执行文件的 gem 后，需要执行一个特别的命令，告诉 rbenv 更新相应的映射关系，这个命令在安装新版本的 Ruby 后也需要执行

rbenv 中的 Ruby 版本有三个不同的作用域：全局 (global)，本地 (local)，当前终端 (shell)，**查找版本的优先级是 当前终端 > 本地 > 全局。**

## 链接

- [使用 rbenv 安装和管理 Ruby 版本](https://gist.github.com/sandyxu/8aceec7e436a6ab9621f)
