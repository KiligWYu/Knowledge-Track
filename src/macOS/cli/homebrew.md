# [Homebrew](https://brew.sh)

## 安装

``` sh
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

## 常用命令

- `brew search [TEXT|/REGEX/]`
- `brew info [FORMULA...]`
- `brew install FORMULA...`
- `brew update`
- `brew upgrade [FORMULA...]`
- `brew uninstall FORMULA...`
- `brew list [FORMULA...]`
- `brew cleanup`

## Formulae

- [annie](https://github.com/iawia002/annie)
- [aria2](https://aria2.github.io/)
- [jq](https://stedolan.github.io/jq/)
- [mas](https://github.com/mas-cli/mas)
- [trash](https://hasseg.org/trash/)
- [xcinfo](https://github.com/xcodereleases/xcinfo)
- [tree](http://mama.indstate.edu/users/ice/tree/)

`tree -N` 解决 `tree` 中文乱码。

> -N            Print non-printable characters as is.

## Casks

- [font-fira-code](https://github.com/tonsky/FiraCode)
- [font-jetbrains-mono](https://www.jetbrains.com/lp/mono)
- [font-source-han-sans](https://github.com/adobe-fonts/source-han-sans)
- [font-source-han-serif](https://github.com/adobe-fonts/source-han-serif)

## 其他

避免触发自动更新

``` sh
export HOMEBREW_NO_AUTO_UPDATE=1
```

## 链接

- [从零开始，编写一个 HomeBrew 缓存清理脚本](https://sspai.com/post/65842)
