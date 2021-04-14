# macOS

记录 macOS 应用与习惯的环境配置。

## 前置网络环境配置

- [Surge](https://nssurge.com)
- [ClashX](https://github.com/yichengchen/clashX)
- [ShadowsocksX-NG](https://github.com/shadowsocks/ShadowsocksX-NG)

## 开发环境配置

- [Oh My Zsh](./cli/ohmyzsh.md)

``` sh
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

- [Homebrew](./cli/homebrew.md)

``` sh
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

- [rbenv](./cli/rbenv.md) - ruby 环境管理工具

``` sh
brew install rbenv ruby-build
```

- [CocoaPods](https://cocoapods.org/)

``` sh
sudo gem install cocoapods
pod setup
```

对于苹果芯片的电脑，需要在命令前加上 `arch -x86_64`。e.g. `arch -x86_64 pod setup`.

## 更多 CLI

- [TinyPNG CLI](https://github.com/websperts/tinypng-cli)

``` sh
npm install -g tinypng-cli
```

> [TinyPNG](https://tinypng.com/) 是一个提供 PNG 和 JPEG 格式图片无损压缩的网站，压缩率高，并且基本不会丢失图片细节。TinyPNG CLI 则是利用其开发者接口制作的命令行工具。
>
> 使用 npm 安装 CLI 工具后，进入 TinyPNG 的 [开发者界面](https://tinypng.com/developers) 申请 API Key，在系统的用户目录创建名为「.tinypng」的文件并将获得的 Key 写入此文件。接着，我们就能在终端中使用 tinypng [filepath] 来压缩图片了，压缩后的图片将直接替换原图片。
>
> 用户每个月能免费压缩 500 张图片，每张图片不能超过 5 MB。

- [PicGo-Core](https://github.com/PicGo/PicGo-Core)

``` sh
npm install -g picgo
```

插件:

``` sh
picgo install autocopy rename-file
```

## 链接

- [Awesome macOS Command Line](https://github.com/herrbischoff/awesome-macos-command-line)
- [换一种姿势使用网页服务，这些命令行工具值得一试](https://sspai.com/post/65934)
