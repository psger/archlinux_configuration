# 基本安装
wifi-menu wlp3s0  
ping www.baidu.com  
timedatectl set-ntp true  
lsblk  
fdisk /dev/sda  

块设备 | 扇区个数 | ID | for  
--- | ---- | ---  
/dev/sda1 | 2097152 | 83 | /boot  
/dev/sda5 | 167772160 | 83 | /  
/dev/sda6 | 104857600 | 83 | /var  
/dev/sda7 | 314572800 | 83 | /home  
/dev/sda8 | 16777216 | 82 | swap  

lsblk  
mkfs.ext4 /dev/sda1	# /boot  
mkfs.ext4 /dev/sda5	# /  
mkfs.ext4 /dev/sda6	# /var  
mkfs.xfs  /dev/sda7	# /home  
mkswap    /dev/sda8	# swap  
swapon    /dev/sda8  
free  
mount /dev/sda5 /mnt # /  
mkdir -p /mnt/boot  
mkdir -p /mnt/var  
mkdir -p /mnt/home  
mount /dev/sda1 /mnt/boot  
mount /dev/sda6 /mnt/var  
mount /dev/sda7 /mnt/home  
df -h  
vim /etc/pacman.d/mirrorlist  
pacman -Syy  
pacstrap -i /mnt base base-devel  
genfstab -U -p /mnt >> /mnt/etc/fstab  
cat /mnt/etc/fstab  
blkid  
arch-chroot /mnt /bin/bash  
pacman -S vim  
vim /etc/pacman.conf  
```bash
#Color
```
> 取消这一行的注释  

vim /etc/locale.gen  
```bash
en_US.UTF-8 UTF-8
zh_CN.UTF-8 UTF-8
zh_TW.UTF-8 UTF-8
```
>en_US、zh_CN、zh_TW的可以都选上  

locale-gen  
echo LANG=en_US.UTF-8 > /etc/locale.conf  
tzselect	# 4 9 1 1  
ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime  
hwclock --systohc --utc  
mkinitcpio -p linux  
pacman -S grub os-prober  
grub-install /dev/sda  
grub-mkconfig -o /boot/grub/grub.cfg  
echo archlinux > /etc/hostname  
vim /etc/hosts  
```bash
127.0.0.1	localhost.localdomain	localhost	 archlinux
::1		localhost.localdomain	localhost	 archlinux
```
ip addr  
systemctl enable dhcpcd@enp4s0f2.service  
pacman -S iw wpa_supplicant dialog  
passwd  
exit  
reboot  

# 桌面配置
## 添加普通用户
useradd -m -g users -G wheel,audio,video,lp,log,uucp,rfkill,network,optical,floppy,storage,scanner,power,games,vboxusers yotta  
passwd yotta  
pacman -S sudo  
visudo	# 或者vim /etc/sudoers  
```bash
root ALL=(ALL) ALL
%wheel ALL=(ALL) ALL
```

