## VirtualBox

VirtualBox 是一款开源虚拟机软件。使用者可以在VirtualBox上安装并且执行Solaris、Windows、DOS、Linux、OS/2 Warp、BSD等系统作为客户端操作系统。[官网](https://www.virtualbox.org/)。

## 编译

官网上在 MAC 上编译 VBOX 的[教程文档](https://www.virtualbox.org/wiki/Mac%20OS%20X%20build%20instructions)停留在了远古时期，什么 Mac OSX 10.6、10.7、Xcode 4，已经六七年的历史了，编译器已经有了好多版本的迭代了。2017年有个大佬写了个新的编译的[帖子](https://forums.virtualbox.org/viewtopic.php?f=10&t=83521)，可以覆盖到了 MacOSX 10.13，对于最新的 MacOSX 10.14 Mojave没支持，参考着这个帖子在 Mojave 上完成了编译。

### 编译的先决条件

* **Xcode6.2.dmg** VBOX 在源码里写死了需要 6.2版本的Xcode（不需要安装，应该是需要其中的gcc编译源码）。
* **MacPorts** 用于加载依赖库。
* **VirtualBox 源码** 我使用的是6.0.4版本的

### 步骤

1. 挂载 Xcode6.2，位置`/Volumes/Xcode`。 命令行：`hdiutil attach Xcode_6.2.dmg`；或者直接双击 dmg 文件。
2. 选择原本的Xcode位置 `sudo xcode-select -s /Applications/Xcode.app`，
`sudo xcodebuild -license accept`。
3. 安装 MacPorts 。安装MacPorts后在`~/.profile`中添加`export PATH=\"/opt/local/bin:/opt/local/sbin:\$PATH\"`。
4. 加载依赖库：

	```
	sudo port -N install libidl +universal doxygen texlive texlive-latex-extra \
                          texlive-fonts-extra cdrtools openssl qt56 subversion \
                          docbook-xml docbook-xsl-nons
	```
	其中，libidl 已经不支持MacPorts安装了，使用 Homebrew 安装libidl。
5. 修改 configure 文件，适配 Mojave。函数 `check_darwinversion()`，仿照以前版本添加Mojave的支持就行。

	```
	case "$darwin_ver" in
	18\.*)
        check_xcode_sdk_path "$WITH_XCODE_DIR"
        [ $? -eq 1 ] || fail
        darwin_ver="10.14" # Mojave
        sdk=$WITH_XCODE_DIR/Developer/SDKs/MacOSX10.6.sdk
        cnf_append "VBOX_WITH_MACOSX_COMPILERS_FROM_DEVEL" "1"
        cnf_append "VBOX_PATH_MACOSX_DEVEL_ROOT" "$WITH_XCODE_DIR/Developer"
        ;;
	```
6. 执行配置文件并编译： 

	```
	./configure --disable-hardening --with-xcode-dir=/Volumes/Xcode/Xcode.app --with-openssl-dir=/usr/local/opt/openssl
	. ./env.sh
	kmk
	```
7. 如果不关闭系统完整性保护（SIP），将无法加载运行VirtualBox所需要的内核。`csrutil status` 查看当前状态。关闭SIP需要进入修复模式，重启电脑，电脑响一声之后，同时按住 `cmd+r`键，进入修复模式之后选择：实用工具 -> 终端程序。在终端中输入：
	
	```
	csrutil disable
	reboot
	```
8. 重启之后，加载内核，`/vbox/out/darwin.amd64/release/dist/loadall.sh`。
9. 内核加载完成之后，就可以运行VBox了。

## 遇到问题

#### 加载内核时，遇到 VBoxDrv.kext 中有两个符号未定义：g\_abSUPBuildCert、g\_cbSUPBuildCert。

`/VirtualBox/src/VBox/HostDrivers/Support/darwin/SUPDrv-darwin.cpp：468`源码中确实没有直接定义这两个字段，在`VirtualBox/src/VBox/HostDrivers/Support/Makefile.kmk：128` 这里 if 里的内容看到会生成证书，但是通过打log 发现这个方法并没有走。

在`VirtualBox/Config.kmk：925`中有：`VBOX_WITH_DARWIN_R0_DARWIN_IMAGE_VERIFICATION = 1`

而VBOX\_SIGNING\_MODE未定义，所以这里并不会去生成证书文件，而在 SUPDrv-darwin.cpp中报错方法恰好是在`#ifdef VBOX_WITH_DARWIN_R0_DARWIN_IMAGE_VERIFICATION
` 中，才去进行证书验证。因此直接将`Config.kmk` 中修改为`VBOX_WITH_DARWIN_R0_DARWIN_IMAGE_VERIFICATION = 0`，或者注释。	
	