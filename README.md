# 使用SSR搭建自己的国内中转MTP代理

> 七夏 April 12, 2019

本文参考自《[使用v2ray搭建自己的墙内中转mtp代理](https://telegra.ph/%E4%BD%BF%E7%94%A8v2ray%E6%90%AD%E5%BB%BA%E8%87%AA%E5%B7%B1%E7%9A%84%E5%A2%99%E5%86%85%E4%B8%AD%E8%BD%ACmtp%E4%BB%A3%E7%90%86-03-21)》   
> 好像是这个月初，电报圈里面的童鞋都发现自己的MTP代理都不好使了！大家经过了一番讨论后，怀疑是一种神秘的东方力量让普通的MTP连不上了。
> 
> 挺久之前，我在x-air的频道上面看到了他们写的v2ray转换MTP的配置文件，但是用的人不多。现在封锁变得越来越严，是时候普及一下墙内中转MTP了。

本文主要介绍使用SSR搭建自己的国内中转MTP代理的方法   

##### 一、服务器选购   

首先要一台国内的服务器，参考文章里面推荐的是阿里云学生机，这里推荐一下其他几家比较划算的国内NAT机商家的机器，带宽也比较大，加载图片之类的会更快一些   

###### 第一家：[OLVPS](https://t667.com/aff.php?aff=261&gid=1)    

兔子家的，推荐买江苏镇江的NAT机，25元那款最划算，不过现在没货，不过随时会补货，订购链接：[Zhenjiang Kvm Nat 256](https://t667.com/aff.php?aff=261&a=add&pid=57)，可以去[梨园论坛](https://bbs.liyuans.com/)看看有没有人卖，最近看见有不少人出   
如果有货的话江苏的 _Jiangsu Kvm Nat 256_ 那款也行   

###### 第二家：[昱格云](https://www.ygeidc.com/aff.php?aff=55&gid=9)   

老咸鱼家的，推荐买绍兴柯桥双线，也是25元那款（现货），订购链接：[SXNat-Bronze](https://www.ygeidc.com/aff.php?aff=55&a=add&pid=24)   
然后他家的江苏高防BGP也不错，就是最近出墙有点问题，商家正在解决，可以的话BGP那款对联通用户更友好一点，也是25元（现货，而且可以使用循环5折优惠码 <font color="red">_JSNat-50_</font> 折后只要14.99元一个月），订购链接：[JSNat-Bronze](https://www.ygeidc.com/aff.php?aff=55&a=add&pid=35)   

另外如果需要不限流量的可以看看 [PQS](https://www.pqs.pw/aff.php?aff=31&gid=15) 这家新上的深圳电信家宽，出墙应该更快   
更便宜的还有 CanBesystems(cbvps) 家的[河南BGP](http://www.cbvps.net/cart.php?gid=14)，不过我没用过   

然后需要一台国外搭建了SSR的服务器，可以自己购买服务器搭建，国外服务器选购我就不推荐了，太杂乱了，搭建方式可以用~~逗比~~（怀念逗比大佬）的脚本，教程：[ssrmu.sh](https://doubibackup.com/z2a4lk3l-2.html) ，不过建议选择购买各大机场的SSR服务，更划算一点，机场选购可以参考一个telegram评测频道，链接： [Professional-V1 Blog](https://t.me/V1_BLOG)   

##### 二、服务搭建   

###### 1.安装SSR客户端   

首先安装git等常用工具，使用XShell或其他ssh终端登陆上你的服务器，建议先用 `sudo su` 命令切换到root用户方便接下来的操作，然后输入下面的代码：   
```bash
# Debian/Ubuntu 执行
apt-get install -y wget ca-certificates openssl sudo vim git curl gcc unzip
# CentOS 执行
yum install -y wget ca-certificates openssl sudo vim git curl gcc unzip
```
> Tips: 如果SSR用到了 chacha20 类的加密方式，请先安装 libsodium 加密库，逗比大佬的脚本：
> ```bash
> wget -N https://raw.githubusercontent.com/ToyoDAdoubi/doubi/master/libsodium.sh && chmod +x libsodium.sh && bash libsodium.sh
>```

然后安装SSR   
```bash
wget -O ssr https://raw.githubusercontent.com/the0demiurge/CharlesScripts/master/charles/bin/ssr
mv ssr /usr/local/bin && chmod +x /usr/local/bin/ssr
ssr install
```
由于国内服务器的原因，ssr install 这一步可能会进行比较缓慢，完整执行后如下所示：   
```shell
[root@localhost ~]# ssr install
Cloning into '/usr/local/share/shadowsocksr'...
remote: Counting objects: 5490, done.
remote: Total 5490 (delta 0), reused 0 (delta 0), pack-reused 5490
Receiving objects: 100% (5490/5490), 1.71 MiB | 410.00 KiB/s, done.
Resolving deltas: 100% (3799/3799), done.
```
紧接着就可以配置SSR客户端啦，使用 `ssr config` 来修改配置文件，然后使用 `ssr start` 启动SSR，配置文件路径 `/usr/local/share/shadowsocksr/config.json`    
> <font color="red">**注意**</font>: 如果不会使用vim，建议先把下面的命令全部复制到你本地的文本编辑器里面，替换掉上面的配置信息（SSR服务器的IP端口和密码等），然后将全部内容复制到剪切板一起执行   

```bash
# 以下全部内容是一个整体，是一个命令，修改后全部复制粘贴到SSH软件中并一起执行！
echo '{
    "server": "13.233.233.233", //SSR服务器IP
    "server_ipv6": "::",
    "server_port": 8080, //SSR服务器端口
    "local_address": "127.0.0.1",
    "local_port": 1080,
    "password": "123456", //对应password，即SSR的密码
    "method": "aes-128-ctr", //对应method，加密方式，即SSR的加密
    "protocol": "auth_aes128_md5", //对应protocl，即SSR的协议
    "protocol_param": "", //对应protocol_param，即SSR的协议参数
    "obfs": "http_simple", //对应obfs，即SSR的混淆
    "obfs_param": "hello.world", //对应obfs_param，即SSR的混淆参数
    "speed_limit_per_con": 0,
    "speed_limit_per_user": 0,
    "additional_ports" : {},
    "additional_ports_only" : false,
    "timeout": 120,
    "udp_timeout": 60,
    "dns_ipv6": false,
    "connect_verbose_info": 0,
    "redirect": "",
    "fast_open": false
}' > /usr/local/share/shadowsocksr/config.json
```
> Tips:    
> 启动 `ssr start`   
> 停止 `ssr stop`   
> 卸载 `ssr uninstall`（这里操作会删除/usr/local/share/shadowsocksr）   
> 帮助及更多命令 `ssr help`   

###### 2.V2Ray/MTP搭建   

本教程使用的是V2Ray自带的MTProto模块搭建MTP   
下载 <https://github.com/v2ray/v2ray-core/releases/download/v4.18.0/v2ray-linux-64.zip> 然后解压，把里面的 v2ray v2ctl 两个文件上传到服务器里面，或者在服务器里面执行代码：   
```bash
mkdir mtp && cd mtp
wget https://github.com/v2ray/v2ray-core/releases/download/v4.18.0/v2ray-linux-64.zip
unzip v2ray-linux-64.zip
```
> Tips: 访问 <https://github.com/v2ray/v2ray-core/releases> 可以获取最新的V2Ray版本号   
> 另外如果下载太慢，可以试一下x-air的镜像，使用下面的命令：   
>    
> ```bash
> mkdir mtp && cd mtp
> wget https://xairforge.cloudns.asia/v2client/lix/v2ray-linux-64.zip
> unzip v2ray-linux-64.zip
> ```

接下来我们需要替换掉原来的配置文件 config.json   
先把下面的命令全部复制到你本地的文本编辑器里面，然后再在服务器上执行 `openssl rand -hex 16` 来获取到一串由阿拉伯数字和字母abcdef组成的32位的字符作为MTP代理的密码，填入如下文本中的 _mySecret_ 处，更改MTP代理的端口为你想要的端口，然后将全部内容复制到剪切板一起执行   

```bash
# 以下全部内容是一个整体，是一个命令，修改后全部复制粘贴到SSH软件中并一起执行！
echo '{
    "policy":{
        "levels":{
            "0":{
                "uplinkOnly":0
            }
        }
    },
    "dns":{
        "servers":[
            "208.67.222.222"
        ]
    },
    "outboundDetour":[
        {
            "tag":"tg-out",
            "protocol":"mtproto",
            "settings":{

            },
            "proxySettings":{
                "tag":"socks-out"
            }
        }
    ],
    "inbound":{
        "tag":"tg-in",
        "port":3389, //这个是MTP代理的端口
        "protocol":"mtproto",
        "settings":{
            "users":[
                {
                    "secret":"mySecret" //这个是MTP代理的密码，生成方式见上方说明
                }
            ]
        }
    },
    "log":{
        "loglevel":"warning"
    },
    "routing":{
        "strategy":"rules",
        "settings":{
            "domainStrategy":"IPIfNonMatch",
            "rules":[
                {
                    "type":"field",
                    "inboundTag":[
                        "tg-in"
                    ],
                    "outboundTag":"tg-out"
                }
            ]
        }
    },
    "outbound":{
        "tag":"socks-out",
        "sendThrough":"0.0.0.0",
        "protocol":"socks",
        "settings":{
            "servers":[
                {
                    "address":"127.0.0.1",
                    "port":1080
                }
            ]
        },
        "streamSettings":{
            "network":"tcp",
            "security":""
        }
    }
}' > config.json
```
配置好过后，就可以运行了   
```bash
chmod +x v2ray && chmod +x v2ctl
./v2ray
```
如果没有发现什么报错，配置就是成功的了。为了让他在后台执行，我们可以先按ctrl+c，退出当前程序，执行下面的代码：`nohup ./v2ray >> /dev/null 2>&1 &`   
最后放行对应的MTP代理端口（将3389替换成你前面自己设置的MTP端口即可）：   
```bash
# CentOS 执行
service iptables save
service ip6tables save
chkconfig --level 2345 iptables on
chkconfig --level 2345 ip6tables on
iptables -I INPUT -m state --state NEW -m tcp -p tcp --dport 3389 -j ACCEPT
ip6tables -I INPUT -m state --state NEW -m tcp -p tcp --dport 3389 -j ACCEPT
service iptables save
service ip6tables save

# Debian/Ubuntu 执行
iptables-save > /etc/iptables.up.rules
ip6tables-save > /etc/ip6tables.up.rules
echo -e '#!/bin/bash\n/sbin/iptables-restore < /etc/iptables.up.rules\n/sbin/ip6tables-restore < /etc/ip6tables.up.rules' > /etc/network/if-pre-up.d/iptables
chmod +x /etc/network/if-pre-up.d/iptables
iptables -I INPUT -m state --state NEW -m tcp -p tcp --dport 3389 -j ACCEPT
ip6tables -I INPUT -m state --state NEW -m tcp -p tcp --dport 3389 -j ACCEPT
iptables-save > /etc/iptables.up.rules
ip6tables-save > /etc/ip6tables.up.rules
```
到此，就可以在你的设备上连接MTP测试啦  
然后每次开机运行如下命令即可：   
```bash
ssr start
cd /root/mtp
nohup ./v2ray >> /dev/null 2>&1 &
```   

> 另外，使用NAT服务器设置端口映射可以参考《[使用NAT VPS服务并联机](http://www.cbvps.net/index.php/knowledgebase/1/NAT-VPS.html) 》   

<font color="red">**注意**</font>：写教程好累的，所以本文上述服务器购买链接都有我的AFF，如果真的很介意的话，下面是所有不带AFF的链接qwq   
> OLVPS: <https://t667.com/cart.php?gid=1>   
> Zhenjiang Kvm Nat 256: <https://t667.com/cart.php?a=add&pid=57>   
> 昱格云: <https://www.ygeidc.com/cart.php?gid=9>   
> SXNat-Bronze: <https://www.ygeidc.com/cart.php?a=add&pid=24>   
> JSNat-Bronze: <https://www.ygeidc.com/cart.php?a=add&pid=35> 循环5折优惠码 <font color="red">_JSNat-50_</font>    
> PQS: <https://www.pqs.pw/cart.php?gid=15>   
> cbvps: <http://www.cbvps.net/cart.php?gid=14>   
