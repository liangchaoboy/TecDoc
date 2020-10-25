### tcpdump 命令使用参考

#### tcpdump 命令简介

- 对网络请求数据进行抓取分析

### 功能

- 支持按 ip、端口 、协议进行抓包并进行分析

### 使用示例

- 指定网卡，如果不指定，默认从第一个网卡进行抓取
```
tcpdump -i eth0 // 可以先通过 ifconfig 命令查看网络
```

- 指定 ip

```
root@TestA:~# tcpdump -i eth0 host 180.101.136.3
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes
21:43:50.679855 IP 180.101.136.3 > TestA: ICMP echo request, id 12638, seq 1, length 64
21:43:50.679924 IP TestA > 180.101.136.3: ICMP echo reply, id 12638, seq 1, length 64
```

- 指定端口
  ```
  root@TestA:~# tcpdump -i eth0 port 81
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes
21:44:52.106245 IP 180.101.136.3.43148 > TestA.81: Flags [S], seq 3823244155, win 29200, options [mss 1460,nop,nop,sackOK,nop,wscale 7], length 0
21:44:53.104791 IP 180.101.136.3.43148 > TestA.81: Flags [S], seq 3823244155, win 29200, options [mss 1460,nop,nop,sackOK,nop,wscale 7], length 0

  ```

- 指定 ip 并且指定 端口
  ```
  root@TestA:~# tcpdump -vi eth0 host 180.101.136.3 and port 89
tcpdump: listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes
21:46:11.715203 IP (tos 0x14, ttl 51, id 6787, offset 0, flags [DF], proto TCP (6), length 52)
    180.101.136.3.55600 > TestA.89: Flags [S], cksum 0xe06c (correct), seq 2939982873, win 29200, options [mss 1460,nop,nop,sackOK,nop,wscale 7], length 0
21:46:12.713423 IP (tos 0x14, ttl 51, id 6788, offset 0, flags [DF], proto TCP (6), length 52)
    180.101.136.3.55600 > TestA.89: Flags [S], cksum 0xe06c (correct), seq 2939982873, win 29200, options [mss 1460,nop,nop,sackOK,nop,wscale 7], length 0

  ```

- 过滤发送到指定 ip 的所有数据
  
  ```
   tcpdump -vi eth0 dst host 180.101.136.3 and port 80
tcpdump: listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes
21:51:09.396492 IP (tos 0x0, ttl 64, id 0, offset 0, flags [DF], proto TCP (6), length 52)
    TestA.http > 180.101.136.3.10810: Flags [S.], cksum 0xffda (incorrect -> 0x5fbc), seq 3875110786, ack 1233074714, win 64240, options [mss 1460,nop,nop,sackOK,nop,wscale 7], length 0
21:51:09.405343 IP (tos 0x0, ttl 64, id 59265, offset 0, flags [DF], proto TCP (6), length 40)
    TestA.http > 180.101.136.3.10810: Flags [.], cksum 0xffce (incorrect -> 0x993a), ack 80, win 502, length 0
21:51:09.405555 IP (tos 0x0, ttl 64, id 59266, offset 0, flags [DF], proto TCP (6), length 899)
    TestA.http > 180.101.136.3.10810: Flags [P.], cksum 0x032a (incorrect -> 0xc9ba), seq 1:860, ack 80, win 502, length 859: HTTP, length: 859
	HTTP/1.1 200 OK
	Server: nginx/1.18.0 (Ubuntu)
	Date: Sun, 18 Oct 2020 13:51:09 GMT
	Content-Type: text/html
	Content-Length: 612
	Last-Modified: Tue, 13 Oct 2020 13:59:53 GMT
	Connection: keep-alive
	ETag: "5f85b2d9-264"
	Accept-Ranges: bytes

	<!DOCTYPE html>
	<html>
	<head>
	<title>Welcome to nginx!</title>
	<style>
	    body {
	        width: 35em;
	        margin: 0 auto;
	        font-family: Tahoma, Verdana, Arial, sans-serif;
	    }
	</style>
	</head>
	<body>
	<h1>Welcome to nginx!</h1>
	<p>If you see this page, the nginx web server is successfully installed and
	working. Further configuration is required.</p>

	<p>For online documentation and support please refer to
	<a href="http://nginx.org/">nginx.org</a>.<br/>
	Commercial support is available at
	<a href="http://nginx.com/">nginx.com</a>.</p>

	<p><em>Thank you for using nginx.</em></p>
	</body>
	</html>
21:51:09.414718 IP (tos 0x0, ttl 64, id 59267, offset 0, flags [DF], proto TCP (6), length 40)
    TestA.http > 180.101.136.3.10810: Flags [F.], cksum 0xffce (incorrect -> 0x95dd), seq 860, ack 81, win 502, length 0
  ```
