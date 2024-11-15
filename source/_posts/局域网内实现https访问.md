---
title: 局域网内实现https访问
date: 2024-11-14
categories:
  - 实战
  - 局域网相关
tags:
  - https
  - 局域网
  - ssl证书
  - mkcert
  - openssl
---

# 背景

在内网环境下，能否实现通过 https 的方式访问内网服务器提供的服务呢？且不希望看到浏览器的证书警告。答案是肯定的，我们可以在这台服务器上使用 `mkcert` 生成 HTTPS 证书，并<font color=red>在所有需要访问该服务的客户端 PC 上安装**根证书 **</font> ！！！

### 创建根证书的意义

当我们说 **创建根证书** 时，指的是生成一个 **自签名证书**，这个证书本身是“根证书颁发机构（CA）”的身份象征，意味着它是信任链的起点。在公钥基础设施（PKI）中，根证书（CA 证书）是所有信任证书的基础。

#### 为什么要有根证书？

1. **信任链**：
   - 现代浏览器和操作系统都建立了 **信任链**。浏览器或者操作系统通过验证证书的签发方来判断一个网站的安全性。如果证书是由一个受信任的根证书颁发的，那么它就会被认为是可信的。
   - 如果我们只是生成一个普通的证书和私钥，而没有根证书，那么生成的证书不会被浏览器信任，用户就会看到“不受信任的证书”的警告。
2. **自签名证书的应用场景**：
   - 在 **本地开发** 环境或者 **企业内部系统** 中，可能不希望依赖外部的证书颁发机构（CA）来申请和管理证书，这时候就可以使用 `mkcert` 来创建一个 **本地根证书**。
   - `mkcert` 会创建一个 **本地自签名根证书**，并将它安装到你的操作系统或者浏览器的信任根证书存储中。这样，你自己生成的证书就会被信任，不会触发浏览器的警告。

### 证书和私钥的生成

直接生成证书和私钥当然是可以的，但没有根证书的情况下，浏览器和操作系统无法识别该证书的有效性，因为它不属于任何一个受信任的证书链。

`mkcert` 通过 **创建根证书并信任它**，来让生成的证书能够在浏览器中被信任。

- 如果只是生成一个证书和私钥，而没有创建根证书，那么它会被视为自签名证书。即使你自己信任它，浏览器依然会警告“不受信任的证书”。
- 通过 `mkcert` 创建根证书后，这个根证书被添加到信任的证书存储中，这样 **所有由这个根证书签发的证书** 都会被自动信任。

# 方案一

优点：快捷、省事

缺点：有效期不可控，默认2年3个月

## 第一步：安装mkcert

- 下载地址：https://github.com/FiloSottile/mkcert

我这里的测试服务器是windows server的，我就在当前服务器上安装 [mkcert-v1.4.4-windows-amd64.exe ](https://github.com/FiloSottile/mkcert/releases/download/v1.4.4/mkcert-v1.4.4-windows-amd64.exe) 就行，并将名字重新命名为mkcert.exe即可，把这个mkcert.exe的存放路径配置到环境变量，方便在任何地方使用命令，这里说的安装就是放在一个目录下即可

## 第二步：生成根证书

```bat
# 安装根证书
mkcert -install

# 保存路径
# macOS/Linux：$HOME/.mkcert
# Windows：%LOCALAPPDATA%\mkcert
```

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20241114/Snipaste_2024-11-14_10-02-15.4ro0fayvfs80.webp)

把生成的rootCA.pem拷贝出来，因为这个要在**第四步**安装到客户端PC

到这里这一步就结束了，这个命令已经默认把证书安装到了本机了，所以在本机不用管**第四步**了

## 第三步：生成证书和密钥，并配置

生成证书时，指定一个或多个域名（可以是 localhost、IP 地址或你自己定义的域名）。

```bat
mkcert example.com "*.example.com" localhost 127.0.0.1 ::1
```

这个命令会为以下域名生成证书：

- `example.com`
- `*.example.com`（包含子域名）
- `localhost`
- `127.0.0.1`（IPv4 地址）
- `::1`（IPv6 地址）

`mkcert` 会为这些域名生成 `example.com+2.pem` 和 `example.com+2-key.pem` 文件：

- `.pem` 文件是公钥证书
- `.key` 文件是私钥

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20241114/Snipaste_2024-11-14_10-07-42.5thyrtqr2840.webp)

把证书配置到服务中去，这里就不再详细讲了

## 第四步：在客户端PC上安装根证书

Windows 提供了 **证书管理器** 工具，可以用来查看和管理证书。你可以将 `.pem` 文件导入证书存储区。

步骤：

1. **打开证书管理器**：

   - 按下 **Win + R**，在运行框中输入 `mmc` 并按 **Enter**。
   - 在 MMC（Microsoft Management Console）中，点击 **文件** > **添加/删除管理单元**。
   - 在管理单元窗口中，选择 **证书**，点击 **添加**。
   - 选择 **计算机账户**，然后点击 **下一步** > **完成**。

   ![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20241114/Snipaste_2024-11-14_10-15-28.4yb6iex6p8s0.webp)

