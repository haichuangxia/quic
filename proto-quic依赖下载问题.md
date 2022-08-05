友情提示:`proto-quic`项目需要在ubutn-14,16,17或者是debian8系统下运行,请确认自己的系统版本!
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

## 5. 生成并添加证书
### 1. 生成CA证书:
```
cd net/tools/quic/certs # 切换到目录
./generate-certs.sh # 生成ca证书,生成后会到证书所在路径
cd - # 切换回上一次所在路径,即proto-quic/src目录
```

### 2. 添加证书到系统
使用以下命令将生成的证书添加到系统.

```openssl x509 -outform der -in net/tools/quic/certs/out/2048-sha256-root.pem -out net/tools/quic/certs/out/2048-sha256-root.crt
certutil -d sql:$HOME/.pki/nssdb -A -t "C,," -n quic -i net/tools/quic/certs/out/2048-sha256-root.crt
```


注意:如果重新生成了证书,那么也要重复此操作,将新的证书添加到系统.


## 6. 准备资源
### 1. `www.example.org`准备测试数据
```

mkdir ~/quic-data
cd ~/quic-data
wget -p --save-headers https://www.example.org
```

### 2. 修改数据
下好之后的index.heml需要手动的调整head部分:
- 移除(如果存在的话)："Transfer-Encoding: chunked"
- 移除(如果存在的话)："Alternate-Protocol: ..."
- 添加：X-Original-Url: https://www.example.org/

调整后的index.html的header部分如下:
```
HTTP/1.1 200 OK^M
Age: 573470^M
Cache-Control: max-age=604800^M
Content-Type: text/html; charset=UTF-8^M
Date: Sat, 30 Jul 2022 17:41:16 GMT^M
Etag: "3147526947+ident"^M
Expires: Sat, 06 Aug 2022 17:41:16 GMT^M
Last-Modified: Thu, 17 Oct 2019 07:18:26 GMT^M
Server: ECS (sab/5697)^M
Vary: Accept-Encoding^M
X-Cache: HIT^M
Content-Length: 1256^M
X-Original-Url: https://www.example.org/
^M
<!doctype html>
```