- 过滤从指定 ip 发出的所有数据
  ```
root@TestA:~# tcpdump -vi eth0 src host 192.168.2.163
    TestA.ssh > 180.164.57.206.36727: Flags [P.], cksum 0xb208 (incorrect -> 0x917e), seq 353124:353160, ack 181, win 501, options [nop,nop,TS val 4140739730 ecr 1168435940], length 36
22:01:08.972861 IP (tos 0x10, ttl 64, id 32726, offset 0, flags [DF], proto TCP (6), length 700)
    TestA.ssh > 180.164.57.206.36727: Flags [P.], cksum 0xb46c (incorrect -> 0x5dab), seq 353160:353808, ack 181, win 501, options [nop,nop,TS val 4140739733 ecr 1168435944], length 648

  ```
  
- 指定 tcp 或者 udp

```
root@TestA:~# tcpdump tcp port 80 and host 192.168.2.163
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes
22:04:04.638487 IP TestA.51338 > 100.100.30.26.http: Flags [P.], seq 1462944204:1462945726, ack 796657844, win 501, length 1522: HTTP
22:04:04.646415 IP 100.100.30.26.http > TestA.51338: Flags [.], ack 1522, win 790, length 0

root@TestA:~# tcpdump udp port 80

```
- 单独抓 http 协议数据包
  ```
  tcpdump  -XvvennSs 0 -i eth0 tcp[20:2]=0x4745 or tcp[20:2]=0x4854  and host 180.101.136.3
tcpdump: listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes
22:12:23.269568 ee:ff:ff:ff:ff:ff > 00:16:3e:1a:b8:32, ethertype IPv4 (0x0800), length 133: (tos 0x14, ttl 51, id 12567, offset 0, flags [DF], proto TCP (6), length 119)
    180.101.136.3.45907 > 192.168.2.163.80: Flags [P.], cksum 0x5f01 (correct), seq 1959312454:1959312533, ack 2476034580, win 229, length 79: HTTP, length: 79
	GET / HTTP/1.1
	User-Agent: curl/7.35.0
	Host: 139.196.214.206
	Accept: */*

	0x0000:  4514 0077 3117 4000 3306 16a2 b465 8803  E..w1.@.3....e..
	0x0010:  c0a8 02a3 b353 0050 74c8 bc46 9395 4a14  .....S.Pt..F..J.
	0x0020:  5018 00e5 5f01 0000 4745 5420 2f20 4854  P..._...GET./.HT
	0x0030:  5450 2f31 2e31 0d0a 5573 6572 2d41 6765  TP/1.1..User-Age
	0x0040:  6e74 3a20 6375 726c 2f37 2e33 352e 300d  nt:.curl/7.35.0.
	0x0050:  0a48 6f73 743a 2031 3339 2e31 3936 2e32  .Host:.139.196.2
	0x0060:  3134 2e32 3036 0d0a 4163 6365 7074 3a20  14.206..Accept:.
	0x0070:  2a2f 2a0d 0a0d 0a                        */*....
22:12:23.269844 00:16:3e:1a:b8:32 > ee:ff:ff:ff:ff:ff, ethertype IPv4 (0x0800), length 913: (tos 0x0, ttl 64, id 58160, offset 0, flags [DF], proto TCP (6), length 899)
    192.168.2.163.80 > 180.101.136.3.45907: Flags [P.], cksum 0x032a (incorrect -> 0x24fc), seq 2476034580:2476035439, ack 1959312533, win 502, length 859: HTTP, length: 859
	HTTP/1.1 200 OK
	Server: nginx/1.18.0 (Ubuntu)
	Date: Sun, 18 Oct 2020 14:12:23 GMT
	Content-Type: text/html
	Content-Length: 612
	Last-Modified: Tue, 13 Oct 2020 13:59:53 GMT
	Connection: keep-alive
	ETag: "5f85b2d9-264"
	Accept-Ranges: bytes

	<!DOCTYPE html>
	<html>
	<head>
	<title>Welcome to nginx!</title>
	<style>
	    body {
	        width: 35em;
	        margin: 0 auto;
	        font-family: Tahoma, Verdana, Arial, sans-serif;
	    }
	</style>
	</head>
	<body>
	<h1>Welcome to nginx!</h1>
	<p>If you see this page, the nginx web server is successfully installed and
	working. Further configuration is required.</p>

	<p>For online documentation and support please refer to
	<a href="http://nginx.org/">nginx.org</a>.<br/>
	Commercial support is available at
	<a href="http://nginx.com/">nginx.com</a>.</p>

	<p><em>Thank you for using nginx.</em></p>
	</body>
	</html>
	0x0000:  4500 0383 e330 4000 4006 5490 c0a8 02a3  E....0@.@.T.....
	0x0010:  b465 8803 0050 b353 9395 4a14 74c8 bc95  .e...P.S..J.t...
	0x0020:  5018 01f6 032a 0000 4854 5450 2f31 2e31  P....*..HTTP/1.1
	0x0030:  2032 3030 204f 4b0d 0a53 6572 7665 723a  .200.OK..Server:
	0x0040:  206e 6769 6e78 2f31 2e31 382e 3020 2855  .nginx/1.18.0.(U
	0x0050:  6275 6e74 7529 0d0a 4461 7465 3a20 5375  buntu)..Date:.Su
	0x0060:  6e2c 2031 3820 4f63 7420 3230 3230 2031  n,.18.Oct.2020.1
	0x0070:  343a 3132 3a32 3320 474d 540d 0a43 6f6e  4:12:23.GMT..Con
	0x0080:  7465 6e74 2d54 7970 653a 2074 6578 742f  tent-Type:.text/
	0x0090:  6874 6d6c 0d0a 436f 6e74 656e 742d 4c65  html..Content-Le
	0x00a0:  6e67 7468 3a20 3631 320d 0a4c 6173 742d  ngth:.612..Last-
	0x00b0:  4d6f 6469 6669 6564 3a20 5475 652c 2031  Modified:.Tue,.1
	0x00c0:  3320 4f63 7420 3230 3230 2031 333a 3539  3.Oct.2020.13:59
	0x00d0:  3a35 3320 474d 540d 0a43 6f6e 6e65 6374  :53.GMT..Connect
	0x00e0:  696f 6e3a 206b 6565 702d 616c 6976 650d  ion:.keep-alive.
	0x00f0:  0a45 5461 673a 2022 3566 3835 6232 6439  .ETag:."5f85b2d9
	0x0100:  2d32 3634 220d 0a41 6363 6570 742d 5261  -264"..Accept-Ra
	0x0110:  6e67 6573 3a20 6279 7465 730d 0a0d 0a3c  nges:.bytes....<
	0x0120:  2144 4f43 5459 5045 2068 746d 6c3e 0a3c  !DOCTYPE.html>.<
	0x0130:  6874 6d6c 3e0a 3c68 6561 643e 0a3c 7469  html>.<head>.<ti
	0x0140:  746c 653e 5765 6c63 6f6d 6520 746f 206e  tle>Welcome.to.n
	0x0150:  6769 6e78 213c 2f74 6974 6c65 3e0a 3c73  ginx!</title>.<s
	0x0160:  7479 6c65 3e0a 2020 2020 626f 6479 207b  tyle>.....body.{
	0x0170:  0a20 2020 2020 2020 2077 6964 7468 3a20  .........width:.
	0x0180:  3335 656d 3b0a 2020 2020 2020 2020 6d61  35em;.........ma
	0x0190:  7267 696e 3a20 3020 6175 746f 3b0a 2020  rgin:.0.auto;...
	0x01a0:  2020 2020 2020 666f 6e74 2d66 616d 696c  ......font-famil
	0x01b0:  793a 2054 6168 6f6d 612c 2056 6572 6461  y:.Tahoma,.Verda
	0x01c0:  6e61 2c20 4172 6961 6c2c 2073 616e 732d  na,.Arial,.sans-
	0x01d0:  7365 7269 663b 0a20 2020 207d 0a3c 2f73  serif;.....}.</s
	0x01e0:  7479 6c65 3e0a 3c2f 6865 6164 3e0a 3c62  tyle>.</head>.<b
	0x01f0:  6f64 793e 0a3c 6831 3e57 656c 636f 6d65  ody>.<h1>Welcome
	0x0200:  2074 6f20 6e67 696e 7821 3c2f 6831 3e0a  .to.nginx!</h1>.
	0x0210:  3c70 3e49 6620 796f 7520 7365 6520 7468  <p>If.you.see.th
	0x0220:  6973 2070 6167 652c 2074 6865 206e 6769  is.page,.the.ngi
	0x0230:  6e78 2077 6562 2073 6572 7665 7220 6973  nx.web.server.is
	0x0240:  2073 7563 6365 7373 6675 6c6c 7920 696e  .successfully.in
	0x0250:  7374 616c 6c65 6420 616e 640a 776f 726b  stalled.and.work
	0x0260:  696e 672e 2046 7572 7468 6572 2063 6f6e  ing..Further.con
	0x0270:  6669 6775 7261 7469 6f6e 2069 7320 7265  figuration.is.re
	0x0280:  7175 6972 6564 2e3c 2f70 3e0a 0a3c 703e  quired.</p>..<p>
	0x0290:  466f 7220 6f6e 6c69 6e65 2064 6f63 756d  For.online.docum
	0x02a0:  656e 7461 7469 6f6e 2061 6e64 2073 7570  entation.and.sup
	0x02b0:  706f 7274 2070 6c65 6173 6520 7265 6665  port.please.refe
	0x02c0:  7220 746f 0a3c 6120 6872 6566 3d22 6874  r.to.<a.href="ht
	0x02d0:  7470 3a2f 2f6e 6769 6e78 2e6f 7267 2f22  tp://nginx.org/"
	0x02e0:  3e6e 6769 6e78 2e6f 7267 3c2f 613e 2e3c  >nginx.org</a>.<
	0x02f0:  6272 2f3e 0a43 6f6d 6d65 7263 6961 6c20  br/>.Commercial.
	0x0300:  7375 7070 6f72 7420 6973 2061 7661 696c  support.is.avail
	0x0310:  6162 6c65 2061 740a 3c61 2068 7265 663d  able.at.<a.href=
	0x0320:  2268 7474 703a 2f2f 6e67 696e 782e 636f  "http://nginx.co
	0x0330:  6d2f 223e 6e67 696e 782e 636f 6d3c 2f61  m/">nginx.com</a
	0x0340:  3e2e 3c2f 703e 0a0a 3c70 3e3c 656d 3e54  >.</p>..<p><em>T
	0x0350:  6861 6e6b 2079 6f75 2066 6f72 2075 7369  hank.you.for.usi
	0x0360:  6e67 206e 6769 6e78 2e3c 2f65 6d3e 3c2f  ng.nginx.</em></
	0x0370:  703e 0a3c 2f62 6f64 793e 0a3c 2f68 746d  p>.</body>.</htm
	0x0380:  6c3e 0a                                  l>.

  ```  