2. **导入证书**：

   - 在左侧窗格中，展开 **证书** > **受信任的根证书颁发机构**。
   - 右键点击 **证书** 文件夹，选择 **所有任务** > **导入**。
   - 在导入向导中，点击 **浏览**，选择你的 `rootCA.pem` 文件。
   - 点击 **下一步** 并完成导入。

   ![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20241114/Snipaste_2024-11-14_10-17-52.6x3q37f67x40.webp)

3. 完成后，你可以查看证书是否已成功添加到 **证书管理器** 中。

## 第五步：访问服务

至此就完成了，查看证书有效期为2年3个月

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20241114/Snipaste_2024-11-14_10-19-04.25cvhbr6et6o.webp)

# 方案二

mkcert 本身是不能控制生成的根证书的有效期的，那么我们可以通过openssl来生成自定义有效期的根证书

优点：有效期可控

缺点：不方便、需要专业知识

## 1. 下载 OpenSSL

目前，OpenSSL 官方不提供直接的 Windows 版安装包，但可以通过第三方编译的版本（例如 **Shining Light Productions** 提供的安装包）来安装。你可以从以下链接下载 Windows 版的 OpenSSL：

- Shining Light Productions: https://slproweb.com/products/Win32OpenSSL.html

下载页面上通常会提供 **32 位** 和 **64 位** 的版本，根据你的系统选择合适的版本下载。

## 2. 安装 OpenSSL

1. 下载完成后，运行安装程序。
2. 安装过程中的关键选项：
   - 选择安装目录。建议安装到 `C:\OpenSSL-Win64` 或 `C:\OpenSSL-Win32`。
   - 选择是否将 OpenSSL 二进制文件目录添加到系统 PATH 中（推荐勾选）。
   - 选择是否安装 Visual C++ Redistributable，如果系统已经有该组件，可以跳过。
3. 完成安装后，如果安装时选择添加了 OpenSSL 到系统 PATH，则无需手动设置。如果没有，则需要手动添加 OpenSSL 的安装目录到 PATH 环境变量中。

## 3. 配置 PATH 环境变量（如有需要）

如果在安装过程中没有勾选“添加 OpenSSL 到系统 PATH”，可以手动配置环境变量：

1. **右键“此电脑”** 或 **“我的电脑”** > **属性**。
2. 点击 **高级系统设置** > **环境变量**。
3. 在“系统变量”部分，找到 **Path**，双击打开。
4. 点击 **新建**，然后输入 OpenSSL 的安装目录路径（例如 `C:\OpenSSL-Win64\bin`）。
5. 确认后关闭窗口。

## 4. 验证 OpenSSL 安装

1. 打开命令提示符（CMD）或 PowerShell。

2. 输入以下命令检查是否安装成功：

   ```
   bash
   
   复制代码
   openssl version
   ```

3. 如果安装成功，会显示 OpenSSL 的版本信息。

## 5. 准备配置文件

1. CA.cnf

```
[ req ]
distinguished_name  = req_distinguished_name
x509_extensions     = root_ca
 
[ req_distinguished_name ]
 
# 以下内容可随意填写
countryName             = CN (2 letter code)
countryName_min         = 2
countryName_max         = 2
stateOrProvinceName     = shanxi
localityName            = taiyuan
0.organizationName      = zhiboredian
organizationalUnitName  = technology 
commonName              = zhibo
commonName_max          = 64
emailAddress            =  
emailAddress_max        = 64
 
[ root_ca ]
basicConstraints        = critical, CA:true
```

2. ServerCA.ext

```
subjectAltName = @zhibo
extendedKeyUsage = serverAuth
 
[zhibo]
 
# 域名，如有多个用DNS.2,DNS.3…来增加
DNS.1 = test2.com
# IP地址
# IP.1 = 192.168.137.1
```

## 6. 生成根证书

```sh
openssl req -x509 -newkey rsa:2048 -out rootCA.cer -outform PEM -keyout rootCA.pvk -days 365 -verbose -config CA.cnf -nodes -sha256 -subj "/CN=ZhiBo RSA SSL CA 2024/O=ZhiBo Inc/OU=xxx.xxx.xx"
```

- `-x509` 表示输出一个X.509证书，`x509` 是证书的标准格式。
- `-newkey rsa:2048` 表示生成一个新的密钥对，`rsa:2048` 表示使用 RSA 算法，并且密钥长度为 2048 位。RSA 是一种常用的非对称加密算法。
- `-out rootCA.cer` 指定了输出的证书文件名。
- `-outform PEM` 指定证书的输出格式为 PEM（Base64 编码的 ASCII 格式）。PEM 格式常用于证书文件的存储。
- `-keyout rootCA.pvk`这个选项指定了生成的私钥文件的输出名称。在这里，私钥将保存为 `rootCA.pvk` 文件。