## 安装x服务和相关驱动
1. 最省事：pacman -S xorg，否则  
pacman -S xorg-server xorg-xinit xorg-utils xorg-server-utils  
lspci | grep -e VGA -e 3D  
pacman -Ss xf86-video | less	# 查找开源显卡驱动  
pacman -S xf86-video-nouveau	# 这个是 NVIDIA  
pacman -S xf86-video-intel	# 这个是 Intel  
pacman -S xf86-video-ati	# 这个是 amd  
pacman -S xf86-input-synapticsf	# 触摸板  
2. 参照archwiki安装[bumblebee](https://wiki.archlinux.org/index.php/Bumblebee)，若安装的是开源显卡驱动，则
pacman -S bumblebee  
gpasswd -a yotta bumblebee  
systemctl enable bumblebeed.service  
> 安装之后，若重启之后看到  
```
[    15.824050] nouveau E[     DRM]Pointer to TMDS table invalid
[    15.824072] nouveau E[     DRM]Pointer to flat panel table invalid
```
> 则不成功，需重新查看wiki  
> 还有一种方法验证，optirun glxgears -info若不成功则optirun glxspheres64成功即可（32位则改64为32）  

## 安装gnome，字体和输入法
pacman -S gnome gnome-extra gnome-tweak-tool  
systemctl enable gdm.service  
systemctl enable NetworkManager.service  
pacman -S wqy-zenhei wqy-microhei  
pacman -S ttf-dejavu  
pacman -S adobe-source-code-pro-fonts  
pacman -S ibus ibus-libpinyin ibus-pinyin ibus-table ibus-table-chinese  
su - yotta  
vim ~/.xprofile  
```bash
export LANG=zh_CN.UTF-8
export LANGUAGE=zh_CN:en_US
export LC_CTYPE=en_US.UTF-8
export GTK_IM_MODULE=ibus
export QT_IM_MODULE=ibus
export XMODIFIERS="@im=ibus"
```
exit  
>输入法也可用小企鹅：pacman -S fcitx-im fcitx-configtool，然后把上面配置文件中的ibus改为fcitx  
>若为ibus，则  
>dconf update  
>ibus-daemon -rdx  

reboot #然后进入图形界面，普通用户（yotta）登陆  
设置->区域和语言，设置为如下样式  
>语言：汉语（中国）  
>formats：中国  
>输入源：  
>>汉语  
>>汉语（Intelligent Pinyin）  
>>汉语（Pinyin）  

su - root #以root身份继续进行其他操作  

## 配置archlinuxcn
>这一步因为要访问网页，但chrome浏览器属于中文仓库（正在配置的就是），无法安装  
>所以暂时先安装火狐浏览器pacman -S firefox firefox-i18n-zh-cn flashplugin  

vim /etc/pacman.conf  
这里先顺带打开所有的非testing仓库  
```bash
[core]
Include = /etc/pacman.d/mirrorlist

[extra]
Include = /etc/pacman.d/mirrorlist

[community]
Include = /etc/pacman.d/mirrorlist

[multilib]
Include = /etc/pacman.d/mirrorlist
```
然后在文件末尾添加  
```bash
[archLinuxcn]
SigLevel = Optional TrustedOnly
Include = /etc/pacman.d/archlinuxcn
```
vim /etc/pacman.d/archlinuxcn  
>[参见github](https://github.com/archlinuxcn/mirrorlist-repo)只取Server那一行组成Server列表  

pacman -S archlinuxcn-keyring  
pacman -Sy yaourt  

## 代理配置（shadowsocks）
pacman -S shadowsocks  
vim /etc/shadowsocks/hk.json  
```json
{
        "server":"hk02-49.ssv7.net",
        "server_port":19320,
        "local_address":"127.0.0.1",
        "local_port":1080,
        "password":"WpjX23DJFAdd",
        "timeout":300,
        "method":"aes-256-cfb",
        "workers":1
}
```
>通用方法是cp /etc/shadowsocks/example.json /etc/shadowsocks/hk.json  
>然后编辑hk.json，删除fast_open那一行  
>同样格式添加jp.json  

systemctl enable shadowsocks@hk.service  
su - yotta  
cd  
git clone https://github.com/Leask/Flora_Pac.git  
cd Flora_Pac  
vim flora_pac.pac  
>查找127.0.0.1，把其对应端口都改为1080  

exit  
cp flora_pac.pac /etc/shadowsocks  
然后进入设置->网络->网络代理，方法改为自动，URL为file:///etc/shadowsocks/flora_pac.pac  

pacman -S proxychains-ng  
vim /etc/proxychains.conf  
`sock5 127.0.0.1 1080`

>proxychains可以给所有软件代理（不只是浏览器）  
>在要执行的命令前加proxychains4就可以走代理了  

## 安装reflector
pacman -S reflector  
[官方wiki](https://wiki.archlinux.org/index.php/Reflector_)  
vim /etc/systemd/system/reflector.service  
```bash
[Unit]
Description=Pacman mirrorlist update
Requires=network-online.target
After=network-online.target

[Service]
Type=oneshot
ExecStart=/usr/bin/reflector --country 'China' --latest 5 --sort score --save /etc/pacman.d/mirrorlist

[Install]
RequiredBy=multi-user.target
```
vim /etc/systemd/system/reflector.timer  
```bash
[Unit]
Description=Run reflector weekly

[Timer]
OnCalendar=weekly
RandomizedDelaySec=12h
Persistent=true

[Install]
WantedBy=timers.target
```
systemctl start reflector.service  
systemctl enable reflector.timer  

## 安装其它工具
pacman -S ntp  
pacman -S google-chrome  
>前提是已经配置好了archlinuxcn，否则只能安装开源版本pacman -S chromium  

pacman -S ntfs-3g  
pacman -S openssh  
pacman -S telepathy  
pacman -S wps-office	# 中文仓库  
yaourt -S haroopad  
pacman -S atom-editor  
pacman -S gedit-plugins  
pacman -S netease-cloud-music	#中文仓库  
pacman -S abs screenfetch wget curl autojump ccal mlocate htop ncdu pkgfile net-tools  
>net-tools提供了ifconfig命令  

pacman -S linux-docs jdk8-openjdk openjdk8-doc qt5-doc gcc-docs groovy-docs php-docs python-docs  
pacman -S virtualbox virtualbox-host-modules-arch  
pacman -S qemu qemu-launcher  
>qemu已经带有调试功能，而bochs则需要源码安装，否则不带有调试功能  

pacman -S gimp  

## 安装oh my zsh
pacman -S zsh  
su - yotta  
sh -c "$(wget https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"  
cp .oh-my-zsh/templates/zshrc.zsh-template ~/.zshrc  
vim ~/.zshrc  
```bash
ZSH_THEME="ys"
plugins=(git archlinux systemd autojump sublime)
```
exit  
>其他配置参见[github](https://github.com/robbyrussell/oh-my-zsh)  

## 安装spf13这个vim插件集合的发行版
```bash
curl https://j.mp/spf13-vim3 -L > spf13-vim.sh && sh spf13-vim.sh
```
>具体使用方法参见[作者github](https://github.com/spf13/spf13-vim)  

## 终端透明
在正在使用的shell的配置文件（~/.zshrc或 ~/.bashrc）中加入如下代码  
```bash
if [ -n "$WINDOWID" ]; then
        TRANSPARENCY_HEX=$(printf 0x%x $((0xffffffff * 80 / 100)))
        xprop -id "$WINDOWID" -f _NET_WM_WINDOW_OPACITY 32c -set _NET_WM_WINDOW_OPACITY "$TRANSPARENCY_HEX"
fi
```
>也可以安装aur中的gnome-terminal-transparency来达到终端透明效果，但是此方法无法实现标题栏透明。

## gnome桌面细节调整
yaourt -S gnome-shell-system-monitor-applet-git  
打开设置->键盘->自定义快捷键，添加两个快捷键如下  
1. 名称：启动终端  
命令：gnome-terminal  
快捷键：Pause Break  
2. 名称：文件管理器  
命令：nautilus  
快捷键：Scroll Lock  

打开gnome-tweak-tool（中文叫“优化工具”）  
>可以在gnome-terminal（因为刚才设置了快捷键，现在按Pause Break键可打开）中输入gnome-tweak-tool命令打开，也可以在左上角“应用程序”->“工具”中找到“优化工具”  
>还可以按super键，然后输入名称（中文和英文名称都可以）搜索  

设置为如下样式  
>外观：  
>>全局黑色主题：打开  
>>启用动画：打开  

>工作区：  
>>创建工作区：动态  

>扩展：  
>>System-monitor：打开

>桌面：  
>>桌面图标：全部取消，然后选择关闭  

>电源：  
>>按下电源按钮时：Suspend  
>>机盖闭合时不要挂起：关闭  

>顶栏：  
>>显示应用程序菜单：打开  
>>其余全部打勾  

应用程序收藏夹（按super键时显示在左侧的那一竖条）：  
>Google Chrome  
>网易云音乐  
>文件  
>终端  
>haroopad  
>Evolution  
>Oracle VM  VirtualBox  
>Firefox  

## grub主题调整
vim /etc/default/grub  
```bash
GRUB_THEME="/usr/share/grub/themes/starfield/theme.txt"
```
grub-mkconfig -o /boot/grub/grub.cfg  

## 安装星际译王
1. pacman -S stardict  
2. 然后打开它（和上面打开优化工具一样，有三种方法可以打开）  
3. 打开之后，首先勾选取词；然后点房子图标，再点下载词典，此时跳转到浏览器，下载金山词霸2011中的“简明英汉词典”和“简明汉英词典”，再下载zh_CN中的“朗道英汉字典”、“朗道汉英字典”、“懒虫简明英汉词典”、“懒虫简明汉英词典”、“计算机词汇”、“现代汉语词典（修正版）”、“汉语成语词典（修正版）”  
4. 下载完之后，在文件所在目录（一般是~/Downloads）下执行：  
```bash
tar -xjvf stardict-powerword2011_1_900-2.4.2.tar.bz2 -C /usr/share/stardict/dic  
tar -xjvf stardict-powerword2011_1_901-2.4.2.tar.bz2 -C /usr/share/stardict/dic  
tar -xjvf stardict-langdao-ec-gb-2.4.2 -C /usr/share/stardict/dic  
tar -xjvf stardict-langdao-ce-gb-2.4.2 -C /usr/share/stardict/dic  
tar -xjvf stardict-lazyworm-ec-2.4.2 -C /usr/share/stardict/dic  
tar -xjvf stardict-lazyworm-ce-2.4.2 -C /usr/share/stardict/dic  
tar -xjvf stardict-kdic-computer-gb-2.4.2 -C /usr/share/stardict/dic  
tar -xjvf stardict-xiandaihanyucidian_fix-2.4.2 -C /usr/share/stardict/dic  
tar -xjvf stardict-hanyuchengyucidian_fix-2.4.2 -C /usr/share/stardict/dic  
```
5. 重启星际译王，点房子图标->词典管理，把这两个词典移到最前面，并把每处顺序都改为计算机词汇、朗道英汉词典、朗道汉英词典、简明英汉词典、简明汉英词典、懒虫简明英汉词典、懒虫简明汉英词典、现代汉语词典、汉语成语词典  
6. 首选项->主窗口->选项，把启动时隐藏主窗口勾选，主窗口->输入，即输即查取消勾选  
> 若下载并配置好以上词典，则忽略以下网络词典的配置  
7. 首选项->网络词典，用账号yottaliu登录，同时把网络浏览器打开网址的命令改为/opt/google/chrome/chrome，把总是使用此命令打开网址勾选  
8. 词典管理->网络词典->添加，在zh_CN中选择以下词典，并按以下顺序排列：  
> 1. 计算机词汇  
> 2. 简明英汉字典  
> 3. 简明汉英字典  
> 4. 懒虫简明英汉词典  
> 5. 懒虫简明汉英词典  
> 6. 现代汉语词典  
> 7. 汉语成语词典  

9. 再次打开gnome-tweak-tool（优化工具）->开机启动程序，添加星际译王  

## 一些好玩的包
* cmatrix------------->字符界面的流星雨动画  
* typespeed---------->测试打字速度的  
* love----------------->图形界面里出现很多爱心  
* tmux---------------->在终端最下面出现一个横条，用来显示日期和时间  
* sl-------------------->屏幕上出现一个字符动画的火车  
* figlet---------------->用户输入字符，终端里输出比较大的字符画面  
* doge---------------->在终端显示一只狗doge  

# gnome重启的技巧
* 方法一：Alt+F2,然后输入r或restart  
* 方法二：pkill -HUP gnome-shell  

# 更多请参考
[WaWei的博客](http://wawei.coding.me)  

# 其他开发环境
对照[archwiki](https://wiki.archlinux.org)安装并配置apache，mysql（现在叫mariaDB），PHP，Adminer，并且把PHP的配置文件中的pdo_mysql（pdo是PHP中连接数据库的方式）和iconv（转换编码的工具）两个扩展打开（即把/etc/php/php.ini中的extension=pdo_mysql.so和extension=iconv.so取消注释）。