## 7.运行quic server和client
启动quic-server和quic-server建议参考:[Playing with QUIC](https://www.chromium.org/quic/playing-with-quic/)

### 1. 启动server

```

./out/Default/quic_server \
  --quic_in_memory_cache_dir=~/quic-data/www.example.org \
  --certificate_file=net/tools/quic/certs/out/leaf_cert.pem \
  --key_file=net/tools/quic/certs/out/leaf_cert.pkcs8
```
注意:`--quic_in_memory_cache_dir`参数指的是下载好的测试数据`www.example.org/index`的路径

### 2. 启动client

使用quic-client以QUIC协议向quic-server请求测试文件:
```
./out/Default/quic_client --host=127.0.0.1 --port=6121 https://www.example.org/
```

客户端访问成功则应显示:

```
qing@QB:~/proto-quic/src$ ./out/Default/quic_client --host=127.0.0.1 --port=6121 https://www.example.org/
Connected to 127.0.0.1:6121
Request:
headers:
{
  :method GET
  :scheme https
  :authority www.example.org
  :path /
}
body: 

Response:
headers: 
{
  :status 200
  age 573470
  cache-control max-age=604800
  content-type text/html; charset=UTF-8
  date Sat, 30 Jul 2022 17:41:16 GMT
  etag "3147526947+ident"
  expires Sat, 06 Aug 2022 17:41:16 GMT
  last-modified Thu, 17 Oct 2019 07:18:26 GMT
  server ECS (sab/5697)
  vary Accept-Encoding
  x-cache HIT
  content-length 1256
  x-original-url https://www.example.org/
}

body: <!doctype html>
<html>
<head>
    <title>Example Domain</title>

    <meta charset="utf-8" />
    <meta http-equiv="Content-type" content="text/html; charset=utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <style type="text/css">
    body {
        background-color: #f0f0f2;
        margin: 0;
        padding: 0;
        font-family: -apple-system, system-ui, BlinkMacSystemFont, "Segoe UI", "Open Sans", "Helvetica Neue", Helvetica, Arial, sans-serif;
        
    }
    div {
        width: 600px;
        margin: 5em auto;
        padding: 2em;
        background-color: #fdfdff;
        border-radius: 0.5em;
        box-shadow: 2px 3px 7px 2px rgba(0,0,0,0.02);
    }
    a:link, a:visited {
        color: #38488f;
        text-decoration: none;
    }
    @media (max-width: 700px) {
        div {
            margin: 0 auto;
            width: auto;
        }
    }
    </style>    
</head>

<body>
<div>
    <h1>Example Domain</h1>
    <p>This domain is for use in illustrative examples in documents. You may use this
    domain in literature without prior coordination or asking for permission.</p>
    <p><a href="https://www.iana.org/domains/example">More information...</a></p>
</div>
</body>
</html>

trailers: {}
Request succeeded (200).
```

# 项目运行中出现的问题
## 1. 不能重新安装XXX，因为无法下载它/依赖XXX，但是它将不会被安装
### 1. 问题描述
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

### 2. 解决方法
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
## 2. 设定ttf-mscorefonts-installer
当通过源更可以正常下载之后，有可能弹出`正在设定ttf-mscorefonts-installer` 的界面，让同意EULA。同意的方法是：

1. 页面拉到最下方
2. 点击`Tab` 键选中确认
3. `Enter`



## 3. 无法使用`./proto_quic_tools/sync.sh` 进行同步
### 1. 问题描述
使用`./proto_quic_tools/sync.sh` 进行同步,以便于获取最新的代码.

```Plain Text
export PROTO_QUIC_ROOT=`pwd`/src
export PATH=$PATH:`pwd`/depot_tools
./proto_quic_tools/sync.sh
```

但是有可能出现问题描: 无法下载`binutils.tar.gz2`
```Plain Text
qing@QB:~/proto-quic$    ./proto_quic_tools/sync.sh
_____ running src/third_party/binutils/download.py
Downloading /home/qing/proto-quic/src/third_party/binutils/Linux_x64/binutils.tar.bz2
```

### 2. 解决方法:使用梯子
这个项目是谷歌的,需要挂梯子才能访问.有一点需要注意:挂了代理之后,浏览器可以科学上网,但是终端还不行,因此需要进行设置,让终端也可以科学上网.在终端配置代理可以参考:

- [如何为实验室服务器配置终端代理（Clash for Linux)](https://juejin.cn/post/7054941050216906760)

- [Linux终端设置代理](https://v2raytech.com/linux-cmd-set-proxy/)

此外,如果使用这个方法为终端设置代理,需要在同一个终端下进行操作,也就是需要在同一个终端下,进行终端代理配置和`proto-quic` 的下载和构建.

## 4.  无法构建
在构建时报错

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
### 1. ninja版本过低
使用ninja进行编译的时候,如果出现ninja版本的问题,就需要自己去升级ninja,报错示例为:
```
ninja: Entering directory `out/Default'
ninja: fatal: ninja version (1.6.0) incompatible with build file ninja_required_version version (1.7.2).
```

升级ninja可以通过apt或者使用源代码进行编译:
#### 1. 使用apt工具进行升级

` sudo apt install ninja`

OR

`sudo apt install ninja-build`

#### 2. 下载源代码手动编译安装
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
ninja的github地址为:https://github.com/ninja-build/ninja ,可以在[ninja releases](https://github.com/ninja-build/ninja/releases)上下载源代码.或者是`git clone https://github.com/ninja-build/ninja.git`

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
#### 1. 问题描述
详细的报错结果为:
```
$ ./out/Default/quic_client --host=127.0.0.1 --port=6121 https://www.example.org/
[0730/085554.541160:ERROR:cert_verify_proc_nss.cc(902)] CERT_PKIXVerifyCert for www.example.org failed err=-8179
[0730/085554.541416:WARNING:proof_verifier_chromium.cc(452)] Failed to verify certificate chain: net::ERR_CERT_AUTHORITY_INVALID
Failed to connect to 127.0.0.1:6121. Error: QUIC_PROOF_INVALID
```
#### 2. 解决方法
客户端需要在系统中添加证书,具体的操作方法参考:
- [Linux Cert Management](https://chromium.googlesource.com/chromium/src/+/HEAD/docs/linux/cert_management.md#linux-cert-management)

- [为linux系统导入根证书](http://notes.maxwi.com/2017/10/14/certificates-import-linux/)


##### 1. 证书文件介绍
证书文件一般是pem格式的(使用ASCII进行编码),后缀名一般为crt或者pem.常见的CA 证书后缀名一般是cer.ca证书可以通过openssl命令导出为ca证书.

.crt文件需要导入到`/etc/ca-certificates.conf`文件夹中.在证书的存放路径

##### 2. proto-quic中导入证书
使用命令将证书添加到系统:

```
openssl x509 -outform der -in net/tools/quic/certs/out/2048-sha256-root.pem -out net/tools/quic/certs/out/2048-sha256-root.crt
certutil -d sql:$HOME/.pki/nssdb -A -t "C,," -n quic -i net/tools/quic/certs/out/2048-sha256-root.crt
```

参考:
- [quic编译和运行](https://www.geek-share.com/detail/2794339952.html)


### 2. quic server 404
#### 1. 问题描述
在运行quic-server后,启动quic-client发送消息时,出现:

```
qing@QB:~/proto-quic/src$ ./out/Default/quic_client --host=127.0.0.1 --port=6121 https://www.example.org/
Connected to 127.0.0.1:6121
Request:
headers:
{
  :method GET
  :scheme https
  :authority www.example.org
  :path /
}
body: 

Response:
headers: 
{
  :status 404
  content-length 14
}

body: file not found
trailers: {}
Request failed (404).
```

#### 2. 分析
可以连接到服务器,说明链接没有问题,但是服务器返回的是404,那么就是资源设置出现了问题,即`www.example.org/index`设置出错.

#### 问题解决
需要按照提示,对`www.example.org`进行编辑.调整如下
1. 移除(如果存在的话)："Transfer-Encoding: chunked"

2. 移除(如果存在的话)："Alternate-Protocol: ..."

3. 添加：X-Original-Url: https://www.example.org/

# Reference
1. [ubuntu16.04 下编译和运行 c++ proto-quic quic_server quic_client](https://www.pengrl.com/p/31237/)
2. [Playing with QUIC](https://www.chromium.org/quic/playing-with-quic/)
3. [Linux Cert Management](https://chromium.googlesource.com/chromium/src/+/HEAD/docs/linux/cert_management.md#linux-cert-management)
4. [使用quic](https://www.shuzhiduo.com/A/kjdwNbPr5N/)
5. [quic编译和运行](https://www.geek-share.com/detail/2794339952.html)