- `-days 365`这个选项指定证书的有效期为 365 天。自签名证书的有效期通常较短，主要用于测试或内部使用。

- `-verbose`启用详细模式，在执行过程中输出更多的调试信息。

- `-config CA.cnf`指定 OpenSSL 使用的配置文件。在这里，`CA.cnf` 是一个配置文件，通常包含有关证书生成的详细设置，例如扩展字段、路径等。

-  `-nodes`这个选项表示生成的私钥不加密。通常情况下，私钥会被加密以增加安全性，但此选项用于跳过私钥加密，通常用于自动化部署。

- `-sha256`这个选项指定使用 SHA-256 哈希算法来生成证书的签名。SHA-256 是一种广泛使用的安全哈希算法，提供了较强的抗碰撞性。

-  `-subj "/CN=ZhiBo RSA SSL CA 2024/O=ZhiBo Inc/OU=xxx.xxx.xx"`这个选项设置证书的主题（Subject），即证书的基本信息。`/CN=ZhiBo RSA SSL CA 2024` 是证书的通用名称（Common Name），`/O=ZhiBo Inc` 是组织名称（Organization），`/OU=xxx.xxx.xx` 是组织单位（Organizational Unit）。这通常用于标识证书的发行机构或所有者，可设置为官网。

## 7.  生成一个新的证书请求

生成的证书请求（`ServerCA.req`）通常会被提交给证书颁发机构（CA）进行签发。如果这是自签名证书的请求，你可以自己签发证书，也可以使用该请求文件让其他证书颁发机构签发证书。

```sh
openssl req -newkey rsa:2048 -keyout ServerCA.pvk -out ServerCA.req -subj /CN=test2.com -sha256 -nodes
```

## 8. 生成证书和密钥，并配置

```sh
openssl x509 -req -CA rootCA.cer -CAkey rootCA.pvk -in ServerCA.req -out ServerCA.cer -days 365 -extfile ServerCA.ext -sha256 -set_serial 0x1111
```

-  `openssl x509`

这是 OpenSSL 用于处理 X.509 证书的命令。X.509 是最常用的数字证书格式，它定义了证书的结构。通过此命令，可以签发证书、查看证书信息等。

-  `-req`

这个选项表示该命令是用来处理一个证书请求（CSR）。通过 CSR 文件（`ServerCA.req`）来签发证书。

- `-CA rootCA.cer`

这个选项指定签发证书的根证书（CA 证书）。在这里，`rootCA.cer` 是用于签署请求的根证书文件。这个根证书将被用来验证所签署的证书。

- `-CAkey rootCA.pvk`

这个选项指定根证书对应的私钥文件。`rootCA.pvk` 是用于签署证书请求的私钥文件。私钥用于生成签名，证明证书是由相应的根证书所有者签发的。

- `-in ServerCA.req`

该选项指定输入文件，即证书签名请求（CSR）。在这里，`ServerCA.req` 是先前生成的证书请求文件。它包含了申请证书的主体信息（如公钥、域名等）。

- `-out ServerCA.cer`

该选项指定输出文件，即签发的证书。在这里，生成的证书将保存为 `ServerCA.cer`，通常是 PEM 格式的 X.509 证书。

- `-days 365`

这个选项指定新生成证书的有效期为 365 天。证书在到期后需要重新签发或续期。

- `-extfile ServerCA.ext`

该选项指定一个外部配置文件，用于添加扩展（extensions）到生成的证书中。扩展文件（通常是 `.ext` 文件）包含额外的证书信息，如主题备用名称（SAN）、密钥用途、CRL 分发点等。这里的 `ServerCA.ext` 是包含扩展配置的文件。

- `-sha256`

这个选项指定使用 SHA-256 哈希算法对证书进行签名。SHA-256 是目前安全性较高的哈希算法，用于生成签名。

- `-set_serial 0x1111`

这个选项为生成的证书设置一个唯一的序列号。证书的序列号必须是唯一的，且不重复。在这里，序列号被设置为 `0x1111`，即十六进制的 `1111`。序列号通常是一个大整数，确保每个证书的唯一性。

## 总结

这条命令的作用是：

- 使用根证书 `rootCA.cer` 和根证书私钥 `rootCA.pvk` 来签署一个证书请求（CSR），该请求文件为 `ServerCA.req`；
- 签发的证书将被保存为 `ServerCA.cer`，并设置为有效期 365 天；
- 使用 SHA-256 哈希算法对证书进行签名；
- 证书的序列号被设置为 `0x1111`；
- 使用 `ServerCA.ext` 文件来指定证书的扩展信息。

最终生成的 `ServerCA.cer` 就是签名的证书，通常它将包含在服务器上，或者提供给需要验证其身份的客户端使用。