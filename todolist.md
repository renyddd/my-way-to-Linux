# mirrors
sudo pacman-mirrors -i -c China

> archlinuxcn

sudo nano /etc/pacman.conf

[archlinuxcn]
SigLevel = Optional TrustedOnly
Server = https://mirrors.ustc.edu.cn/archlinuxcn/$arch

# make it work
sudo pacman-mirrors -g

# update all
sudo pacman -Syyu

# pacman key setting
sudo pacman -Syy && sudo pacman -S archlinuxcn-keyring


# update driveres (NOT DONE)
sudo pacman -Qs drivers
> sudo pacman -Sy xf86-video-ati

# time set
sudo ntpdate cn.pool.ntp.org
sudo hwclock -W

# git config
[SSH key config](https://blog.csdn.net/u013778905/article/details/83501204)

# ssqt5
当打开软件并且成功 connect 后，配置 system setting：
HHTP、SSL、FTP 地址栏都填写空并且端口为 0，仅在 SOCKS Proxy 中填写 127.0.0.1 主机与相应的 1080 端口。
在使用 ss -tnl 后，便可查得相应端口已开启，不过在不使用时应将软件与 manually proxy configuration 同时关闭。
此时浏览器能正常访问，当命令行工具无法正常访问，需要使用 proxychains 工具：
sudo pacman -S proxychains
之后修改 /etc/proxychains.conf 文件，注释掉 proxy_dns 行以避免 DNS 污染，并且在末行按要求添加 socks5 行。

效果检测：
```bash
curl ifconfig.me
curl http://members.3322.org/dyndns/getip
```
对比两次运行结果差异即可。

[link](https://github.com/shadowsocks/shadowsocks-qt5/issues/764)
1. https://forum.suse.org.cn/t/shadowsocks-qt5/4245
2. https://www.litcc.com/2016/12/29/Ubuntu16-shadowsocks-pac/index.html
3. https://zcla.top/views/category/linux/2019/manjaro%E5%AE%89%E8%A3%85shadowsocks-qt5.html
4. https://guangchuangyu.github.io/cn/2018/09/proxychains/

# raw.githubusercontent.com
vim /etc/hosts
199.232.68.133 raw.githubusercontent.com

# fcitx
https://wiki.archlinux.org/index.php/Fcitx_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)
JUST leave out, and login.

# TIM
https://wiki.archlinux.org/index.php/Tencent_QQ_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)#Deepin_QQ/TIM

# some tips
- F12：拉幕式终端
- Alt+空格：调出全局搜索

# set golang
在 $HOME/.profile 中添加，解决`go install`错误
export GOPATH=$HOME/work
export GOBIN=$GOPATH/bin

# download DWM
http://dwm.suckless.org/
comment out ~/.xinitrc last line `exec dwm`.

# virtualBox 内核版本不匹配的错误
https://www.cnblogs.com/sztom/p/10461281.html
https://wiki.manjaro.org/index.php/Virtualbox

# KVM 安装命令
niubility: https://hacpai.com/article/1577420689618
sudo pacman -S virt-manager qemu vde2 ebtables dnsmasq bridge-utils openbsd-netcat
> network 'default' error: https://github.com/kubernetes/minikube/issues/828#issuecomment-261541451

# INTERESTING THINGS BACKUP
- https://v2raying.com/index.php/
- https://wiki.archlinux.org/index.php/ATI_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)  显卡驱动问题 

# NICE TOOL
https://coolshell.cn/articles/7829.html