- 抓包达到指定大小保存一次，如果不主动停止，会一直抓包下去
  
  ```
  tcpdump -Z root -C 1  -w ./3xx.pcap -s0 
  // -Z 指定用户，不指定会报错
  // -C 每当文件达到指定大小时会新保存一个文件，单位是 MB ( 1,000,000 bytes,  not 1,048,576 bytes)
  // -w 指定保存的文件名
  // -s0 表示每个报文的大小是接收到的指定大小，如果没有这个选项，则超过比如1500字节的报文，就会被切除1500字节以外的部分
  ```


- 指定间隔多长时间抓包一次，并指定多久退出
  
  ```
  root@iZuf6fnkpstcttph1gn47zZ:~# tcpdump -Z root  host 58.33.27.210 and port 80 -G 5 -W 3 -w ./%Y_%m_%d_%H_%M_%S_xx.pcap
	tcpdump: listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes
	Maximum file limit reached: 3
	14501 packets captured
	14823 packets received by filter
	0 packets dropped by kernel
	root@iZuf6fnkpstcttph1gn47zZ:~# ll
	-rw-r--r--  1 root    root    3317105 Oct 20 23:34 2020_10_20_23_34_36_xx.pcap
	-rw-r--r--  1 root    root    3788704 Oct 20 23:34 2020_10_20_23_34_41_xx.pcap
	-rw-r--r--  1 root    root    3792069 Oct 20 23:34 2020_10_20_23_34_46_xx.pcap
	// -G 间隔多少秒保留一次文件，单位是秒
	// -W 与-G选项一起使用，指定保存文件的数量，达到限制后以状态0退出。如果也与-C一起使用，每个时间片具有周期性文件
  ```

