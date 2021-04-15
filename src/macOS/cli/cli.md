# 命令行

假装「黑客」一样行事直到不希望被发现。😜

## 工具

- [`bat`](https://github.com/sharkdp/bat) - A cat(1) clone with syntax highlighting and Git integration.

## Alias

``` sh
# Proxy
alias sson='export https_proxy=http://127.0.0.1:6152;export http_proxy=http://127.0.0.1:6152;export all_proxy=socks5://127.0.0.1:6153'
alias ssoff='unset all_proxy'

export https_proxy=http://127.0.0.1:6152;export http_proxy=http://127.0.0.1:6152;export all_proxy=socks5://127.0.0.1:6153

# 更新 ClashX Country.mmdb
alias updateGeoIP='curl -o ~/.config/clash/Country.mmdb https://github.com/Hackl0us/GeoIP2-CN/raw/release/Country.mmdb'

# Show Connected Device's UDID
alias udid='system_profiler SPUSBDataType | sed -n -e '/iPad/,/Serial/p' -e '/iPhone/,/Serial/p''

#  Show External IP Address
alias ip='dig +short myip.opendns.com @resolver1.opendns.com'

# Hexo
alias cgd='hexo clean && hexo g && hexo d'
alias cgs='hexo clean && hexo g && hexo s'

# Alias the system vim to MacVim
alias vi="/Applications/MacVim.app/Contents/MacOS/Vim"
alias vim="vi"
alias mvim="vi -g"

# Aria2
alias aria="~/OneDrive/Backups/aria2-trackers-update.sh"

# ping
alias ping5='ping -c 5'

function ~() {
    cd ~
}

# switch between release and beta xcodes
function xcswitch() {
    RELEASE="Xcode.app"
    BETA="Xcode-beta.app"

    CURRENT=$(xcode-select -p)
    NEXT=""

    if [[ "$CURRENT" =~ "$RELEASE" ]]
    then
        NEXT="$BETA"
    else
        NEXT="$RELEASE"
    fi

    sudo xcode-select -s "/Applications/$NEXT"
    echo "Switched to $NEXT"
}
```
