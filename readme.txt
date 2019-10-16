# GFWDNS
利用自建DNS来科学上网

不是什么高深的东西，所有的软件都是各路大神已经开发好的。我这里只是从新组合，拓展思路的一个方案。
原理很简单，在不久的以前，也有很多人做过可以自定义DNS服务器这类的app，结合DNS劫持，将gfwlist的域名劫持到海外的SNIPROXY服务，以达到科学上网的目的。
我这里主要记录一下怎么自建这种DNS。至于这里边的好处就是完全自己可控，不受制于其他人。

主体思路如下：
1、利用dnsmasq部署一个DNS缓存服务器，利用dnsmasq的可自定义域名和解析地址的特性，将gfwlist里边的域名劫持到一个国内的目标IP去（通常我的做法就是劫持到我这台DNS服务器）。
2、流量劫持过来之后，数据包的目的端口就变成了DNS服务器的IP地址（后续简称DNSIP）。目的端口就是DNSIP的80和443（这里只考虑普通的http和https浏览页面上网功能）。
3、在DNS服务器上，利用shadowsocks或者v2ray的透明代理功能，将流量转发至你自己的海外VPS服务器，达到翻墙的目的。

思路很简单，玩过dnsmasq，iptables，ss或者v2ray透明代理，这三样东西的人，应该立马知道怎么去做了。

我这里简要说明一下。
1、dnsmasql
安装很简单，主要说一下配置。
最简单的dnsmasq.conf内容如下

################################
conf-dir=/etc/dnsmasq.d
################################

对，你没有看错，其实这么一行不填，它都能正常运行。但我填了这么一行就是为了加载 /etc/dnsmasq.d 这个目录下的配置文件而已。
而这个目录下，有一个我自建的叫做  dnsfq 的文件。 这个文件简略内容如下：

################################
address=/agnesb.fr/192.168.1.10
address=/akiba-web.com/192.168.1.10
address=/altrec.com/192.168.1.10
address=/angela-merkel.de/192.168.1.10
address=/angola.org/192.168.1.10
address=/apartmentratings.com/192.168.1.10
address=/apartments.com/192.168.1.10
address=/arena.taipei/192.168.1.10
address=/asianspiss.com/192.168.1.10
################################

当然，文件内容肯定不止这么多，这个文件的内容应该和gfwlist一样多。我这边大概是 5480多行。
# wc -l /etc/dnsmasq.d/dnsfq 
5485 /etc/dnsmasq.d/dnsfq

怎样获取和更新这个 dnsfq的文件呢，也很简单，我从网上找到的一个python脚本搞定（脚本内作者和来源信息我没有更改，尊重原创者）。 脚本内的变量，按需替换成自己的IP地址即可。

################################
#!/usr/bin/env python  
#coding=utf-8
#  
# Generate a list of dnsmasq rules with ipset for gfwlist
#  
# Copyright (C) 2014 http://www.shuyz.com   
# Ref https://code.google.com/p/autoproxy-gfwlist/wiki/Rules    
 
import urllib2 
import re
import os
import datetime
import base64
import shutil
 
myproxyip = '192.168.1.10'

# the url of gfwlist
baseurl = 'https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt'
# match comments/title/whitelist/ip address
comment_pattern = '^\!|\[|^@@|^\d+\.\d+\.\d+\.\d+'
domain_pattern = '([\w\-\_]+\.[\w\.\-\_]+)[\/\*]*' 

#outfile = '/etc/dnsmasq.d/backup/dnsfq'
outfile = '/etc/dnsmasq.d/dnsfq'
tmpfile = '/tmp/gfwlisttmp'
 
print 'fetching list...'
content = urllib2.urlopen(baseurl, timeout=15).read().decode('base64')
print 'page content fetched, analysis...'

tfs = open(tmpfile, 'w')
tfs.write(content)
tfs.close()

fs =  file(outfile, 'w')

domainlist = []
tfs = open(tmpfile, 'r')
for line in tfs.readlines():
    if re.findall(comment_pattern, line):
        #print 'this is a comment line: ' + line
        comment = 'this is a comment line: ' + line
    else:
        domain = re.findall(domain_pattern, line)
        if domain:
            try:
                found = domainlist.index(domain[0])
                #print domain[0] + ' exists.'
            except ValueError:
                #print 'saving ' + domain[0]
                domainlist.append(domain[0])
                fs.write('address=/%s/%s\n'%(domain[0],myproxyip))

tfs.close()
fs.close()
################################

这样，就可以启动dnsmasq服务了。启动起来之后，你可以在linux里使用dig，windows里使用nslookup进行测试。看看 www.google.com 是否已经解析成 192.168.1.10 这个内网IP地址了。  也别忘了测试一些 gfwlist 以外的地址，例如  www.baidu.com 是否仍然解析成它应该的外网地址。

检查无误后，你就可以将你的手机或者电脑的网卡配置里的DNS服务器地址，改成这个DNSIP了（192.168.1.10）.
到此，你的手机和电脑的DNS已经被自己劫持到自己的DNS服务器了。 但还不能科学上网。

2、利用iptables对目标端口为80和443的流量进行REDIRECT操作。
方法也很简单：
################################
echo 1 > /proc/sys/net/ipv4/ip_forward ##开启linux系统的数据包转发功能

