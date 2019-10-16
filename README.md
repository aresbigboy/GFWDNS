<html>

<head>
<meta http-equiv=Content-Type content="text/html; charset=gb2312">
<meta name=Generator content="Microsoft Word 15 (filtered)">
<style>
<!--
 /* Font Definitions */
 @font-face
	{font-family:"Cambria Math";
	panose-1:2 4 5 3 5 4 6 3 2 4;}
@font-face
	{font-family:等线;
	panose-1:2 1 6 0 3 1 1 1 1 1;}
@font-face
	{font-family:"\@等线";
	panose-1:2 1 6 0 3 1 1 1 1 1;}
 /* Style Definitions */
 p.MsoNormal, li.MsoNormal, div.MsoNormal
	{margin:0cm;
	margin-bottom:.0001pt;
	text-align:justify;
	text-justify:inter-ideograph;
	font-size:10.5pt;
	font-family:等线;}
.MsoChpDefault
	{font-family:等线;}
 /* Page Definitions */
 @page WordSection1
	{size:595.3pt 841.9pt;
	margin:72.0pt 90.0pt 72.0pt 90.0pt;
	layout-grid:15.6pt;}
div.WordSection1
	{page:WordSection1;}
-->
</style>

</head>

<body lang=ZH-CN style='text-justify-trim:punctuation'>

<div class=WordSection1 style='layout-grid:15.6pt'>

<p class=MsoNormal><span lang=EN-US># GFWDNS</span></p>

<p class=MsoNormal>利用自建<span lang=EN-US>DNS</span>来科学上网</p>

<p class=MsoNormal><span lang=EN-US>&nbsp;</span></p>

<p class=MsoNormal>不是什么高深的东西，所有的软件都是各路大神已经开发好的。我这里只是从新组合，拓展思路的一个方案。</p>

<p class=MsoNormal>原理很简单，在不久的以前，也有很多人做过可以自定义<span lang=EN-US>DNS</span>服务器这类的<span
lang=EN-US>app</span>，结合<span lang=EN-US>DNS</span>劫持，将<span lang=EN-US>gfwlist</span>的域名劫持到海外的<span
lang=EN-US>SNIPROXY</span>服务，以达到科学上网的目的。</p>

<p class=MsoNormal>我这里主要记录一下怎么自建这种<span lang=EN-US>DNS</span>。至于这里边的好处就是完全自己可控，不受制于其他人。</p>

<p class=MsoNormal><span lang=EN-US>&nbsp;</span></p>

<p class=MsoNormal>主体思路如下：</p>

<p class=MsoNormal><span lang=EN-US>1</span>、利用<span lang=EN-US>dnsmasq</span>部署一个<span
lang=EN-US>DNS</span>缓存服务器，利用<span lang=EN-US>dnsmasq</span>的可自定义域名和解析地址的特性，将<span
lang=EN-US>gfwlist</span>里边的域名劫持到一个国内的目标<span lang=EN-US>IP</span>去（通常我的做法就是劫持到我这台<span
lang=EN-US>DNS</span>服务器）。</p>

<p class=MsoNormal><span lang=EN-US>2</span>、流量劫持过来之后，数据包的目的端口就变成了<span
lang=EN-US>DNS</span>服务器的<span lang=EN-US>IP</span>地址（后续简称<span lang=EN-US>DNSIP</span>）。目的端口就是<span
lang=EN-US>DNSIP</span>的<span lang=EN-US>80</span>和<span lang=EN-US>443</span>（这里只考虑普通的<span
lang=EN-US>http</span>和<span lang=EN-US>https</span>浏览页面上网功能）。</p>

<p class=MsoNormal><span lang=EN-US>3</span>、在<span lang=EN-US>DNS</span>服务器上，利用<span
lang=EN-US>shadowsocks</span>或者<span lang=EN-US>v2ray</span>的透明代理功能，将流量转发至你自己的海外<span
lang=EN-US>VPS</span>服务器，达到翻墙的目的。</p>

<p class=MsoNormal><span lang=EN-US>&nbsp;</span></p>