- tcpdump 抓 udp 的包

```
tcpdump udp port 53
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes
17:12:25.152320 IP iZuf6fnkpstcttph1gn47zZ.56705 > 100.100.2.138.domain: 29351+ AAAA? cn-shanghai.axt.aliyun.com. (44)
17:12:25.152496 IP 100.100.2.138.domain > iZuf6fnkpstcttph1gn47zZ.56705: 29351* 0/1/0 (117)
17:12:25.152792 IP iZuf6fnkpstcttph1gn47zZ.51306 > 100.100.2.138.domain: 50513+ AAAA? cn-shanghai.axt.aliyun.com. (44)
17:12:25.152920 IP iZuf6fnkpstcttph1gn47zZ.48707 > 100.100.2.138.domain: 46831+ PTR? 138.2.100.100.in-addr.arpa. (44)
17:12:25.152955 IP 100.100.2.138.domain > iZuf6fnkpstcttph1gn47zZ.51306: 50513* 0/1/0 (117)
17:12:25.153061 IP 100.100.2.138.domain > iZuf6fnkpstcttph1gn47zZ.48707: 46831 NXDomain* 0/1/0 (99)
17:12:25.153277 IP iZuf6fnkpstcttph1gn47zZ.35792 > 100.100.2.138.domain: 60822+ A? cn-shanghai.axt.aliyun.com. (44)
17:12:25.153377 IP 100.100.2.138.domain > iZuf6fnkpstcttph1gn47zZ.35792: 60822* 1/0/0 A 100.100.36.108 (60)

```

### 命令详解

```

-c 指定抓多少个包后退出 // tcpdump -c 2 host 58.33.27.210 and   port 80
-w 保存到本地文件
-Z 指定用户名，有时需要指定 root 才能进行抓包
-G 间隔多少秒保留一次文件，单位是秒
-W 与-G选项一起使用，指定保存文件的数量，达到限制后以状态0退出。如果也与-C一起使用，每个时间片具有周期性文件
-C 每当文件达到指定大小时会新保存一个文件，单位是 MB ( 1,000,000 bytes,  not 1,048,576 bytes)
-r 读取 -w 保存到本地的数据包，文件名后缀需要是 .pcap //tcpdump  -r ./test_c_command.pcap -Z root

```