iptables -t nat -I PREROUTING -i eth0 -p tcp -m multiport --dports 80,443 -j REDIRECT --to-ports 1080
##这里重点说明一下这条iptables，  作用是在服务器的nat规则的PREROUTING链，插入一条规则，将目的端口80和443的数据包重定向到服务器的 1080端口。注意，除了这条iptables，尽量将服务器的其他任何iptables规则提前清空，除非你自己很明白iptables怎么回事。
##如果无法清空本地iptables，那请针对上述规则，进行放行策略。
iptables -I FORWARD -d 192.168.1.10 -j ACCEPT
iptables -I INPUT --dport 1080 -j ACCEPT
################################

至此，你访问google.com的数据包也已经成功劫持到了DNS服务器的 1080端口。

3、最后一步就是将DNS服务器的1080弄成透明代理了。
我之前用的是SS，因为后续GFW升级导致封锁严重，现在已经换成v2ray+ws+tls了。我逐一贴一下简要步骤。

a) SS

ss-redir 配置文件如下：
################################
{
"server": "$IP",
"server_port": "$PORT",
"local_address": "0.0.0.0",
"local_port": "1080",
"password": "$PASSWORD",
"timeout": "300",
"method": "aes-256-cfb",
"protocol": "auth_sha1_v4",
"protocol_param": "",
"obfs": "http_simple",
"obfs_param": "www.baidu.com"
}
################################

启动命令如下：
################################
ss-redir -c ss-redir.json &
################################

V2Ray 配置文件如下(默认配置文件/etc/v2ray/config.json)：
################################
{
  "inbounds": [

 {
   "port": 1080,
   "protocol": "dokodemo-door",
   "settings": {
     "network": "tcp,udp",
     "followRedirect": true
   },
   "sniffing": {
     "enabled": true,
     "destOverride": ["http", "tls"]
   }
 },

    {
      "port": 1081,
      "listen": "127.0.0.1",
      "settings": {
        "udp": false
      },
      "protocol": "socks"
    }
  ],
  "outbounds": [
    {
      "protocol": "vmess",
      "settings": {
        "vnext": [
          {
            "address": "你自己的域名或者IP地址",
            "users": [
              {
                "id": "你自己的海外VPS服务端的UUID",
                "alterId": 64,
                "security": "auto",
                "level": 0
              }
            ],
            "port": 443
          }
        ]
      },
      "streamSettings": {
        "wsSettings": {
          "path": "你自己的PATH地址",
          "headers": {}
        },
        "tlsSettings": {
          "allowInsecure": true,
          "alpn": [
            "http/1.1"
          ],
          "serverName": "你自己的域名或者IP地址",
          "allowInsecureCiphers": true
        },
        "security": "tls",
        "network": "ws",
        "sockopt": {}
      }
    }
  ]
}
################################

v2ray的启动
################################
systemctl start v2ray
################################

是不是很简单，这样就都搞定了！可以试试这个DNS服务器的翻墙效果了。


对于DNS服务器实际使用时候遇到的一些问题，其实还有其他高深的衍生玩法。我这里大致说一下思路。后续可能会详细写出来。
问题一： 这个DNS服务器我放到外网行不行？
行，没问题。但要面临两个大问题。 
1、这个DNS服务器会不会被其他的人利用进行科学上网？

我大致回答一下，如果你将DNS服务器放到外网，比如阿里云，或者其他的IDC，那必然暴露了 udp53 这个DNS端口 以及更重要的 80和443 端口 。那别人一定会用。

这怎么办？ 可以结合ipset和iptables 自定义一个“开关”，来达到只有我可以用的目的。
怎么做？ 将服务器的 udp 53 端口默认设置为拒绝所有IP链接。然后写一个定时脚本，每隔5秒钟去扫一下 dnsmasq的查询日志最后500行，然后过滤一个你自定义的关键字，然后触发iptables将你的外网IP地址设置位允许访问本机的 udp53 端口。 
我给一下我这拙略的步骤，以作参考：
################################
先建立ipset规则集
ipset -N allowip iphash
ipset flush allowip

然后修改之前提到的iptables转发规则如下：
iptables -t nat -A PREROUTING -i eth0 -p tcp -m set --match-set allowip src -m multiport --dports 80,443 -j  REDIRECT --to-ports 1080

最后，增加允许访问dns服务和80，443端口的规则
iptables -A INPUT -p tcp -m set --match-set allowip src --dport 1080 -j ACCEPT
iptables -A INPUT -p udp -m set --match-set allowip src --dport 53 -j ACCEPT
################################

定时脚本内容：
################################
while ((1));do

    for IP in $(tail -n 500 "dnsmasq的日志" | awk '/query\[A\].*关键字.*/{a[$NF]++}END{for (i in a)print i}')
    do
        ipset -A allowip $IP 2> /dev/null && echo "$(date +%F_%T) added $IP"
    done

    sleep 5

done
################################

2、放到外网后，怎样规避自家宽带运营商的DNS劫持？
这个没什么好办法，只能是把dnsmasq的端口从默认的 udp 53 改为其他非标端口。 但本地客户端，例如手机，电脑可就不那么好弄向外的流量重定向了。
其实我也只是知道 linux利用iptables怎么做。
iptables -t nat -A OUTPUT -d 208.67.222.222/32 -p udp -m udp --dport 53 -j DNAT --to-destination 208.67.222.222:443

所以运营商劫持DNS没什么好办法。 我这里写的也就更适用于内网环境而已。比如一家规模不大的公司内，或者一个团队使用。

