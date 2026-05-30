# Install-and-launch-.deb-file-in-manjaro-system-using-shortcut-command

该文档演示如何在 Manjaro 中，在无网络代理、无法通过 AUR 仓库下载的情况下，手动下载 `.deb` 文件，并使用快捷命令成功启动程序。
本文以安装 **CC-Switch** 为例。（若有更棒的方法欢迎PR😊）

---

 # 背景说明
 
在 Manjaro / Arch 系 Linux 中，通常推荐通过以下方式安装软件：
```bash
sudo pacman -S package-name
```
或通过 AUR：
```bash
yay -S package-name
```

但是在某些网络环境下，由于没有代理或访问 GitHub / AUR 较慢，可能无法正常通过 AUR 下载源码。
<img width="1588" height="1326" alt="image" src="https://github.com/user-attachments/assets/11675f50-586e-4459-b6ea-181a7264ac04" />

>注意：
>`.deb `是 Debian / Ubuntu 系统的软件包格式。
>Manjaro 并不原生使用 `.deb`，因此本文方法本质上是手动解包 .deb 并把文件释放到系统对应目录中。

# 下载`.deb`文件
以**CC-Switch**为例，从[官方 Release ](https://github.com/farion1231/cc-switch)下载`.deb`文件
```bash
CC-Switch-xxx-Linux-x86_64.deb
```

例如文件位于：
```bash
~/Downloads/CC-Switch-xxx-Linux-x86_64.deb
```

进入下载目录：
```bash
cd  ~/Downloads/CC-Switch-xxx-Linux-x86_64.deb
```

# 创建解包目录

为了避免文件混乱，先创建一个专门的目录：
```bash
mkdir -p ~/cc-switch-deb
cd ~/cc-switch-deb
```

# 解包`.deb`文件

使用`bsdtar`解包`.deb`：
```bash
bsdtar -xf ~/Downloads/CC-Switch-xxx-Linux-x86_64.deb
```

解包后使用`ls`查看当前目录，并通常可以看到类似内容：
```bash
ls
control.tar.gz
data.tar.gz
debian-binary
```

>其中：  
>`control.tar.*`：包的控制信息  
>`data.tar.*`：真正要安装到系统中的文件  
>`debian-binary`：Debian 包格式标识文件

# 安装`.deb`中的文件到系统 
将`data.tar.*`解压到根目录`/`:
```bash
sudo bsdtar -xpf data.tar.* -C /
```

>其中：
>`-x`：解压
>`-p`：保留文件权限
>`-f`:指定压缩包文件
>`-C/`:解压到系统根目录

安装完成后，检查主程序是否存在：
```bash
ls -l /usr/bin/cc-switch
```

如果看到类似输出：
```bash
-rwxr-xr-x 1 root root ... /usr/bin/cc-switch
```
说明程序已经安装到系统中。

# 运行程序

可以直接运行：
```bash
cc-switch
```

# 查看桌面启动文件

由于 CC-Switch 的 desktop 文件名中包含空格：
```bash
CC Switch.desktop
```

所以查看时不能直接写：
```bash
sudo cat /usr/share/applications/CC Switch.desktop
```
否则shell会将这行命令识别为两个文件：
```bash
/usr/share/applications/CC
Switch.desktop
```

正确写法如下两种：
```bash
# 方法一：使用引导
sudo cat "/usr/share/applications/CC Switch.desktop"
# 方法二：转义空格
sudo cat /usr/share/applications/CC\ Switch.desktop
```

>其中
>`Exec`=后面的内容为桌面环境启动程序时执行的命令。

# 🤔如果cc-switch启动的是旧AppImage(以下步骤可能对你有帮助)
比如：
执行：
```bash
cc-switch
```

可能会出现报错：
```bash
/home/username/.local/bin/cc-switch: 行 2: /home/username/Applications/CC-Switch-xxx-Linux-x86_64 .AppImage: 没有那个文件或目录
```

说明此时执行的并不是我们当前需要的主程序路径：
```bash
/usr/bin/cc-switch
```

而是之前创建过的旧快捷脚本。

# 检查当前 cc-switch 命令来源

执行：
```bash
command -v cc-switch
```

如果输出并不是：
```bash
/usr/bin/cc-switch
```
说明 ~/.local/bin 里的旧脚本优先级高于 /usr/bin/cc-switch。

也可以查看所有同名命令：
```bash
type -a cc-switch
```

## 删除旧快捷命令方式
删除旧的本地快捷命令：
```bash
rm -f ~/.local/bin/cc-switch
```

清理shell命令缓存：
```bash
hash -r
```

然后重新检查：
```bash
command -v cc-switch
```

此时输出应该类似于：
```bash
/usr/bin/cc-switch
```

再次运行：
```bash
cc-switch
```

# 🤨为你的cc-switch创建一个快捷命令

首先创建本地命令目录：
```bash
mkdir -p ~/.local/bin
```

创建快捷脚本：
```bash
cat > ~/.local/bin/ccs <<'EOF'
#!/usr/bin/env bash
exec /usr/bin/cc-switch "$@"
EOF
```

添加执行权限：
```bash
chmod +x ~/.local/bin/ccs
```

确保 `~/.local/bin `在 PATH 中：
```bash
echo $PATH | tr ':' '\n' | grep "$HOME/.local/bin"
```

之后就可以使用：
```bash
ccs
```
打开CC-Switch。
