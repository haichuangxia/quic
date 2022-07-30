# proto-quic项目运行
## 1. 下载proto-quic源代码
首先下载代码,然后进行版本回退.(最近两次commit已经删库跑路了,因此要回退版本)
```
git clone https://github.com/google/proto-quic.git
git reset --hard 9f1ab92724ce2647585e62b0e8155ee7368697ce
```

## 2. 第一次编译下载相关依赖
下载依赖时,注意apt源,我电脑上国内清华源比阿里源快点.
```
cd proto-quic
export PROTO_QUIC_ROOT=`pwd`/src
export PATH=$PATH:`pwd`/depot_tools
./src/build/install-build-deps.sh
```

## 3. 构建QUIC client和server
```
cd src
gn gen out/Default
```
构建成功后,会显示:
```
.gclient file in parent directory /home/qing/proto-quic might not be the file you want to use
Done. Made 276 targets from 97 files in 107ms
```

## 4. 编译
编译需要使用ninja工具.命令为:`ninja -C out/Default quic_client quic_server net_unittests`

编译成功之后会在`proto-quic/src/out/Default`目录下生成quic-server和quic-client等文件

# 项目运行中出现的问题
## 1\. 不能重新安装XXX，因为无法下载它/依赖XXX，但是它将不会被安装
### 问题描述
使用ubuntu16.04LTS版本下载相关依赖时候，报错：

```Plain Text
It produces the following output:
正在读取软件包列表... 完成
正在分析软件包的依赖关系树       
正在读取状态信息... 完成       
注意，选中 'unity-gtk2-module' 而非 'appmenu-gtk'
不能重新安装 libfontconfig1，因为无法下载它。
.......
不能重新安装 libcups2，因为无法下载它。
有一些软件包无法被安装。如果您用的是 unstable 发行版，这也许是
因为系统无法达到您要求的状态造成的。该版本中可能会有一些您需要的软件
包尚未被创建或是它们已被从新到(Incoming)目录移出。
下列信息可能会对解决问题有所帮助：

下列软件包有未满足的依赖关系：
 cmake : 依赖: libcurl3 (>= 7.16.2) 但是它将不会被安装 
 linux-libc-dev : 破坏: linux-libc-dev:i386 (!= 4.4.0-186.216) 但是 4.4.0-21.37 正要被安装
E: 错误，pkgProblemResolver::Resolve 发生故障，这可能是有软件包被要求保持现状的缘故。

You will have to install the above packages yourself.
```

### 解决方法
出现这个问题的原因在于版本冲突，百度了一下，主要有两种方法:使用aptitude安装或者更换下载源.使用aptitude来安装并不起作用，一般来说，版本冲突，那就手动安装版本低的那个。而我没有换过下载源，尝试更换下载源方法，通过更换到清华源得以正常下载相关依赖（阿里源太慢，半天加载不出来）。更换清华园的步骤为：

1. 备份原来的sources.list文件

ubuntu的源信息在`/etc/apt/sources.list` 文件中，保险起见，先把原来的文件备份一下:

` sudo cp sources.list sources.list.backu`

2. 修改sources.list文件的内容
通过`sudo vi /etc/apt/sources.list` 命令修改`sources.list` 文件的内容。清空文件的内容，在文件中写入：

```Plain Text
# deb cdrom:[Ubuntu 16.04 LTS _Xenial Xerus_ - Release amd64 (20160420.1)]/ xenial main restricted
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial main restricted
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-updates main restricted
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial universe
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-updates universe
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial multiverse
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-updates multiverse
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-backports main restricted universe multiverse
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-security main restricted
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-security universe
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-security multiverse
```
## 2\. 正在设定ttf-mscorefonts-installer
当通过源更可以正常下载之后，有可能弹出`正在设定ttf-mscorefonts-installer` 的界面，让同意EULA。同意的方法是：

1. 页面拉到最下方
2. 点击`Tab` 键选中确认
3. `Enter`



## 3\.  无法使用`./proto_quic_tools/sync.sh` 进行同步
### 问题描述
可以通过`./proto_quic_tools/sync.sh` 命令,使用脚本进行同步,以便于获取最新的代码.

```Plain Text
export PROTO_QUIC_ROOT=`pwd`/src
export PATH=$PATH:`pwd`/depot_tools
./proto_quic_tools/sync.sh
```
### 问题描述: 无法下载`binutils.tar.gz2`
```Plain Text
qing@QB:~/proto-quic$    ./proto_quic_tools/sync.sh
_____ running src/third_party/binutils/download.py
Downloading /home/qing/proto-quic/src/third_party/binutils/Linux_x64/binutils.tar.bz2
```
### 解决方法:使用梯子
这个项目是谷歌的,很多东西需要使用梯子才能上网.有一点需要注意:挂了代理之后,浏览器可以科学上网,但是终端还不行,因此需要进行设置,让终端也可以科学上网.在终端配置代理可以参考:

