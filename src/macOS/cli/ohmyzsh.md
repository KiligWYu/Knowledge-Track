# [Oh My Zsh](https://ohmyz.sh/#install)

## 安装

``` sh
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

## 主题

`ZSH_THEME="af-magic"` (inside `~/.zshrc`).

## [zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions) 自动补全

``` sh
brew install zsh-autosuggestions
```

Add the plugin to the list of plugins for Oh My Zsh to load (inside `~/.zshrc`):

``` sh
plugins=(zsh-autosuggestions)
ZSH_AUTOSUGGEST_HIGHLIGHT_STYLE="fg=#ff00ff,bg=cyan,bold,underline" # Suggestion Highlight Style
```