<p class=MsoNormal>思路很简单，玩过<span lang=EN-US>dnsmasq</span>，<span lang=EN-US>iptables</span>，<span
lang=EN-US>ss</span>或者<span lang=EN-US>v2ray</span>透明代理，这三样东西的人，应该立马知道怎么去做了。</p>

<p class=MsoNormal><span lang=EN-US>&nbsp;</span></p>

<p class=MsoNormal>我这里简要说明一下。</p>

<p class=MsoNormal><span lang=EN-US>1</span>、<span lang=EN-US>dnsmasql</span></p>

<p class=MsoNormal>安装很简单，主要说一下配置。</p>

<p class=MsoNormal>最简单的<span lang=EN-US>dnsmasq.conf</span>内容如下</p>

<p class=MsoNormal><span lang=EN-US>&nbsp;</span></p>

<p class=MsoNormal><span lang=EN-US>################################</span></p>

<p class=MsoNormal><span lang=EN-US>conf-dir=/etc/dnsmasq.d</span></p>

<p class=MsoNormal><span lang=EN-US>################################</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;</span></p>

<p class=MsoNormal>对，你没有看错，其实这么一行不填，它都能正常运行。但我填了这么一行就是为了加载 <span lang=EN-US>/etc/dnsmasq.d
</span>这个目录下的配置文件而已。</p>

<p class=MsoNormal>而这个目录下，有一个我自建的叫做<span lang=EN-US>&nbsp; dnsfq </span>的文件。 这个文件简略内容如下：</p>

<p class=MsoNormal><span lang=EN-US>&nbsp;</span></p>

<p class=MsoNormal><span lang=EN-US>################################</span></p>

<p class=MsoNormal><span lang=EN-US>address=/agnesb.fr/192.168.1.10</span></p>

<p class=MsoNormal><span lang=EN-US>address=/akiba-web.com/192.168.1.10</span></p>

<p class=MsoNormal><span lang=EN-US>address=/altrec.com/192.168.1.10</span></p>

<p class=MsoNormal><span lang=EN-US>address=/angela-merkel.de/192.168.1.10</span></p>

<p class=MsoNormal><span lang=EN-US>address=/angola.org/192.168.1.10</span></p>

<p class=MsoNormal><span lang=EN-US>address=/apartmentratings.com/192.168.1.10</span></p>

<p class=MsoNormal><span lang=EN-US>address=/apartments.com/192.168.1.10</span></p>

<p class=MsoNormal><span lang=EN-US>address=/arena.taipei/192.168.1.10</span></p>

<p class=MsoNormal><span lang=EN-US>address=/asianspiss.com/192.168.1.10</span></p>

<p class=MsoNormal><span lang=EN-US>################################</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;</span></p>

<p class=MsoNormal>当然，文件内容肯定不止这么多，这个文件的内容应该和<span lang=EN-US>gfwlist</span>一样多。我这边大概是<span
lang=EN-US> 5480</span>多行。</p>

<p class=MsoNormal><span lang=EN-US># wc -l /etc/dnsmasq.d/dnsfq </span></p>

<p class=MsoNormal><span lang=EN-US>5485 /etc/dnsmasq.d/dnsfq</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;</span></p>

<p class=MsoNormal>怎样获取和更新这个<span lang=EN-US> dnsfq</span>的文件呢，也很简单，我从网上找到的一个<span
lang=EN-US>python</span>脚本搞定（脚本内作者和来源信息我没有更改，尊重原创者）。 脚本内的变量，按需替换成自己的<span
lang=EN-US>IP</span>地址即可。</p>

<p class=MsoNormal><span lang=EN-US>&nbsp;</span></p>

<p class=MsoNormal><span lang=EN-US>################################</span></p>

<p class=MsoNormal><span lang=EN-US>#!/usr/bin/env python&nbsp; </span></p>

<p class=MsoNormal><span lang=EN-US>#coding=utf-8</span></p>

<p class=MsoNormal><span lang=EN-US>#&nbsp; </span></p>

