# 1. Zerotier

优点：使用最简单，能建立P2P的情况下延迟低，速度快
缺点：实测能建立P2P的情况很少，中转模式下以来国外的Planet根服务器，延迟极高速度极慢不稳定

总结之前的经验来看，只有少数情况下zt可以建立点对点，基本上还是靠中转。自建moon中转效果不明显，可能是需要换个docker镜像。根本解决办法是自建planet。

> 使用docker自建planet教程
> https://blog.csdn.net/u012153104/article/details/131829837?type=blog&rId=131829837&refer=APP&source=koude11

> github仓库https://github.com/xubiaolin/docker-zerotier-planet

**不能解析地址，ip变动需要全部重新配置**
**一键部署只支持Linux，windows下要自己想办法**

这是一开始采用的命令
```powershel
docker run -d -v zerotier-app:/app -v zerotier-one:/var/lib/zerotier-one -p 9994:9994 -p 9994:9994/udp -p 3443:3443 --name zerotier-planet --restart unless-stopped xubiaolin/zerotier-planet:latest
```

这样配置有bug：
管理页面可以登陆但是进入网络配置就会报错
Error retrieving list of networks on this controller: RequestError: connect ECONNREFUSED 127.0.0.1:9994
![image](https://github.com/user-attachments/assets/59d6c47f-0915-4f4b-b1ec-e2458cd9bfc1)

这是后来从deploy.sh中截取的命令，猜测对应替换掉其中的地址可以避免上述报错，但是由于这个镜像不能解析域名，整个项目都是为这个一键部署的脚本适配，自己通过docker命令，发现作者埋了一堆坑，根本发挥不出docker作用，属实过于飞舞。域名解析有多个issue提议，但是甚至不在开发计划中，疑似作者想通过卖他自己的服务赚米，不想尝试了

```powershell
docker run -d \
        --name ${CONTAINER_NAME} \
        -p ${ZT_PORT}:${ZT_PORT} \
        -p ${ZT_PORT}:${ZT_PORT}/udp \
        -p ${API_PORT}:${API_PORT} \
        -p ${FILE_PORT}:${FILE_PORT} \
        -e IP_ADDR4=${ipv4} \
        -e IP_ADDR6=${ipv6} \
        -e ZT_PORT=${ZT_PORT} \
        -e API_PORT=${API_PORT} \
        -e FILE_SERVER_PORT=${FILE_PORT} \
        -v ${DIST_PATH}:/app/dist \
        -v ${ZTNCUI_PATH}:/app/ztncui \
        -v ${ZEROTIER_PATH}/one:/var/lib/zerotier-one \
        -v ${CONFIG_PATH}:/app/config \
        --restart unless-stopped \
        ${DOCKER_IMAGE}
```

# 2. OpenVPN

优点：全部流量都走转发，在国内有公网服务器的情况下非常稳定，且支持域名解析
缺点：延迟比P2P大，带宽收到服务器的限制，服务端配置麻烦，客户端需要证书授权麻烦

> Windows Server下搭建：
> https://www.cnblogs.com/simonlee-hello/articles/15830715.html#:~:text=%E6%AD%A5%E9%AA%A4%E4%B8%80%EF%BC%9A%E5%AE%89%E8%A3%85Open

原网站防丢存档：

<html>
<body>
<!--StartFragment--><h2 style="margin: 0px 0px 4px; font-size: 20px; color: rgb(0, 0, 0); font-family: &quot;PingFang SC&quot;, &quot;Microsoft YaHei&quot;, &quot;Helvetica Neue&quot;, Helvetica, Arial, sans-serif; font-style: normal; font-variant-ligatures: normal; font-variant-caps: normal; letter-spacing: normal; orphans: 2; text-align: start; text-indent: 0px; text-transform: none; widows: 2; word-spacing: 0px; -webkit-text-stroke-width: 0px; white-space: normal; background-color: rgb(255, 255, 255); text-decoration-thickness: initial; text-decoration-style: initial; text-decoration-color: initial;"><a id="cb_post_title_url" class="postTitle2 vertical-middle" href="https://www.cnblogs.com/simonlee-hello/articles/15830715.html" title="发布于 2022-01-21 16:20" style="color: rgb(34, 51, 85); text-decoration: none;"><span role="heading" aria-level="2" style="vertical-align: middle;">Windows server 2012、2016、2019上搭建openV（亲测成功）</span></a></h2><div class="postbody" style="color: rgb(0, 0, 0); font-family: &quot;PingFang SC&quot;, &quot;Microsoft YaHei&quot;, &quot;Helvetica Neue&quot;, Helvetica, Arial, sans-serif; font-size: 14px; font-style: normal; font-variant-ligatures: normal; font-variant-caps: normal; font-weight: 400; letter-spacing: normal; orphans: 2; text-align: start; text-indent: 0px; text-transform: none; widows: 2; word-spacing: 0px; -webkit-text-stroke-width: 0px; white-space: normal; background-color: rgb(255, 255, 255); text-decoration-thickness: initial; text-decoration-style: initial; text-decoration-color: initial;"><div id="cnblogs_post_body" class="blogpost-body cnblogs-markdown" style="margin-bottom: 20px; word-break: break-word;"><h3 id="blogTitle0" style="font-size: 18px; border-bottom: 1px solid rgb(187, 187, 187); font-weight: bold; line-height: 1.5; margin: 10px 10px 10px 0px; padding-left: 0px; padding-bottom: 5px;">步骤一：安装OpenVPN服务</h3><p style="margin: 10px auto; text-indent: 0px;">下载最新版本的OpenVPN服务端，可以从这个地址下载：</p><p style="margin: 10px auto; text-indent: 0px;"><a href="https://openvpn.net/community-downloads/" target="_blank" rel="noopener nofollow" style="color: rgb(29, 88, 209); text-decoration: none;">Community Downloads | OpenVPN</a></p><p style="margin: 10px auto; text-indent: 0px;">下载的文件名为：OpenVPN-2.5.5-I602-amd64.msi</p><p style="margin: 10px auto; text-indent: 0px;">双击安装，选择“Customize”</p><p style="margin: 10px auto; text-indent: 0px;"><img src="https://photobeds.oss-cn-shanghai.aliyuncs.com/uPic/%E6%88%AA%E5%B1%8F2022-01-20%20%E4%B8%8B%E5%8D%885.41.16.png" alt="截屏2022-01-20 下午5.41.16" loading="lazy" class="medium-zoom-image" style="border: 0px; max-width: 100%; height: auto; cursor: zoom-in; transition: transform 300ms cubic-bezier(0.2, 0, 0.2, 1) !important;"></p><p style="margin: 10px auto; text-indent: 0px;">默认情况下不会安装的两个特性，我们需要在安装过程中进行选择。</p><ul style="margin-left: 30px; padding-left: 0px; list-style-type: disc;"><li style="list-style: inherit;">OpenVPN Service</li><li style="list-style: inherit;">OpenSSL Utilities</li></ul><p style="margin: 10px auto; text-indent: 0px;"><img src="https://photobeds.oss-cn-shanghai.aliyuncs.com/uPic/%E6%88%AA%E5%B1%8F2022-01-20%20%E4%B8%8B%E5%8D%885.44.20.png" alt="截屏2022-01-20 下午5.44.20" loading="lazy" class="medium-zoom-image" style="border: 0px; max-width: 100%; height: auto; cursor: zoom-in; transition: transform 300ms cubic-bezier(0.2, 0, 0.2, 1) !important;"></p><p style="margin: 10px auto; text-indent: 0px;"><img src="https://photobeds.oss-cn-shanghai.aliyuncs.com/uPic/image-20220120174658478.png" alt="image-20220120174658478" loading="lazy" class="medium-zoom-image" style="border: 0px; max-width: 100%; height: auto; cursor: zoom-in; transition: transform 300ms cubic-bezier(0.2, 0, 0.2, 1) !important;"></p><p style="margin: 10px auto; text-indent: 0px;">点击install now</p><p style="margin: 10px auto; text-indent: 0px;">安装完成后，点击close。</p><p style="margin: 10px auto; text-indent: 0px;">这时会弹出一个提醒框，提示没有找到可连接的配置文件，现在不用管它，点击ok即可。</p><p style="margin: 10px auto; text-indent: 0px;">我们打开网络和共享中心，点击更改适配器设置，可以看到多了两个网络连接</p><p style="margin: 10px auto; text-indent: 0px;"><img src="https://photobeds.oss-cn-shanghai.aliyuncs.com/uPic/image-20220120175236107.png" alt="image-20220120175236107" loading="lazy" class="medium-zoom-image" style="border: 0px; max-width: 100%; height: auto; cursor: zoom-in; transition: transform 300ms cubic-bezier(0.2, 0, 0.2, 1) !important;"></p><h3 id="blogTitle1" style="font-size: 18px; border-bottom: 1px solid rgb(187, 187, 187); font-weight: bold; line-height: 1.5; margin: 10px 10px 10px 0px; padding-left: 0px; padding-bottom: 5px;">步骤二：设置CA证书、生成服务端和客户端的证书和私钥等</h3><p style="margin: 10px auto; text-indent: 0px;">找到目录“C:\Program Files\OpenVPN\easy-rsa”，将文件vars.example复制一份改名为vars，“vars”文件包含内置的Easy-RSA配置设置。可以保持默认设置，也可以自定义更改。</p><div class="table-wrapper" style="display: block; overflow-x: auto;">
属性 | 默认值 | 作用
-- | -- | --
set_var EASYRSA | C:\Program Files\OpenVPN\easy-rsa | Defines the folder location of easy-rsa scripts
set_var EASYRSA_OPENSSL | C:\Program Files\OpenVPN\bin\openssl.exe | Defines the OpenSSL binary path
set_var EASYRSA_PKI | C:\Program Files\OpenVPN\easy-rsa\pki | The folder location of SSL/TLS file exists after creation
set_var EASYRSA_DN | cn_only | This is used to adjust what elements are included in the Subject field as the DN
set_var EASYRSA_REQ_COUNTRY | “US” | Our Organisation Country
set_var EASYRSA_REQ_PROVINCE | “California” | Our Organisation Province
set_var EASYRSA_REQ_CITY | “San Francisco” | Our Organisation City
set_var EASYRSA_REQ_ORG | “Copyleft Certificate Co” | Our Organisation Name
set_var EASYRSA_REQ_EMAIL | “me@example.net” | Our Organisation contact email
set_var EASYRSA_REQ_OU | “My Organizational Unit” | Our Organisation Unit name
set_var EASYRSA_KEY_SIZE | 2048 | Define the key pair size in bits
set_var EASYRSA_ALGO | rsa | The default crypt mode
set_var EASYRSA_CA_EXPIRE | 3650 | The CA key expire days
set_var EASYRSA_CERT_EXPIRE | 825 | The Server certificate key expire days
set_var EASYRSA_NS_SUPPORT | “no” | Support deprecated Netscape extension
set_var EASYRSA_NS_COMMENT | “HAKASE-LABS CERTIFICATE AUTHORITY” | Defines NS comment
set_var EASYRSA_EXT_DIR | "$EASYRSA/x509-types" | Defines the x509 extension directory
set_var EASYRSA_SSL_CONF | "$EASYRSA/openssl-easyrsa.cnf" | Defines the openssl config file location
set_var EASYRSA_DIGEST | "sha256" | Defines the cryptographic digest to use

</div><h3 id="blogTitle2" style="font-size: 18px; border-bottom: 1px solid rgb(187, 187, 187); font-weight: bold; line-height: 1.5; margin: 10px 10px 10px 0px; padding-left: 0px; padding-bottom: 5px;">步骤三：配置ip转发和网络共享</h3><p style="margin: 10px auto; text-indent: 0px;">打开注册表，win+R，输入regedit.exe，依次找到：<strong>HKEY LOCAL MACHINESYSTEM\CurrentControlSet\Services\Tcpip\Parameters</strong>将IPEableRouter值改为1，如下图</p><p style="margin: 10px auto; text-indent: 0px;"><img src="https://photobeds.oss-cn-shanghai.aliyuncs.com/uPic/image-20220120184135935.png" alt="image-20220120184135935" loading="lazy" class="medium-zoom-image" style="border: 0px; max-width: 100%; height: auto; cursor: zoom-in; transition: transform 300ms cubic-bezier(0.2, 0, 0.2, 1) !important;"></p><p style="margin: 10px auto; text-indent: 0px;">然后打开控制面板，找到“控制面板\网络和 Internet\网络连接”，右键点击“以太网”，点击“属性”，在“共享”中钩上“允许其他网络用户通过此计算机的internet连接来连接”，并选择“OpenVPN TAP-Windows6”，点击确定。</p><p style="margin: 10px auto; text-indent: 0px;"><img src="https://photobeds.oss-cn-shanghai.aliyuncs.com/uPic/image-20220120184424489.png" alt="image-20220120184424489" loading="lazy" class="medium-zoom-image" style="border: 0px; max-width: 100%; height: auto; cursor: zoom-in; transition: transform 300ms cubic-bezier(0.2, 0, 0.2, 1) !important;"></p><p style="margin: 10px auto; text-indent: 0px;"><img src="https://photobeds.oss-cn-shanghai.aliyuncs.com/uPic/image-20220120184527697.png" alt="image-20220120184527697" loading="lazy" class="medium-zoom-image" style="border: 0px; max-width: 100%; height: auto; cursor: zoom-in; transition: transform 300ms cubic-bezier(0.2, 0, 0.2, 1) !important;"></p><h3 id="blogTitle3" style="font-size: 18px; border-bottom: 1px solid rgb(187, 187, 187); font-weight: bold; line-height: 1.5; margin: 10px 10px 10px 0px; padding-left: 0px; padding-bottom: 5px;">步骤四：创建服务端配置文件</h3><p style="margin: 10px auto; text-indent: 0px;">首先打开Windows资源管理器，进入C:\Program Files\OpenVPN\sample-config文件夹，将server.ovpn文件复制一份到C:\Program Files\OpenVPN\config目录下。</p><p style="margin: 10px auto; text-indent: 0px;">同时将以下文件一并复制到C:\Program Files\OpenVPN\config目录下</p><ul style="margin-left: 30px; padding-left: 0px; list-style-type: disc;"><li style="list-style: inherit;">ca.crt</li><li style="list-style: inherit;">dh.pem</li><li style="list-style: inherit;">SERVER.crt</li><li style="list-style: inherit;">SERVER.key</li><li style="list-style: inherit;">tls-auth.key</li></ul><p style="margin: 10px auto; text-indent: 0px;"><img src="https://photobeds.oss-cn-shanghai.aliyuncs.com/uPic/image-20220120185305290.png" alt="image-20220120185305290" loading="lazy" class="medium-zoom-image" style="border: 0px; max-width: 100%; height: auto; cursor: zoom-in; transition: transform 300ms cubic-bezier(0.2, 0, 0.2, 1) !important;"></p><p style="margin: 10px auto; text-indent: 0px;">编辑server.ovpn文件，修改以下地方，其他保持默认即可</p><pre class="highlighter-hljs" highlighted="true" style="transition-duration: 0.2s; transition-property: background, font-size, border-color, border-radius, border-width, padding, margin, color; overflow: auto; margin: 10px auto; font-size: 1em;"><code class="highlighter-hljs hljs language-perl" style="font-family: &quot;Courier New&quot;, sans-serif; transition-duration: 0.2s; transition-property: background, font-size, border-color, border-radius, border-width, padding, margin, color; background: rgb(245, 245, 245); color: rgb(68, 68, 68); padding: 1em; display: block; font-size: 12px; border: 1px solid rgb(204, 204, 204); border-radius: 3px; overflow-x: auto; line-height: 1.5; margin: 0px;"><span class="hljs-keyword" style="color: rgb(0, 0, 255);">local</span> x.x.x.x  <span class="hljs-comment" style="color: rgb(0, 128, 0);"># 这个是服务器的内网IP地址，或者保持默认</span>
ca ca.crt
cert SERVER.crt
key SERVER.key
dh dh.pem
<span class="hljs-keyword" style="color: rgb(0, 0, 255);">push</span> <span class="hljs-string" style="color: rgb(163, 21, 21);">"redirect-gateway def1 bypass-dhcp"</span>
<span class="hljs-keyword" style="color: rgb(0, 0, 255);">push</span> <span class="hljs-string" style="color: rgb(163, 21, 21);">"dhcp-option DNS 114.114.114.114"</span> <span class="hljs-comment" style="color: rgb(0, 128, 0);"># 可以改成其他的DNS服务器</span>
<span class="hljs-keyword" style="color: rgb(0, 0, 255);">push</span> <span class="hljs-string" style="color: rgb(163, 21, 21);">"dhcp-option DNS 8.8.8.8"</span>
tls-auth tls-auth.key <span class="hljs-number" style="color: rgb(136, 0, 0);">0</span> <span class="hljs-comment" style="color: rgb(0, 128, 0);"># This file is secret</span>
cipher AES-<span class="hljs-number" style="color: rgb(136, 0, 0);">256</span>-GCM
</code></pre><p style="margin: 10px auto; text-indent: 0px;">点击连接，图标变绿就正常。</p><p style="margin: 10px auto; text-indent: 0px;"><img src="https://photobeds.oss-cn-shanghai.aliyuncs.com/uPic/image-20220120190423076.png" alt="image-20220120190423076" loading="lazy" style="border: 0px; max-width: 100%; height: auto;"></p><h3 id="blogTitle4" style="font-size: 18px; border-bottom: 1px solid rgb(187, 187, 187); font-weight: bold; line-height: 1.5; margin: 10px 10px 10px 0px; padding-left: 0px; padding-bottom: 5px;">配置客户端</h3><p style="margin: 10px auto; text-indent: 0px;">复制以下文件到你的客户端，并且在同一目录下</p><ul style="margin-left: 30px; padding-left: 0px; list-style-type: disc;"><li style="list-style: inherit;">ca.crt</li><li style="list-style: inherit;">CLIENT.crt</li><li style="list-style: inherit;">CLIENT.key</li><li style="list-style: inherit;">tls-auth.key</li><li style="list-style: inherit;">client.ovpn（“C:\Program Files\OpenVPN\sample-config”）</li></ul><p style="margin: 10px auto; text-indent: 0px;">编辑client.ovpn，修改以下参数，其他保持默认</p><pre class="highlighter-hljs" highlighted="true" style="transition-duration: 0.2s; transition-property: background, font-size, border-color, border-radius, border-width, padding, margin, color; overflow: auto; margin: 10px auto; font-size: 1em;"><code class="highlighter-hljs hljs language-vbnet" style="font-family: &quot;Courier New&quot;, sans-serif; transition-duration: 0.2s; transition-property: background, font-size, border-color, border-radius, border-width, padding, margin, color; background: rgb(245, 245, 245); color: rgb(68, 68, 68); padding: 1em; display: block; font-size: 12px; border: 1px solid rgb(204, 204, 204); border-radius: 3px; overflow-x: auto; line-height: 1.5; margin: 0px;">remote *.*.*.* <span class="hljs-number" style="color: rgb(136, 0, 0);">1194</span> # 服务器公网IP地址
ca ca.crt
cert CLIENT.crt
<span class="hljs-keyword" style="color: rgb(0, 0, 255);">key</span> CLIENT.<span class="hljs-keyword" style="color: rgb(0, 0, 255);">key</span>
tls-auth tls-auth.<span class="hljs-keyword" style="color: rgb(0, 0, 255);">key</span> <span class="hljs-number" style="color: rgb(136, 0, 0);">1</span>
cipher AES-<span class="hljs-number" style="color: rgb(136, 0, 0);">256</span>-GCM
</code></pre><p style="margin: 10px auto; text-indent: 0px;">下载对应系统的客户端</p><p style="margin: 10px auto; text-indent: 0px;"><a href="https://openvpn.net/vpn-client/" target="_blank" rel="noopener nofollow" style="color: rgb(29, 88, 209); text-decoration: none;">OpenVPN Connect Client | Our Official VPN Client | OpenVPN</a></p><p style="margin: 10px auto; text-indent: 0px;">如windows系统，将client.ovpn文件拖入客户端界面即可，如下</p><p style="margin: 10px auto; text-indent: 0px;"><img src="https://photobeds.oss-cn-shanghai.aliyuncs.com/uPic/image-20220120191352128.png" alt="image-20220120191352128" loading="lazy" class="medium-zoom-image" style="border: 0px; max-width: 100%; height: auto; cursor: zoom-in; transition: transform 300ms cubic-bezier(0.2, 0, 0.2, 1) !important;"></p><p style="margin: 10px auto; text-indent: 0px;">随后点击连接，出现如下界面则表示连接成功</p><p style="margin: 10px auto; text-indent: 0px;"><img src="https://photobeds.oss-cn-shanghai.aliyuncs.com/uPic/image-20220120191650721.png" alt="image-20220120191650721" loading="lazy" class="medium-zoom-image" style="border: 0px; max-width: 100%; height: auto; cursor: zoom-in; transition: transform 300ms cubic-bezier(0.2, 0, 0.2, 1) !important;"></p><p style="margin: 10px auto; text-indent: 0px;">这时ping一下10.8.0.1测试是否连通，ping下百度测试DNS是否正常。</p><p style="margin: 10px auto; text-indent: 0px;"><img src="https://photobeds.oss-cn-shanghai.aliyuncs.com/uPic/image-20220120191756447.png" alt="image-20220120191756447" loading="lazy" class="medium-zoom-image" style="border: 0px; max-width: 100%; height: auto; cursor: zoom-in; transition: transform 300ms cubic-bezier(0.2, 0, 0.2, 1) !important;"></p><p style="margin: 10px auto; text-indent: 0px;">这就成功了。</p><h3 id="blogTitle5" style="font-size: 18px; border-bottom: 1px solid rgb(187, 187, 187); font-weight: bold; line-height: 1.5; margin: 10px 10px 10px 0px; padding-left: 0px; padding-bottom: 5px;">吊销客户端证书</h3><p style="margin: 10px auto; text-indent: 0px;">当我们创建了多个用户使用，然后某些原因，个别用户需要禁用的时候，我们就可以使用吊销证书的方式来处理。</p><p style="margin: 10px auto; text-indent: 0px;">首先在你的OpenVPN服务器上，打开cmd管理员权限窗口，进入目录“C:\Program Files\OpenVPN\easy-rsa\”</p><pre class="highlighter-hljs" highlighted="true" style="transition-duration: 0.2s; transition-property: background, font-size, border-color, border-radius, border-width, padding, margin, color; overflow: auto; margin: 10px auto; font-size: 1em;"><code class="highlighter-hljs hljs language-bash" style="font-family: &quot;Courier New&quot;, sans-serif; transition-duration: 0.2s; transition-property: background, font-size, border-color, border-radius, border-width, padding, margin, color; background: rgb(245, 245, 245); color: rgb(68, 68, 68); padding: 1em; display: block; font-size: 12px; border: 1px solid rgb(204, 204, 204); border-radius: 3px; overflow-x: auto; line-height: 1.5; margin: 0px;"><span class="hljs-built_in" style="color: rgb(0, 0, 255);">cd</span> <span class="hljs-string" style="color: rgb(163, 21, 21);">"C:\Program Files\OpenVPN\easy-rsa"</span>
EasyRSA-Start.bat
</code></pre><p style="margin: 10px auto; text-indent: 0px;">进入easyrsa的shell界面</p><p style="margin: 10px auto; text-indent: 0px;"><img src="https://photobeds.oss-cn-shanghai.aliyuncs.com/uPic/image-20220120192518643.png" alt="image-20220120192518643" loading="lazy" class="medium-zoom-image" style="border: 0px; max-width: 100%; height: auto; cursor: zoom-in; transition: transform 300ms cubic-bezier(0.2, 0, 0.2, 1) !important;"></p><p style="margin: 10px auto; text-indent: 0px;">输入以下命令</p><pre class="highlighter-hljs" highlighted="true" style="transition-duration: 0.2s; transition-property: background, font-size, border-color, border-radius, border-width, padding, margin, color; overflow: auto; margin: 10px auto; font-size: 1em;"><code class="highlighter-hljs hljs language-bash" style="font-family: &quot;Courier New&quot;, sans-serif; transition-duration: 0.2s; transition-property: background, font-size, border-color, border-radius, border-width, padding, margin, color; background: rgb(245, 245, 245); color: rgb(68, 68, 68); padding: 1em; display: block; font-size: 12px; border: 1px solid rgb(204, 204, 204); border-radius: 3px; overflow-x: auto; line-height: 1.5; margin: 0px;">./easyrsa revoke &lt;client&gt;  <span class="hljs-comment" style="color: rgb(0, 128, 0);"># 你要吊销的客户端名</span>
./easyrsa gen-crl <span class="hljs-comment" style="color: rgb(0, 128, 0);"># 生成crl.pem文件，用来记录吊销的证书</span>
</code></pre><p style="margin: 10px auto; text-indent: 0px;"><img src="https://photobeds.oss-cn-shanghai.aliyuncs.com/uPic/image-20220120192845049.png" alt="image-20220120192845049" loading="lazy" class="medium-zoom-image" style="border: 0px; max-width: 100%; height: auto; cursor: zoom-in; transition: transform 300ms cubic-bezier(0.2, 0, 0.2, 1) !important;"></p><p style="margin: 10px auto; text-indent: 0px;">编辑server.ovpn（“C:\Program Files\OpenVPN\config”下），在行尾加入一行</p><pre class="highlighter-hljs" highlighted="true" has-selection="true" style="transition-duration: 0.2s; transition-property: background, font-size, border-color, border-radius, border-width, padding, margin, color; overflow: auto; margin: 10px auto; font-size: 1em;"><code class="highlighter-hljs hljs language-swift" style="font-family: &quot;Courier New&quot;, sans-serif; transition-duration: 0.2s; transition-property: background, font-size, border-color, border-radius, border-width, padding, margin, color; background: rgb(245, 245, 245); color: rgb(68, 68, 68); padding: 1em; display: block; font-size: 12px; border: 1px solid rgb(204, 204, 204); border-radius: 3px; overflow-x: auto; line-height: 1.5; margin: 0px;">crl<span class="hljs-operator" style="color: rgb(171, 86, 86);">-</span>verify <span class="hljs-string" style="color: rgb(163, 21, 21);">"C:<span class="hljs-subst">\\</span>Program Files<span class="hljs-subst">\\</span>OpenVPN<span class="hljs-subst">\\</span>easy-rsa<span class="hljs-subst">\\</span>pki<span class="hljs-subst">\\</span>crl.pem"</span> # 用来告知服务端有哪些证书是被吊销的
</code></pre><p style="margin: 10px auto; text-indent: 0px;">这一步如果不做则不会生效，添加了这行之后，服务端重连一下即可。</p><p style="margin: 10px auto; text-indent: 0px;">参考：</p><p style="margin: 10px auto; text-indent: 0px;"><a href="https://supporthost.in/how-to-setup-openvpn-on-windows-server-2019/" target="_blank" rel="noopener nofollow" style="color: rgb(29, 88, 209); text-decoration: none;">https://supporthost.in/how-to-setup-openvpn-on-windows-server-2019/</a></p><p style="margin: 10px auto; text-indent: 0px;"><a href="https://www.shangmayuan.com/a/28de9b94188547acb8a48c10.html" target="_blank" rel="noopener nofollow" style="color: rgb(29, 88, 209); text-decoration: none;">https://www.shangmayuan.com/a/28de9b94188547acb8a48c10.html</a></p></div></div><!--EndFragment-->
</body>
</html>[Windows server 2012、2016、2019上搭建openV（亲测成功）](https://www.cnblogs.com/simonlee-hello/articles/15830715.html)
步骤一：安装OpenVPN服务
下载最新版本的OpenVPN服务端，可以从这个地址下载：

[Community Downloads | OpenVPN](https://openvpn.net/community-downloads/)

下载的文件名为：OpenVPN-2.5.5-I602-amd64.msi

双击安装，选择“Customize”

截屏2022-01-20 下午5.41.16

默认情况下不会安装的两个特性，我们需要在安装过程中进行选择。

OpenVPN Service
OpenSSL Utilities
截屏2022-01-20 下午5.44.20

image-20220120174658478

点击install now

安装完成后，点击close。

这时会弹出一个提醒框，提示没有找到可连接的配置文件，现在不用管它，点击ok即可。

我们打开网络和共享中心，点击更改适配器设置，可以看到多了两个网络连接

image-20220120175236107

步骤二：设置CA证书、生成服务端和客户端的证书和私钥等
找到目录“C:\Program Files\OpenVPN\easy-rsa”，将文件vars.example复制一份改名为vars，“vars”文件包含内置的Easy-RSA配置设置。可以保持默认设置，也可以自定义更改。

属性	默认值	作用
set_var EASYRSA	C:\Program Files\OpenVPN\easy-rsa	Defines the folder location of easy-rsa scripts
set_var EASYRSA_OPENSSL	C:\Program Files\OpenVPN\bin\openssl.exe	Defines the OpenSSL binary path
set_var EASYRSA_PKI	C:\Program Files\OpenVPN\easy-rsa\pki	The folder location of SSL/TLS file exists after creation
set_var EASYRSA_DN	cn_only	This is used to adjust what elements are included in the Subject field as the DN
set_var EASYRSA_REQ_COUNTRY	“US”	Our Organisation Country
set_var EASYRSA_REQ_PROVINCE	“California”	Our Organisation Province
set_var EASYRSA_REQ_CITY	“San Francisco”	Our Organisation City
set_var EASYRSA_REQ_ORG	“Copyleft Certificate Co”	Our Organisation Name
set_var EASYRSA_REQ_EMAIL	“me@example.net”	Our Organisation contact email
set_var EASYRSA_REQ_OU	“My Organizational Unit”	Our Organisation Unit name
set_var EASYRSA_KEY_SIZE	2048	Define the key pair size in bits
set_var EASYRSA_ALGO	rsa	The default crypt mode
set_var EASYRSA_CA_EXPIRE	3650	The CA key expire days
set_var EASYRSA_CERT_EXPIRE	825	The Server certificate key expire days
set_var EASYRSA_NS_SUPPORT	“no”	Support deprecated Netscape extension
set_var EASYRSA_NS_COMMENT	“HAKASE-LABS CERTIFICATE AUTHORITY”	Defines NS comment
set_var EASYRSA_EXT_DIR	"$EASYRSA/x509-types"	Defines the x509 extension directory
set_var EASYRSA_SSL_CONF	"$EASYRSA/openssl-easyrsa.cnf"	Defines the openssl config file location
set_var EASYRSA_DIGEST	"sha256"	Defines the cryptographic digest to use
如没有特殊要求，则vars文件保持默认即可。

现在打开cmd（管理员权限），切换到“C:\Program Files\OpenVPN\easy-rsa”目录下

cd C:\Program Files\OpenVPN\easy-rsa
EasyRSA-Start.bat
输入EasyRSA-Start.bat回车后，我们会进入到easy-rsa3的shell会话

image-20220120180519240

执行init-pki来创建pki目录

./easyrsa init-pki
image-20220120180713529

现在，使用下面的命令构建证书颁发机构(CA)密钥。这个CA根证书文件稍后将用于签署其他证书和密钥。我们使用的“nopass”选项用于禁用密码。

./easyrsa build-ca nopass
命令将被要求输入通用名称。这里我输入的VPN服务器主机名是OPENVPNSERVER，这是一种常见的做法。在这里，我们可以自由使用任何名称或值。同时创建的CA证书将被保存到文件夹“C:\Program Files\OpenVPN\easy-rsa\pki”，文件名为“ca .crt”。请参考下面的截图。

image-20220120181013212

现在使用下面的命令构建一个服务器证书和密钥。这里将< SERVER >替换为您自己的服务器名。我还使用Option nopass来禁用密码。

./easyrsa build-server-full <SERVER> nopass
颁发的服务器证书将在“C:\Program Files\OpenVPN\easy-rsa\pki\issued”文件夹中，文件名为SERVER .crt。

image-20220120181413354

这里可以使用以下命令进行验证，返回ok就没问题

openssl verify -CAfile pki/ca.crt pki/issued/SERVER.crt
现在，使用下面的命令构建客户端证书和密钥。将< CLIENT >替换为您的客户端名称。也使用选项nopass来禁用密码。

./easyrsa build-client-full <CLIENT> nopass
颁发的客户端证书也会被保存到“C:\Program Files\OpenVPN\easy-rsa\pki\issued”文件夹中，文件名为“CLIENT.crt”。

image-20220120181729642

同样这里可以使用以下命令进行验证，返回ok就没问题

openssl verify -CAfile pki/ca.crt pki/issued/CLIENT.crt
到这里就完成了CA证书，服务器和客户端证书的生成和密钥。这些密钥将用于OpenVPN服务器和客户端之间的身份验证。

现在生成一个用于标准RSA证书/密钥之外的共享密钥。文件名为tls-auth.key。

使用这个密钥，我们启用TLS -auth指令，它添加一个额外的HMAC签名到所有SSL/TLS握手包的完整性验证。任何不带有正确HMAC签名的UDP包可以被丢弃而无需进一步处理。

启用tls-auth可以保护我们免受：

OpenVPN UDP端口上的DoS攻击或端口泛洪。
端口扫描，以确定哪些服务器UDP端口处于侦听状态。
SSL/TLS实现中的缓冲区溢出漏洞。
从未经授权的机器发起SSL/TLS握手。
首先使用GitHub链接https://github.com/TinCanTech/easy-tls下载Easy-TLS。它是一个Easy-RSA扩展工具，我们正在使用它来生成tls-auth密钥。

单击code选项卡下的Download zip选项。请参考下面的截图。

image-20220120182209404

然后解压“easy-tls-master”文件夹，将“easytls”和“easytls-openssl.cnf”文件拷贝到“C:\Program files \OpenVPN\easy-rsa”目录下。查看下面的截图作为参考。

image-20220120182324122

现在回到EasyRSA shell提示符并输入下面的命令。初始化easy-tls脚本程序。

./easytls init-tls
现在，使用下面的命令生成tls-auth密钥。

./easytls build-tls-auth 
该命令将生成名为“tls-auth”的密钥文件。在“C:\Program Files\OpenVPN\easy-rsa\pki\easytls”文件夹下。请参考下面的截图。

image-20220120182604157

现在我们需要生成Diffie Hellman参数

OpenVPN服务器必须要生成Diffie Hellman参数

这些参数定义了OpenSSL如何执行Diffie-Hellman (DH)密钥交换。Diffie-Hellman密钥交换是一种通过公共信道安全地交换密码密钥的方法

发出下面的命令，从EasyRSA shell生成Diffie Hellman参数(这个过程可能需要1分钟左右时间)

./easyrsa gen-dh
该命令将在“C:\Program Files\OpenVPN\easy-rsa\pki”文件夹下创建dh文件，文件名为“dh .pem”。请参考下面的截图。

image-20220120182929454

这就完成了OpenVPN服务所需的SSL/TLS密钥文件的生成。我们将能够在下面的文件夹中找到创建的文件。

目录	内容
C:\Program Files\OpenVPN\easy-rsa\pki	CA file, DH file and other OpenSSL related files like config file
C:\Program Files\OpenVPN\easy-rsa\pki\private	Include the private key files of CA, Server and Client certificates
C:\Program Files\OpenVPN\easy-rsa\pki\easytls	Contains the tls-auth key
C:\Program Files\OpenVPN\easy-rsa\pki\issued	Contains issued Server and Client certificates
image-20220120183320753

下面是有关文件的简短说明

Filename	Needed By	Purpose	Secret
ca.crt	server + all clients	Root CA certificate	No
ca.key	Server Only	Root CA key	YES
dh.pem	server only	Diffie Hellman parameters	No
SERVER.crt	server only	Server Certificate	No
SERVER.key	server only	Server Key	Yes
CLIENT.crt	Client only	Client Certificate	No
CLIENT.key	Client only	Client Key	Yes
tls-auth.key	server + all clients	Used for tls-auth directive	No
步骤三：配置ip转发和网络共享
打开注册表，win+R，输入regedit.exe，依次找到：HKEY LOCAL MACHINESYSTEM\CurrentControlSet\Services\Tcpip\Parameters将IPEableRouter值改为1，如下图

image-20220120184135935

然后打开控制面板，找到“控制面板\网络和 Internet\网络连接”，右键点击“以太网”，点击“属性”，在“共享”中钩上“允许其他网络用户通过此计算机的internet连接来连接”，并选择“OpenVPN TAP-Windows6”，点击确定。

image-20220120184424489

image-20220120184527697

步骤四：创建服务端配置文件
首先打开Windows资源管理器，进入C:\Program Files\OpenVPN\sample-config文件夹，将server.ovpn文件复制一份到C:\Program Files\OpenVPN\config目录下。

同时将以下文件一并复制到C:\Program Files\OpenVPN\config目录下

ca.crt
dh.pem
SERVER.crt
SERVER.key
tls-auth.key
image-20220120185305290

编辑server.ovpn文件，修改以下地方，其他保持默认即可

local x.x.x.x  # 这个是服务器的内网IP地址，或者保持默认
ca ca.crt
cert SERVER.crt
key SERVER.key
dh dh.pem
push "redirect-gateway def1 bypass-dhcp"
push "dhcp-option DNS 114.114.114.114" # 可以改成其他的DNS服务器
push "dhcp-option DNS 8.8.8.8"
tls-auth tls-auth.key 0 # This file is secret
cipher AES-256-GCM
点击连接，图标变绿就正常。

image-20220120190423076

配置客户端
复制以下文件到你的客户端，并且在同一目录下

ca.crt
CLIENT.crt
CLIENT.key
tls-auth.key
client.ovpn（“C:\Program Files\OpenVPN\sample-config”）
编辑client.ovpn，修改以下参数，其他保持默认

remote *.*.*.* 1194 # 服务器公网IP地址
ca ca.crt
cert CLIENT.crt
key CLIENT.key
tls-auth tls-auth.key 1
cipher AES-256-GCM
下载对应系统的客户端

[OpenVPN Connect Client | Our Official VPN Client | OpenVPN](https://openvpn.net/vpn-client/)

如windows系统，将client.ovpn文件拖入客户端界面即可，如下

image-20220120191352128

随后点击连接，出现如下界面则表示连接成功

image-20220120191650721

这时ping一下10.8.0.1测试是否连通，ping下百度测试DNS是否正常。

image-20220120191756447

这就成功了。

吊销客户端证书
当我们创建了多个用户使用，然后某些原因，个别用户需要禁用的时候，我们就可以使用吊销证书的方式来处理。

首先在你的OpenVPN服务器上，打开cmd管理员权限窗口，进入目录“C:\Program Files\OpenVPN\easy-rsa\”

cd "C:\Program Files\OpenVPN\easy-rsa"
EasyRSA-Start.bat
进入easyrsa的shell界面

image-20220120192518643

输入以下命令

./easyrsa revoke <client>  # 你要吊销的客户端名
./easyrsa gen-crl # 生成crl.pem文件，用来记录吊销的证书
image-20220120192845049

编辑server.ovpn（“C:\Program Files\OpenVPN\config”下），在行尾加入一行

crl-verify "C:\\Program Files\\OpenVPN\\easy-rsa\\pki\\crl.pem" # 用来告知服务端有哪些证书是被吊销的
这一步如果不做则不会生效，添加了这行之后，服务端重连一下即可。

参考：

https://supporthost.in/how-to-setup-openvpn-on-windows-server-2019/

https://www.shangmayuan.com/a/28de9b94188547acb8a48c10.html