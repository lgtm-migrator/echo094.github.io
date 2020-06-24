---
layout: post
title:  "Ubuntu配置HTTPS服务器"
date:   2020-06-24 00:00:00 +0000
categories: linux apache2 ssl
---



系统： Ubuntu 20.04



以前直接用IIS搭了个服务器，共享一些PT站下载的资源，能够满足日常需求。但随着文件的增加，不得不对文件进行组织。为了不破坏原有的文件结构，发现可以使用`mklink`的硬链接功能。但体验以后，还是不如linux的软链接来得方便。

另外，之前SSL证书通过`www.sslforfree.com`生成，但现在它换了供应商后，通配符证书要收费了，于是改用acme工具。

既然上述需求都需要linux环境，就干脆将整个服务都迁移过去了。以后有了别的什么问题，网上的资料还多些。



## 申请SSL证书

操作还是很简单的。

1. 安装工具：

    ```bash
    curl  https://get.acme.sh | sh
    ```

2. 去域名提供商申请API，这一步就跳过了。

3. 执行脚本自动申请证书：

    操作方法见wiki：[How to use DNS API](https://github.com/acmesh-official/acme.sh/wiki/dnsapi)，这里的域名服务商是阿里云
    
    先将需要的`api_key`和`api_secret`放到环境变量中
    
    ```bash
    export Ali_Key=""
    export Ali_Secret=""
    ```
    
    然后执行脚本，结束后如果没有通配符证书，删掉`-d example.com`再执行一遍
    
    ```bash
    acme.sh --issue --dns dns_ali -d example.com -d *.example.com
    ```

这样证书就保存到`~/.acme/.sh/*.example.com/`目录下了。

如果要将证书转成`pfx`格式给IIS使用，需要先参照`*.example.com.cer`把`fullchain.cer`中自己那个证书删除，再执行转换指令，否则生成的文件导入IIS后无法导出，然后在绑定域名时出错。

转化指令如下：

```bash
openssl pkcs12 -export -out "export.pfx" -inkey "*.example.com.key" -in "*.example.com.cer" -certfile fullchain.cer
```



## 配置apache2服务器

**basic**

首先安装apache2：

```bash
sudo apt-get install apache2
```

然后进入目录`/etc/apache2/`修改配置文件。

将默认SSL站点配置文件软链接到目录`sites-enabled`中，表示启动该配置文件：

```bash
ln -s /etc/apache2/sites-available/default-ssl.conf /etc/apache2/sites-enabled/000-default-ssl.conf
```

修改配置文件`default-ssl.conf`，指向我们申请的证书，这里就使用绝对路径了：

```sh
SSLCertificateFile	/home/${user}/.acme.sh/*.example.com/*.example.com.cer
SSLCertificateKeyFile /home/${user}/.acme.sh/*.example.com/*.example.com.key
```

然后开启服务：

```bash
sudo a2enmod ssl
sudo /etc/init.d/apache2 restart
```

如果没有报错，网页应该能正常显示了。

如果没有修改网站根目录，把网页放到`/var/www/html`即可。

**extra**

为了提高安全性，可以有选择性地关闭目录浏览功能，比如关闭根目录的浏览。

在配置文件`apache2.conf`中找到`Directory /var/www/`项，修改其`Options`：

```sh
Options -Indexes +FollowSymLinks
```

也可以添加虚拟路径，在相应的站点配置文件中添加目录：

```sh
# 添加虚拟路径
Alias /virtual-path "/real-path/"
# 配置对应实际目录的权限
<Directory "/real-path/">
	Options +Indexes +FollowSymlinks
	AllowOverride All
	Require all granted
</Directory>
```