<p class=MsoNormal><span lang=EN-US># Generate a list of dnsmasq rules with
ipset for gfwlist</span></p>

<p class=MsoNormal><span lang=EN-US>#&nbsp; </span></p>

<p class=MsoNormal><span lang=EN-US># Copyright (C) 2014
http://www.shuyz.com&nbsp;&nbsp; </span></p>

<p class=MsoNormal><span lang=EN-US># Ref
https://code.google.com/p/autoproxy-gfwlist/wiki/Rules&nbsp;&nbsp;&nbsp; </span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;</span></p>

<p class=MsoNormal><span lang=EN-US>import urllib2 </span></p>

<p class=MsoNormal><span lang=EN-US>import re</span></p>

<p class=MsoNormal><span lang=EN-US>import os</span></p>

<p class=MsoNormal><span lang=EN-US>import datetime</span></p>

<p class=MsoNormal><span lang=EN-US>import base64</span></p>

<p class=MsoNormal><span lang=EN-US>import shutil</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;</span></p>

<p class=MsoNormal><span lang=EN-US>myproxyip = '192.168.1.10'</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;</span></p>

<p class=MsoNormal><span lang=EN-US># the url of gfwlist</span></p>

<p class=MsoNormal><span lang=EN-US>baseurl =
'https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt'</span></p>

<p class=MsoNormal><span lang=EN-US># match comments/title/whitelist/ip address</span></p>

<p class=MsoNormal><span lang=EN-US>comment_pattern =
'^\!|\[|^@@|^\d+\.\d+\.\d+\.\d+'</span></p>

<p class=MsoNormal><span lang=EN-US>domain_pattern =
'([\w\-\_]+\.[\w\.\-\_]+)[\/\*]*' </span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;</span></p>

<p class=MsoNormal><span lang=EN-US>#outfile = '/etc/dnsmasq.d/backup/dnsfq'</span></p>

<p class=MsoNormal><span lang=EN-US>outfile = '/etc/dnsmasq.d/dnsfq'</span></p>

<p class=MsoNormal><span lang=EN-US>tmpfile = '/tmp/gfwlisttmp'</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;</span></p>

<p class=MsoNormal><span lang=EN-US>print 'fetching list...'</span></p>

<p class=MsoNormal><span lang=EN-US>content = urllib2.urlopen(baseurl,
timeout=15).read().decode('base64')</span></p>

<p class=MsoNormal><span lang=EN-US>print 'page content fetched, analysis...'</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;</span></p>

<p class=MsoNormal><span lang=EN-US>tfs = open(tmpfile, 'w')</span></p>

<p class=MsoNormal><span lang=EN-US>tfs.write(content)</span></p>

<p class=MsoNormal><span lang=EN-US>tfs.close()</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;</span></p>

<p class=MsoNormal><span lang=EN-US>fs =&nbsp; file(outfile, 'w')</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;</span></p>

<p class=MsoNormal><span lang=EN-US>domainlist = []</span></p>

<p class=MsoNormal><span lang=EN-US>tfs = open(tmpfile, 'r')</span></p>

<p class=MsoNormal><span lang=EN-US>for line in tfs.readlines():</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp;&nbsp; if re.findall(comment_pattern,
line):</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
#print 'this is a comment line: ' + line</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
comment = 'this is a comment line: ' + line</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp;&nbsp; else:</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
domain = re.findall(domain_pattern, line)</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
if domain:</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
try:</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
found = domainlist.index(domain[0])</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
#print domain[0] + ' exists.'</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
except ValueError:</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
#print 'saving ' + domain[0]</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
domainlist.append(domain[0])</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
fs.write('address=/%s/%s\n'%(domain[0],myproxyip))</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;</span></p>

<p class=MsoNormal><span lang=EN-US>tfs.close()</span></p>

<p class=MsoNormal><span lang=EN-US>fs.close()</span></p>

<p class=MsoNormal><span lang=EN-US>################################</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;</span></p>