[如何为实验室服务器配置终端代理（Clash for Linux)](https://juejin.cn/post/7054941050216906760)

[Linux终端设置代理](https://v2raytech.com/linux-cmd-set-proxy/)

需要注意的是,如果使用这个方法为终端设置代理,需要在同一个终端下进行操作,也就是需要在同一个终端下,进行终端代理配置和`proto-quic` 的下载和构建.

## 4.  无法构建
1. 在构建时报错

```Plain Text
qing@QB:~/proto-quic/src$  gn gen out/Default && ninja -C out/Default quic_client quic_server net_unittests
.gclient file in parent directory /home/qing/proto-quic might not be the file you want to use
ERROR at //build/config/linux/pkg_config.gni:103:17: Script returned non-zero exit code.
    pkgresult = exec_script(pkg_config_script, args, "value")
                ^----------
Current dir: /home/qing/proto-quic/src/out/Default/
Command: python -- /home/qing/proto-quic/src/build/config/linux/pkg-config.py -s /home/qing/proto-quic/src/build/linux/debian_jessie_amd64-sysroot -a x64 glib-2.0 gmodule-2.0 gobject-2.0 gthread-2.0
Returned 1.
stderr:

Package glib-2.0 was not found in the pkg-config search path.
Perhaps you should add the directory containing `glib-2.0.pc'
to the PKG_CONFIG_PATH environment variable
No package 'glib-2.0' found
Package gmodule-2.0 was not found in the pkg-config search path.
Perhaps you should add the directory containing `gmodule-2.0.pc'
to the PKG_CONFIG_PATH environment variable
No package 'gmodule-2.0' found
Package gobject-2.0 was not found in the pkg-config search path.
Perhaps you should add the directory containing `gobject-2.0.pc'
to the PKG_CONFIG_PATH environment variable
No package 'gobject-2.0' found
Package gthread-2.0 was not found in the pkg-config search path.
Perhaps you should add the directory containing `gthread-2.0.pc'
to the PKG_CONFIG_PATH environment variable
No package 'gthread-2.0' found
Traceback (most recent call last):
  File "/home/qing/proto-quic/src/build/config/linux/pkg-config.py", line 219, in <module>
    sys.exit(main())
  File "/home/qing/proto-quic/src/build/config/linux/pkg-config.py", line 138, in main
    prefix = GetPkgConfigPrefixToStrip(options, args)
  File "/home/qing/proto-quic/src/build/config/linux/pkg-config.py", line 80, in GetPkgConfigPrefixToStrip
    "--variable=prefix"] + args, env=os.environ)
  File "/usr/lib/python2.7/subprocess.py", line 574, in check_output
    raise CalledProcessError(retcode, cmd, output=output)
subprocess.CalledProcessError: Command '['pkg-config', '--variable=prefix', 'glib-2.0', 'gmodule-2.0', 'gobject-2.0', 'gthread-2.0']' returned non-zero exit status 1

See //build/config/linux/BUILD.gn:83:3: whence it was called.
  pkg_config("glib") {
  ^-------------------
See //base/BUILD.gn:1528:26: which caused the file to be included.
      linux_configs += [ "//build/config/linux:glib" ]
                         ^--------------------------
```
这个错误,原理应该差不多,同样是国内无法访问的问题.参照上述的终端设置代理.

## 5. 无法编译
### ninja版本过低
使用ninja进行编译的时候,如果出现ninja版本的问题,就需要自己去升级ninja,报错示例为:
```
ninja: Entering directory `out/Default'
ninja: fatal: ninja version (1.6.0) incompatible with build file ninja_required_version version (1.7.2).
```

升级ninja可以通过apt或者使用源代码进行编译:
1. 使用apt工具进行升级
` sudo apt install ninja`

OR

`sudo apt install ninja-build`

2. 下载源代码手动编译安装
使源代码手动安装ninja参考:[ninja-build环境安装](https://www.cnblogs.com/freeweb/p/9334612.html).详细步骤为:
1. 下载解压re2c
re2c的github地址为:[re2c releases](https://github.com/skvadrik/re2c/releases)
 下载源代码,zip格式或者tar.gz都可以,以tar.gz为例:
`tar -zxvf XXX.tar.gz`

2. 安装re2c
```
cd XXX # 进入re2c的解压目录
sudo ./autogen.sh
sudo ./configure
sudo make
sudo make install
```
3. 下载ninja源代码
ninja的github地址为:https://github.com/ninja-build/ninja,可以在[ninja releases](https://github.com/ninja-build/ninja/releases)上下载源代码.或者是`git clone https://github.com/ninja-build/ninja.git`

4. 构建ninja
按照ninja中README.md的介绍,这里使用python来进行构建.
```
cd ninja-xxx # 进入ninja源代码目录
sudo ./configure.py --bootstrap # 编译
sudo cp ninja /usr/bin/ # 创建链接
sudo cp ninja /usr/sbin/ # 创建链接
```

自己使用apt进行安装,但是版本不对,因此还是选择了手动进行编译.

## 6. 运行服务器和客户端异常
### 1. Error: QUIC_PROOF_INVALID
详细的报错结果为:
```
$ ./out/Default/quic_client --host=127.0.0.1 --port=6121 https://www.example.org/
[0730/085554.541160:ERROR:cert_verify_proc_nss.cc(902)] CERT_PKIXVerifyCert for www.example.org failed err=-8179
[0730/085554.541416:WARNING:proof_verifier_chromium.cc(452)] Failed to verify certificate chain: net::ERR_CERT_AUTHORITY_INVALID
Failed to connect to 127.0.0.1:6121. Error: QUIC_PROOF_INVALID
```

# Reference
\[ ubuntu16.04 下编译和运行 c++ proto-quic quic\_server quic\_client\](he)
