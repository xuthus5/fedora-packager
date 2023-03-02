# fedora-packager

> 目前维护了 fedora 下的 wechat, wxwork, deepin-wine6-stable, deepin-wine-helper 等.

## 添加软件源

```bash
sudo dnf config-manager --add-repo https://download.opensuse.org/repositories/home:xuthus5/Fedora_$(rpm -E %fedora)/home:xuthus5.repo
```

## 安装软件

**注**: 目前不再维护 `Fedora release version < 35` 的版本.

> `fedora-deepin-wine6` 和 `fedora-deepin-wine-helper` 是必装的依赖

- `fedora-deepin-wine6` 用于驱动原生 `wine`
- `fedora-deepin-wine-helper` 用于驱动打包好的程序。

### 前置以来版本安装

```bash
sudo dnf install fedora-deepin-wine6 fedora-deepin-wine-helper -y
```

## 目前维护的软件列表

```bash
# 安装 fedora-deepin-wine6 版本的微信
sudo dnf install fedora-deepin-wechat-wine6 -y
```

| 包名                     | 描述           | 版本                   | 兼容wine5 | 兼容wine6+ |
| ---------------------- | ------------ | -------------------- | ------- | -------- |
| fedora-deepin-wine6    | deepin-wine6 | -                    | 是       | 是        |
| fedora-deepin-wechat-wine6   | 微信           | 3.9.0 | 是       | 是        |
| fedora-deepin-wework   | 企业微信         | 3.1.12.2             | 是       | 是        |
| fedora-deepin-189cloud | 天翼云盘         | 6.3.8.1              | 是       | 否        |
| fedora-deepin-iqiyi    | 爱奇艺          | 7.6.114.2            | 是       | 否        |
| fedora-deepin-pvz      | 植物大战僵尸       | 1.0.0.1              | 是       | 否        |
| fedora-deepin-kugou    | 酷狗音乐         | 9.1.44.1             | 是       | 否        |

## 移植日记

### wine6+ 的移植说明

fedora35+上对 `libpcap` 以及 `openldap` 进行了更新，导致无法对原生的 `deepin-wine` 进行移植。本仓库的 `fedora-deepin-wine6` 对其中基础库依赖做了部分替换，由于无法对其影响作出正确的评估，所以使用时有一定软件崩坏风险，由此产出的一系列问题，本人概不负责。

## 常见问题

> 对于安装后常规的字体等问题请自行解决。

### 修复字体方块问题

#### 方法一

最简单的方法是下载一份已经处理好了的微软雅黑+宋体融合字体

```bash
wget https://images.xuthus.cc/images/fake_simsun.ttc
cp fake_simsun.ttc ~/.deepinwine/Deepin-WeChat/drive_c/windows/Fonts/
# 微信重启之
/opt/apps/com.qq.weixin.deepin/files/run.sh
```

#### 方法二

自定义需要配置的字体

下载 微软雅黑 字体 放置到 `~/.deepinwine/Deepin-WeChat/drive_c/windows/Fonts/` 下

```bash
cp /path/to/MSYH.TTC ~/.deepinwine/Deepin-WeChat/drive_c/windows/Fonts/msyh.ttc
```

设置deepin-wine5下的系统默认字体

```bash
vim ~/.deepinwine/Deepin-WeChat/system.reg

"MS Shell Dlg"="msyh"
"MS Shell Dlg 2"="msyh"

vim ~/.deepinwine/Deepin-WeChat/msyh.reg

REGEDIT4
[HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion\FontLink\SystemLink]
"Lucida Sans Unicode"="msyh.ttc"
"Microsoft Sans Serif"="msyh.ttc"
"MS Sans Serif"="msyh.ttc"
"Tahoma"="msyh.ttc"
"Tahoma Bold"="msyhbd.ttc"
"msyh"="msyh.ttc"
"Arial"="msyh.ttc"
"Arial Black"="msyh.ttc"
```

注册表注册之

```bash
WINEPREFIX=~/.deepinwine/Deepin-WeChat/ deepin-wine5 regedit ~/.deepinwine/Deepin-WeChat/system.reg

WINEPREFIX=~/.deepinwine/Deepin-WeChat/ deepin-wine5 regedit ~/.deepinwine/Deepin-WeChat/msyh.reg
```

### WechatWin.dll文件缺失

下载一份额外的依赖包(该包提供低版本的 deepin-wine6 依赖库)

https://software.opensuse.org//download.html?project=home%3Axuthus5&package=fedora-deepin-extra-lib

```bash
# 忽略冲突安装
sudo rpm -ivh --force fedora-deepin-extra-lib-0.0.1-2.1.x86_64.rpm
# 你也可以直接线上安装
sudo rpm -ivh --force https://download.opensuse.org/repositories/home:/xuthus5/Fedora_$(rpm -E %fedora)/x86_64/fedora-deepin-extra-lib-0.0.1-7.1.x86_64.rpm

# 必须按照如下步骤进行软链接
cd /usr/lib/
sudo ln -sf liblber-2.4.so.2.10.10 liblber-2.4.so.2
sudo ln -sf libldap_r-2.4.so.2.10.10 libldap_r-2.4.so.2
sudo ln -sf libldap_r-2.4.so.2 libldap-2.4.so.2

# 下载wine旧版本的wldap32.dll.so
wget -O wldap32.dll.so https://images.xuthus.cc/images/akrHXou_wldap32.dll.so
sudo mv wldap32.dll.so /opt/deepin-wine6-stable/lib/wldap32.dll.so
```

接下来你可以执行 `/opt/apps/com.qq.weixin.deepin/files/run.sh` 重启微信。

### 我无法运行软件 也无法拿到运行日志？

wine提供 `WINEDEBUG` 环境变量可供你在运行时获取不同通道`channel`的日志信息。
你只需要在你需要运行的软件前 添加 `WINEDEBUG=${log_level}+${channel}` 即可。
详见：[WineHQ:Debug Channels](https://wiki.winehq.org/Debug_Channels)

举一个例子：

```bash
# 打印微信运行时所有通道的error级别信息
# will turn on WARN messages for all channels, in addition to already enabled ERR and FIXME messages.
WINEDEBUG=warn+all /opt/apps/com.qq.weixin.deepin/files/run.sh
```

### 提示 wldap32.dll not found

```bsh
# 自行下载wine旧版本的wldap32.dll.so 
wget https://images.xuthus.cc/images/akrHXou_wldap32.dll.so
sudo mv akrHXou_wldap32.dll.so /opt/deepin-wine6-stable/lib/wldap32.dll.so
```

## 如何打包

参考我的此篇文章: [fedora 打包 wechat RPM 包](https://xuthus.cc/linux/fedora-packaged-wechat-rpm.html)

## 项目地址

项目地址是: [https://build.opensuse.org/project/show/home:xuthus5](https://build.opensuse.org/project/show/home:xuthus5)，构建文件和源代码包都裸露在上面。

## 鸣谢

[OBS](https://build.opensuse.org/) 整个的打包流程依赖 `openSUSE Build Service` 构建服务。

[vufa/deepin-wine-wechat-arch](https://github.com/vufa/deepin-wine-wechat-arch) wechat源包提供者

[deepin-wine-helper](https://aur.archlinux.org/packages/deepin-wine-helper/) deepin-wine-helper源包提供者

[com.qq.weixin.work.deepin](https://aur.archlinux.org/packages/com.qq.weixin.work.deepin/) wework源包提供者

[deepin-wine5](https://aur.archlinux.org/packages/deepin-wine5/) deepin-wine5源包提供者