<p class=MsoNormal>这样，就可以启动<span lang=EN-US>dnsmasq</span>服务了。启动起来之后，你可以在<span
lang=EN-US>linux</span>里使用<span lang=EN-US>dig</span>，<span lang=EN-US>windows</span>里使用<span
lang=EN-US>nslookup</span>进行测试。看看<span lang=EN-US> www.google.com </span>是否已经解析成<span
lang=EN-US> 192.168.1.10 </span>这个内网<span lang=EN-US>IP</span>地址了。<span
lang=EN-US>&nbsp; </span>也别忘了测试一些<span lang=EN-US> gfwlist </span>以外的地址，例如<span
lang=EN-US>&nbsp; www.baidu.com </span>是否仍然解析成它应该的外网地址。</p>

<p class=MsoNormal><span lang=EN-US>&nbsp;</span></p>

<p class=MsoNormal>检查无误后，你就可以将你的手机或者电脑的网卡配置里的<span lang=EN-US>DNS</span>服务器地址，改成这个<span
lang=EN-US>DNSIP</span>了（<span lang=EN-US>192.168.1.10</span>）<span lang=EN-US>.</span></p>

<p class=MsoNormal>到此，你的手机和电脑的<span lang=EN-US>DNS</span>已经被自己劫持到自己的<span
lang=EN-US>DNS</span>服务器了。 但还不能科学上网。</p>

<p class=MsoNormal><span lang=EN-US>&nbsp;</span></p>

<p class=MsoNormal><span lang=EN-US>2</span>、利用<span lang=EN-US>iptables</span>对目标端口为<span
lang=EN-US>80</span>和<span lang=EN-US>443</span>的流量进行<span lang=EN-US>REDIRECT</span>操作。</p>

<p class=MsoNormal>方法也很简单：</p>

<p class=MsoNormal><span lang=EN-US>################################</span></p>

<p class=MsoNormal><span lang=EN-US>echo 1 &gt; /proc/sys/net/ipv4/ip_forward
##</span>开启<span lang=EN-US>linux</span>系统的数据包转发功能</p>

<p class=MsoNormal><span lang=EN-US>&nbsp;</span></p>

<p class=MsoNormal><span lang=EN-US>iptables -t nat -I PREROUTING -i eth0 -p
tcp -m multiport --dports 80,443 -j REDIRECT --to-ports 1080</span></p>

<p class=MsoNormal><span lang=EN-US>##</span>这里重点说明一下这条<span lang=EN-US>iptables</span>，<span
lang=EN-US>&nbsp; </span>作用是在服务器的<span lang=EN-US>nat</span>规则的<span
lang=EN-US>PREROUTING</span>链，插入一条规则，将目的端口<span lang=EN-US>80</span>和<span
lang=EN-US>443</span>的数据包重定向到服务器的<span lang=EN-US> 1080</span>端口。注意，除了这条<span
lang=EN-US>iptables</span>，尽量将服务器的其他任何<span lang=EN-US>iptables</span>规则提前清空，除非你自己很明白<span
lang=EN-US>iptables</span>怎么回事。</p>

<p class=MsoNormal><span lang=EN-US>##</span>如果无法清空本地<span lang=EN-US>iptables</span>，那请针对上述规则，进行放行策略。</p>

<p class=MsoNormal><span lang=EN-US>iptables -I FORWARD -d 192.168.1.10 -j
ACCEPT</span></p>

<p class=MsoNormal><span lang=EN-US>iptables -I INPUT --dport 1080 -j ACCEPT</span></p>

<p class=MsoNormal><span lang=EN-US>################################</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;</span></p>

<p class=MsoNormal>至此，你访问<span lang=EN-US>google.com</span>的数据包也已经成功劫持到了<span
lang=EN-US>DNS</span>服务器的<span lang=EN-US> 1080</span>端口。</p>

<p class=MsoNormal><span lang=EN-US>&nbsp;</span></p>

<p class=MsoNormal><span lang=EN-US>3</span>、最后一步就是将<span lang=EN-US>DNS</span>服务器的<span
lang=EN-US>1080</span>弄成透明代理了。</p>

<p class=MsoNormal>我之前用的是<span lang=EN-US>SS</span>，因为后续<span lang=EN-US>GFW</span>升级导致封锁严重，现在已经换成<span
lang=EN-US>v2ray+ws+tls</span>了。我逐一贴一下简要步骤。</p>

<p class=MsoNormal><span lang=EN-US>&nbsp;</span></p>

<p class=MsoNormal><span lang=EN-US>a) SS</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;</span></p>

<p class=MsoNormal><span lang=EN-US>ss-redir </span>配置文件如下：</p>

<p class=MsoNormal><span lang=EN-US>################################</span></p>

<p class=MsoNormal><span lang=EN-US>{</span></p>

<p class=MsoNormal><span lang=EN-US>&quot;server&quot;: &quot;$IP&quot;,</span></p>

<p class=MsoNormal><span lang=EN-US>&quot;server_port&quot;: &quot;$PORT&quot;,</span></p>

<p class=MsoNormal><span lang=EN-US>&quot;local_address&quot;:
&quot;0.0.0.0&quot;,</span></p>

<p class=MsoNormal><span lang=EN-US>&quot;local_port&quot;: &quot;1080&quot;,</span></p>

<p class=MsoNormal><span lang=EN-US>&quot;password&quot;: &quot;$PASSWORD&quot;,</span></p>

<p class=MsoNormal><span lang=EN-US>&quot;timeout&quot;: &quot;300&quot;,</span></p>

<p class=MsoNormal><span lang=EN-US>&quot;method&quot;:
&quot;aes-256-cfb&quot;,</span></p>

<p class=MsoNormal><span lang=EN-US>&quot;protocol&quot;:
&quot;auth_sha1_v4&quot;,</span></p>

<p class=MsoNormal><span lang=EN-US>&quot;protocol_param&quot;: &quot;&quot;,</span></p>

<p class=MsoNormal><span lang=EN-US>&quot;obfs&quot;: &quot;http_simple&quot;,</span></p>

<p class=MsoNormal><span lang=EN-US>&quot;obfs_param&quot;:
&quot;www.baidu.com&quot;</span></p>

<p class=MsoNormal><span lang=EN-US>}</span></p>

<p class=MsoNormal><span lang=EN-US>################################</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;</span></p>

<p class=MsoNormal>启动命令如下：</p>

<p class=MsoNormal><span lang=EN-US>################################</span></p>

<p class=MsoNormal><span lang=EN-US>ss-redir -c ss-redir.json &amp;</span></p>

<p class=MsoNormal><span lang=EN-US>################################</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;</span></p>

<p class=MsoNormal><span lang=EN-US>V2Ray </span>配置文件如下<span lang=EN-US>(</span>默认配置文件<span
lang=EN-US>/etc/v2ray/config.json)</span>：</p>

<p class=MsoNormal><span lang=EN-US>################################</span></p>

<p class=MsoNormal><span lang=EN-US>{</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp; &quot;inbounds&quot;: [</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;{</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp; &quot;port&quot;: 1080,</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp; &quot;protocol&quot;:
&quot;dokodemo-door&quot;,</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp; &quot;settings&quot;: {</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp;&nbsp;&nbsp;
&quot;network&quot;: &quot;tcp,udp&quot;,</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp; &nbsp;&nbsp;&nbsp;&quot;followRedirect&quot;:
true</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp; },</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp; &quot;sniffing&quot;: {</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp;&nbsp;&nbsp;
&quot;enabled&quot;: true,</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp;&nbsp;&nbsp;
&quot;destOverride&quot;: [&quot;http&quot;, &quot;tls&quot;]</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp; }</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;},</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp;&nbsp; {</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&quot;port&quot;: 1081,</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&quot;listen&quot;: &quot;127.0.0.1&quot;,</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&quot;settings&quot;: {</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&quot;udp&quot;: false</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; },</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&quot;protocol&quot;: &quot;socks&quot;</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp;&nbsp; }</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp; ],</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp; &quot;outbounds&quot;: [</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp;&nbsp; {</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&quot;protocol&quot;: &quot;vmess&quot;,</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&quot;settings&quot;: {</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&quot;vnext&quot;: [</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
{</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&quot;address&quot;: &quot;</span>你自己的域名或者<span lang=EN-US>IP</span>地址<span
lang=EN-US>&quot;,</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&quot;users&quot;: [</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
{</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&quot;id&quot;: &quot;</span>你自己的海外<span lang=EN-US>VPS</span>服务端的<span
lang=EN-US>UUID&quot;,</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&quot;alterId&quot;: 64,</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&quot;security&quot;: &quot;auto&quot;,</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&quot;level&quot;: 0</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
}</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
],</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&quot;port&quot;: 443</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
}</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
]</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; },</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&quot;streamSettings&quot;: {</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&quot;wsSettings&quot;: {</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&quot;path&quot;: &quot;</span>你自己的<span lang=EN-US>PATH</span>地址<span
lang=EN-US>&quot;,</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&quot;headers&quot;: {}</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
},</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&quot;tlsSettings&quot;: {</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&quot;allowInsecure&quot;: true,</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&quot;alpn&quot;: [</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&quot;http/1.1&quot;</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
],</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&quot;serverName&quot;: &quot;</span>你自己的域名或者<span lang=EN-US>IP</span>地址<span
lang=EN-US>&quot;,</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&quot;allowInsecureCiphers&quot;: true</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
},</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&quot;security&quot;: &quot;tls&quot;,</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&quot;network&quot;: &quot;ws&quot;,</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&quot;sockopt&quot;: {}</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp;&nbsp; }</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp; ]</span></p>

<p class=MsoNormal><span lang=EN-US>}</span></p>

<p class=MsoNormal><span lang=EN-US>################################</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;</span></p>

<p class=MsoNormal><span lang=EN-US>v2ray</span>的启动</p>

<p class=MsoNormal><span lang=EN-US>################################</span></p>

<p class=MsoNormal><span lang=EN-US>systemctl start v2ray</span></p>

<p class=MsoNormal><span lang=EN-US>################################</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;</span></p>

<p class=MsoNormal>是不是很简单，这样就都搞定了！可以试试这个<span lang=EN-US>DNS</span>服务器的翻墙效果了。</p>

<p class=MsoNormal><span lang=EN-US>&nbsp;</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;</span></p>

<p class=MsoNormal>对于<span lang=EN-US>DNS</span>服务器实际使用时候遇到的一些问题，其实还有其他高深的衍生玩法。我这里大致说一下思路。后续可能会详细写出来。</p>

<p class=MsoNormal>问题一： 这个<span lang=EN-US>DNS</span>服务器我放到外网行不行？</p>

<p class=MsoNormal>行，没问题。但要面临两个大问题。<span lang=EN-US> </span></p>

<p class=MsoNormal><span lang=EN-US>1</span>、这个<span lang=EN-US>DNS</span>服务器会不会被其他的人利用进行科学上网？</p>

<p class=MsoNormal><span lang=EN-US>&nbsp;</span></p>

<p class=MsoNormal>我大致回答一下，如果你将<span lang=EN-US>DNS</span>服务器放到外网，比如阿里云，或者其他的<span
lang=EN-US>IDC</span>，那必然暴露了<span lang=EN-US> udp53 </span>这个<span lang=EN-US>DNS</span>端口
以及更重要的<span lang=EN-US> 80</span>和<span lang=EN-US>443 </span>端口 。那别人一定会用。</p>

<p class=MsoNormal><span lang=EN-US>&nbsp;</span></p>

<p class=MsoNormal>这怎么办？ 可以结合<span lang=EN-US>ipset</span>和<span lang=EN-US>iptables
</span>自定义一个<span lang=EN-US>“</span>开关<span lang=EN-US>”</span>，来达到只有我可以用的目的。</p>

<p class=MsoNormal>怎么做？ 将服务器的<span lang=EN-US> udp 53 </span>端口默认设置为拒绝所有<span
lang=EN-US>IP</span>链接。然后写一个定时脚本，每隔<span lang=EN-US>5</span>秒钟去扫一下<span
lang=EN-US> dnsmasq</span>的查询日志最后<span lang=EN-US>500</span>行，然后过滤一个你自定义的关键字，然后触发<span
lang=EN-US>iptables</span>将你的外网<span lang=EN-US>IP</span>地址设置位允许访问本机的<span
lang=EN-US> udp53 </span>端口。<span lang=EN-US> </span></p>

<p class=MsoNormal>我给一下我这拙略的步骤，以作参考：</p>

<p class=MsoNormal><span lang=EN-US>################################</span></p>

<p class=MsoNormal>先建立<span lang=EN-US>ipset</span>规则集</p>

<p class=MsoNormal><span lang=EN-US>ipset -N allowip iphash</span></p>

<p class=MsoNormal><span lang=EN-US>ipset flush allowip</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;</span></p>

<p class=MsoNormal>然后修改之前提到的<span lang=EN-US>iptables</span>转发规则如下：</p>

<p class=MsoNormal><span lang=EN-US>iptables -t nat -A PREROUTING -i eth0 -p
tcp -m set --match-set allowip src -m multiport --dports 80,443 -j&nbsp;
REDIRECT --to-ports 1080</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;</span></p>

<p class=MsoNormal>最后，增加允许访问<span lang=EN-US>dns</span>服务和<span lang=EN-US>80</span>，<span
lang=EN-US>443</span>端口的规则</p>

<p class=MsoNormal><span lang=EN-US>iptables -A INPUT -p tcp -m set --match-set
allowip src --dport 1080 -j ACCEPT</span></p>

<p class=MsoNormal><span lang=EN-US>iptables -A INPUT -p udp -m set --match-set
allowip src --dport 53 -j ACCEPT</span></p>

<p class=MsoNormal><span lang=EN-US>################################</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;</span></p>

<p class=MsoNormal>定时脚本内容：</p>

<p class=MsoNormal><span lang=EN-US>################################</span></p>

<p class=MsoNormal><span lang=EN-US>while ((1));do</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp;&nbsp; for IP in $(tail -n 500
&quot;dnsmasq</span>的日志<span lang=EN-US>&quot; | awk '/query\[A\].*</span>关键字<span
lang=EN-US>.*/{a[$NF]++}END{for (i in a)print i}')</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp;&nbsp; do</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
ipset -A allowip $IP 2&gt; /dev/null &amp;&amp; echo &quot;$(date +%F_%T) added
$IP&quot;</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp;&nbsp; done</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;&nbsp;&nbsp; sleep 5</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;</span></p>

<p class=MsoNormal><span lang=EN-US>done</span></p>

<p class=MsoNormal><span lang=EN-US>################################</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;</span></p>

<p class=MsoNormal><span lang=EN-US>2</span>、放到外网后，怎样规避自家宽带运营商的<span
lang=EN-US>DNS</span>劫持？</p>

<p class=MsoNormal>这个没什么好办法，只能是把<span lang=EN-US>dnsmasq</span>的端口从默认的<span
lang=EN-US> udp 53 </span>改为其他非标端口。 但本地客户端，例如手机，电脑可就不那么好弄向外的流量重定向了。</p>

<p class=MsoNormal>其实我也只是知道<span lang=EN-US> linux</span>利用<span lang=EN-US>iptables</span>怎么做。</p>

<p class=MsoNormal><span lang=EN-US>iptables -t nat -A OUTPUT -d
208.67.222.222/32 -p udp -m udp --dport 53 -j DNAT --to-destination
208.67.222.222:443</span></p>

<p class=MsoNormal><span lang=EN-US>&nbsp;</span></p>

<p class=MsoNormal>所以运营商劫持<span lang=EN-US>DNS</span>没什么好办法。 我这里写的也就更适用于内网环境而已。比如一家规模不大的公司内，或者一个团队使用。</p>

<p class=MsoNormal><span lang=EN-US>&nbsp;</span></p>

</div>

</body>

</html>
